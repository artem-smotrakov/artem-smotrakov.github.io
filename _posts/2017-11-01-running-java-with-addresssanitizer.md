---
layout: post
title: Running Java with AddressSanitizer
date: 2017-11-01 09:38:29.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- Open Source
- Security
meta:
  _edit_last: '1'
  mask_links: default
  _wpgo_column_layout_save: ''
  _wpgo_column_layout: default
  _oembed_6f8d2ab3bbf7f76150c14a7476bc3e65: "{{unknown}}"
  _oembed_636bc6e40ea5630cf5cd2b216711ee08: "{{unknown}}"
  _syntaxhighlighter_encoded: '1'
  _oembed_9edd0123303dcf23f455851c2ba12ff2: "{{unknown}}"
  _oembed_49b9337c670e76e404239a3363f1127c: "{{unknown}}"
  _aioseop_opengraph_settings: a:14:{s:32:"aioseop_opengraph_settings_title";s:0:"";s:31:"aioseop_opengraph_settings_desc";s:0:"";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}
  _aioseop_keywords: Java, AddressSanitizer, ASan, OpenJDK, JTREG, buffer overflow,
    memory corruption, detection
  _aioseop_description: 'How to use AddressSanitizer with Java: building OpenJDK with
    enabled AddressSanitizer, running jtreg tests, and known issues.'
  _aioseop_title: Running Java with AddressSanitizer
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: ''
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_focuskw_text_input: AddressSanitizer
  _yoast_wpseo_focuskw: AddressSanitizer
  _yoast_wpseo_metadesc: OpenJDK and AddressSanitizer are well-known open source projects.
    OpenJDK sources contain C/C++ code which may be affected by memory corruption
    issues and memory leaks. Such issues may be detected at runtime with memory checkers
    like AddressSanitizer. Now it's going to be easier to use AddressSanitizer for
    OpenJDK development to check for memory corruptions and leaks.
  _yoast_wpseo_linkdex: '65'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617982800'
permalink: "/en/security/running-java-with-addresssanitizer.html"
---
OpenJDK and AddressSanitizer are well-known open source projects. OpenJDK sources contain C/C++ code which may be affected by memory corruption issues and memory leaks. Such issues may be detected at runtime with memory checkers like AddressSanitizer. Now it's going to be easier to use AddressSanitizer for OpenJDK development to check for memory corruptions and leaks.



## What is AddressSanitizer?

AddressSanitizer (aka ASan) is a memory error detector for C/C++ which looks for issues like read/write buffer overruns, use-after-free issues and memory leaks. Basically AddressSanitizer instruments C/C++ code with additional checks which allows to detect memory corruptions when the instrumented code is running. This tools is available in GCC 4.8+ and Clang 3.1+ and is supported on multiple platforms including Linux x64 and MacOS x64. The tools can be enabled by passing `-fsanitize=address` command line options to GCC or Clang compiler and linker. See more details on the official page:

[https://github.com/google/sanitizers/wiki/AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)

## How AddressSanitizer can be used in OpenJDK development?

Java (or OpenJDK to be precise) contains C/C++ code, for example:

- Most of Hotspot JVM is written in C/C++
- Java Core Libraries use JNI to access system functions. See for example I/O and networking components.
- Java Security Libraries use JNI to perform some cryptography operations faster, and access system crypto API with PKCS11 API.

OpenJDK developers are only humans, and may make mistakes which may result to buffer overrun bugs and memory leaks. To find such issues, memory checkers like AddressSanitizer may be used.

## How to use AddressSanitizer with Java

From now OpenJDK can be built with enabled AddressSanitizer. Support for AddressSanitizer has been recently added to OpenJDK configure/make files:

[http://hg.openjdk.java.net/jdk10/master/rev/77a5f2ef1807](http://hg.openjdk.java.net/jdk10/master/rev/77a5f2ef1807)

To enable ASan in OpenJDK, `--enable-asan` command line option should be passed to `configure` script. The commands below shows how OpenJDK can be built with AddressSanitizer:

```
# clone OpenJDK workspace
hg clone http://hg.openjdk.java.net/jdk10/master jdk10
cd jdk10

# configure
# you can also use 'fastdebug' or just --enable-debug
bash configure --with-boot-jdk=/path/to/jdk9 \
    --with-debug-level=slowdebug \
    --enable-asan

# build
make images
```

You can run `bash configure --help`, or find more details here

&nbsp;[http://hg.openjdk.java.net/jdk10/master/file/5a1afee9b9e2/doc/building.md](http://hg.openjdk.java.net/jdk10/master/file/5a1afee9b9e2/doc/building.md)

When `make` finishes, run&nbsp;`java -version`&nbsp;as a smoke test. You'll need to disabling SEGV handler by passing `handle_segv=0`&nbsp;to AddressSanitizer:

```
bash-4.2$ ASAN_OPTIONS="handle_segv=0" ./build/linux-x64/images/jdk/bin/java -version
java version "10-internal"
Java(TM) SE Runtime Environment (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan)
Java HotSpot(TM) 64-Bit Server VM (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan, mixed mode)
```

Without `handle_segv=0` option you are going to get the following error:

```
ASAN:SIGSEGV
=================================================================
==115049==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7f2e1c5eb513 sp 0x7f2e381709d8 bp 0x7f2e3df5a5c0 T1)
#0 0x7f2e1c5eb512 (+0x512)
AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV ??:0 ??
Thread T1 created by T0 here:
#0 0x7f2e418f18aa in __interceptor_pthread_create /opt/jprt/jprtadm/erik/jdk9-dev/build/devkit/src/gcc-4.9.2/libsanitizer/asan/asan_interceptors.cc:183
#1 0x7f2e41284416 in ContinueInNewThread0 (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0x1c416)
#2 0x7f2e4128006d in ContinueInNewThread (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0x1806d)
#3 0x7f2e4128467a in JVMInit (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0x1c67a)
#4 0x7f2e41275161 in JLI_Launch (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0xd161)
#5 0x401272 in main (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/java+0x401272)
#6 0x7f2e40cc4b34 in __libc_start_main (/lib64/libc.so.6+0x21b34)
==115049==ABORTING
```

By default, AddressSanitizer also enables [LeakSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer). Currently it's not quite clear if LeakSanitizer works correctly with Hotspot JVM, so you may also want to put `detect_leaks=0` to ASAN\_OPTIONS to disable LeakSanitizer:

```
bash-4.2$ ASAN_OPTIONS="handle_segv=0 detect_leaks=0" ./build/linux-x64/images/jdk/bin/java -version
java version "10-internal"
Java(TM) SE Runtime Environment (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan)
Java HotSpot(TM) 64-Bit Server VM (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan, mixed mode)
```

You can find more info about AddressSanitizer's flags here

[https://github.com/google/sanitizers/wiki/SanitizerCommonFlags](https://github.com/google/sanitizers/wiki/SanitizerCommonFlags)

## Running tests with AddressSanitizer

[If you contribute to OpenJDK](http://openjdk.java.net/contribute/), and your patch touches C/C++ code, you may want to run existing tests in OpenJDK workspace with enabled AddressSanitizer. To do that, just run `make test`. This command is going to run all automated tests. If you'd like to run only subset of existing tests you can use something like `make run-test TEST=jdk_lang`. Or, just run tests with Jtreg command. Refer to&nbsp;[Testing OpenJDK](http://hg.openjdk.java.net/jdk10/master/file/5a1afee9b9e2/doc/testing.md) for details.

## Known issues

Code which create new processes with ProcessBuilder class may fail with a message like the following:

```
Address 0x7fceacab7280 is located in stack of thread T24 (MainThread)==10175==AddressSanitizer CHECK failed: ../../../../src/libsanitizer/asan/asan_thread.cc:231 "((ptr[0] == kCurrentStackFrameMagic)) != (0)" (0x0, 0x0)
```

Process API uses fork() and vfork() calls. But at the same time, Hotspot JVM uses threads API. It looks like AddressSanitizer can't work if both fork() and threads are used, see the following bug report for details

[https://github.com/google/sanitizers/issues/255](https://github.com/google/sanitizers/issues/255)

## Limitations

Even if AddressSanitizer is a well-known good tool, it's not a silver bullet. First, you still need use your brain to avoid common issues in C/C++ code like buffer overflows, use-after-free and other issues. And then, you may want to be armed with a good set of tools like runtime memory checkers and static code analyzers.

## Next steps

Greg Steuck suggested that would be cool if Java runtime compilers inserted AddressSanitizer checks to generated code. I am not an expert in runtime compilation. and talked to Igor Ignatyev from [Hotspot Group](http://openjdk.java.net/census#hotspot). He said it might be hard to do that in C1/C2 compilers, but it might be easier to introduce such checks with Graal compiler.

Have fun!

