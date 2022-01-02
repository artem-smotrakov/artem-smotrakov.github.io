---
layout: post
title: Python marshal module fuzzing
date: 2016-09-20 19:24:32.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Fuzzing
- Python
permalink: "/en/security/python-marshal-module-fuzzing.html"
---
[The marshal module](https://docs.python.org/3/library/marshal.html) provides a serialization mechanism for Python values. In other words, the module contains&nbsp;functions for writing/reading Python objects in a binary format. Unfortunately the format is undocumented, and Python maintainers may&nbsp;change&nbsp;the format in backward incompatible ways between Python version. The marshal module is used internally by other Python components, for example, for reading and writing .pyc files which contain pseudo-compiled Python code. But&nbsp;Python also has public API to access this serialization mechanism.

This post shows how the marshal module can be quickly tested with a simple dumb fuzzer, and why the module shouldn't be used with untrusted data.

The marshal module is [implemented](https://hg.python.org/cpython/file/tip/Python/marshal.c) in C, so the simplest&nbsp;goal of fuzzing here is just to look for typical&nbsp;issues in C code like buffer overflows, use-after-free, null-pointer dereferences, etc. [AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer) (ASan) is a great memory checker&nbsp;which can help with identifying such issues. AddressSanitizer instruments code&nbsp;while compilation. The tool replaces malloc and free functions, and adds check for memory corruption issues. Then, at runtime it tries to detect memory corruptions, and report them immediately with lots of useful information. AddressSanitizer is part of GCC 4.8+ which can be used to build Python.

## Building Python with AddressSanitizer

Python code (CPython) can be cloned with the following command:

```
hg clone https://hg.python.org/cpython
```

If you run `./configure --help`, you can see that it has `--with-address-sanitizer` option which is supposed to enable AddressSanitizer. But for some reason it didn't work for me, so I just used the following commands to build Python:

```
CFLAGS="-g -fsanitize=address -fno-omit-frame-pointer -O0" \
CPPFLAGS="-fsanitize=address -fno-omit-frame-pointer -O0" \
LDFLAGS="-fsanitize=address" \
    ./configure \
        --prefix=/home/artem/projects/fuzzing/python/build/ \
        --disable-ipv6
ASAN_OPTIONS="detect_leaks=0" make
ASAN_OPTIONS="detect_leaks=0" make install
```

Let me quickly explain&nbsp;what those options mean:

1. CFLAGS, LDFLAGS,&nbsp;CPPFLAGS are standard enviroment variable which specify options for C/C++ compiler and linker.
2. `-fsanitize=address` enables AddressSanitizer (it has to be passed to both compiler and linker)
3. `-g` makes GCC produce debugging information.
4. `-O0` turns off compiler optimizations (but slows down execution).
5. `-fno-omit-frame-pointer` is for nicer stack traces.
6. ASAN\_OPTIONS is an environment variable which contains parameters for AddressSanitizer at runtime.
7. `ASAN_OPTIONS="detect_leaks=0"` turns off [memory leaks checker](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer) which is part of AddressSanitizer.
8. `--prefix` specifies a directory where it should put output binaries, libs, etc.
9. `--disable-ipv6` disables IPv6 (nothing surprising).

If the build runs smoothly, you can run `python3.6 --version` as a smoke test.

## Fuzzing Python marshal module

There are a lot of fuzzers. You can choose a simple dumb fuzzer like [zzuf](https://github.com/samhocevar/zzuf), or use something more intelligent like [American Fuzzy Lop](http://lcamtuf.coredump.cx/afl/) (ALF). Or, you can always invent a bicycle - here is a simple dumb fuzzer for the marshal module written in Python:

[https://github.com/artem-smotrakov/python-marshal-fuzzer](https://github.com/artem-smotrakov/python-marshal-fuzzer)

In general, this fuzzer is very similar to zzuf. Here is a couple of words about how it works:

1. DumbByteArrayFuzzer class is a simple dumb fuzzer for a byte array. It takes a byte array, and randomly modifies it depending on initial settings.
  1. `data` is an original byte array to fuzz.
  2. `seed` parameter specifies a seed for pseudo-random generator.
  3. `min_ratio` and `max_ratio` parameters specify min and max fraction&nbsp;of the byte array to be fuzzed.
  4. `DumbByteArrayFuzzer` generates reproducible data (test case), and `start_test` parameter specifies a test case to start from.
  5. `ignored_bytes` specifies symbols that should be ignored while fuzzing.
2. First, `fuzzer.py` parses command line options.
3. Next, it defines `value` object which is then serialized by `marshal.dumps()` method.
4. In the end of `fuzzer.py`, it creates an instance of `DumbByteArrayFuzzer`, and starts the main fuzzing loop
5. In the loop, it&nbsp;calls `next()` method to generate fuzzed data which is passed to `marshal.loads()`
6. The spec says that the following exception are expected: `EOFError`, `ValueError`, `TypeError`. The fuzzer just ignores them.

The fuzzer can be run with default parameters with the command like the following (no checks for memory leaks):

```
ASAN_OPTIONS="detect_leaks=0" \
    /home/artem/projects/fuzzing/python/build/bin/python fuzzer.py
```

## Segmentation fault in the marshal module

After some time, AddressSanitizer reported the following problem:

```
ASAN:SIGSEGV
=================================================================
==20296==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000008 (pc 0x000000582064 bp 0x7ffc9e581310 sp 0x7ffc9e5812f0 T0)
#0 0x582063 in PyObject_Hash Objects/object.c:769
#1 0x5a3662 in tuplehash Objects/tupleobject.c:358
#2 0x5820ae in PyObject_Hash Objects/object.c:771
#3 0x5a3662 in tuplehash Objects/tupleobject.c:358
#4 0x5820ae in PyObject_Hash Objects/object.c:771
#5 0x58fac8 in set_add_key Objects/setobject.c:422
#6 0x59a85c in PySet_Add Objects/setobject.c:2323
#7 0x760d9d in r_object Python/marshal.c:1310
#8 0x76029d in r_object Python/marshal.c:1223
#9 0x760015 in r_object Python/marshal.c:1195
#10 0x7621dc in read_object Python/marshal.c:1465
#11 0x7639be in marshal_loads Python/marshal.c:1767
#12 0x577ff3 in PyCFunction_Call Objects/methodobject.c:109
#13 0x708a05 in call_function Python/ceval.c:4744
#14 0x6fb5a7 in PyEval_EvalFrameEx Python/ceval.c:3256
#15 0x70276f in _PyEval_EvalCodeWithName Python/ceval.c:4050
#16 0x70299f in PyEval_EvalCodeEx Python/ceval.c:4071
#17 0x6e07d7 in PyEval_EvalCode Python/ceval.c:778
#18 0x432354 in run_mod Python/pythonrun.c:980
#19 0x431e5b in PyRun_FileExFlags Python/pythonrun.c:933
#20 0x42e929 in PyRun_SimpleFileExFlags Python/pythonrun.c:396
#21 0x42caba in PyRun_AnyFileExFlags Python/pythonrun.c:80
#22 0x45f995 in run_file Modules/main.c:319
#23 0x4619c8 in Py_Main Modules/main.c:777
#24 0x41d258 in main Programs/python.c:69
#25 0x7f374629babf in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x20abf)
#26 0x41ce28 in _start (/home/artem/projects/fuzzing/python/build/bin/python3.6+0x41ce28)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV Objects/object.c:769 PyObject_Hash
==20296==ABORTING
```

Here is the original data structure which was used for fuzzing:

```
value = ( # tuple1
    "this is a string", #string1
    [
        1, # int1
        2, # int2
        3, # int3
        4 # int4
    ],
    ( #tuple2
        "more tuples", #string2
        1.0, # float1
        2.3, # float2
        4.5 # float3
    ),
    "this is yet another string"
)
```

The fuzzer modified it with the following:

1. First, it update type of `int2` item to `TYPE_SET`.
2. As a result, `int3` item became a length of the set.
3. Then, it updated `float3` item to `TYPE_REF` which points to `tuple1` item.

In other words, now it is a&nbsp;a recursive tuple.&nbsp;What happened when marshal.loads() tried to deserialize this fuzzed data:

1. `int2` item&nbsp;is now a set of length 3.
2. First, It adds `int4` item to the set.
3. Next, it adds `tuple2` item:
  1. When an object is added to a set, it calculates a hash of this object
  2. When it calculates a hash of a tuple, it calculates hashes of all items&nbsp;from this tuple.
  3. During calculating a hash of `tuple2`, it calculates a hash of `tuple1` because `float3` now is a `TYPE_REF` item which points to `tuple1`.
  4. But `tuple1` is not complete yet. The length of `tuple1` is 4, but only `string1` has been added&nbsp;to it so far.
  5. `tuplehash()` function reads a length of a tuple, and then calls `PyObject_Hash()` fucntion for each item of the tuple.
  6. But it doesn't check if a tuple is complete, and all elements have been&nbsp;added to the tuple.
  7. As a result, a null-pointer dereference happens in `tuplehash()` function when it&nbsp;reads&nbsp;second item of `tuple1`.

See [https://hg.python.org/cpython/file/tip/Objects/tupleobject.c#l347](https://hg.python.org/cpython/file/tip/Objects/tupleobject.c#l347)&nbsp;for details:

```
static Py_hash_t
tuplehash(PyTupleObject *v)
{
    Py_uhash_t x; /* Unsigned for defined overflow behavior. */
    Py_hash_t y;
    Py_ssize_t len = Py_SIZE(v);
    PyObject **p;
    Py_uhash_t mult = _PyHASH_MULTIPLIER;
    x = 0x345678UL;
    p = v->ob_item;
    while (--len >= 0) {
    y = PyObject_Hash(*p++); <=
```

For `tuple1`,&nbsp;`Py_SIZE(v)` returns 4, but `tuple1` contains only one element `string1`. A null-pointer dereference happens in `PyObject_Hash()` while reading second element.

Even if it doesn't seem to be a serious security issue, the problem was originally reported to Python Security Response Team. They said they&nbsp;don't consider crashes due to malicious&nbsp;marshal data to be security bugs. And documentation for the marshal module has a note about it:

> Warning: The marshal module is not intended to be secure against erroneous or maliciously constructed data. Never unmarshal data received from an untrusted or unauthenticated source.

Then, the problem was [reported](http://bugs.python.org/issue27826) to Python maintainers, but they decided not to fix it probably because of performance.

## Conclusion

As they mentioned in documentation for the marshal module, it should not be used for unmarshaling data received from an untrusted party because the module is not intended to be secure against malicious data. Furthermore, some issues (like above) are not going to be fixed even if they are known.

The interesting thing is that at the moment of posting this article I found&nbsp;[59601 usages of marshal.loads() function on GitHub](https://github.com/search?l=python&q=marshal.loads&type=Code&utf8=%E2%9C%93). I hope they know what they are doing.

