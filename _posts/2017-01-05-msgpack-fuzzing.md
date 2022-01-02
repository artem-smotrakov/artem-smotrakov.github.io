---
layout: post
title: MessagePack fuzzing
date: 2017-01-05 10:55:29.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Fuzzing
- Open Source
- Security
permalink: "/en/security/msgpack-fuzzing.html"
---
[MessagePack](http://msgpack.org/)&nbsp;is a binary serialization format. There are lots of open source implementations of this protocol on various languages including C/C++. It's good to do something good in new year.&nbsp;For example, it can be a little&nbsp;contribution to an open source project. Let's check quickly if the implementation on C/C++ has any memory corruption issues. One of the best ways is of course fuzzing.

![]({{ site.baseurl }}/assets/images/2017/01/msgpack_fuzzing_afl-300x202.png)



As you can see on the picture above, we are going to use American Fuzzy Lop (AFL) because it's just a great tool. First, we need to clone the repo, and build msgpack with AFL. We are going to use AddressSanitizer (ASan) - it's another great tool. If you have not heard about it - it's a fast and cool memory checker which can detect issues like read/write buffer overflows, use-after-free, etc.

# Fuzzing MessagePack with American Fuzzy Lop

Here are commands which builds msgpack instrumented with AFL and ASan:

```
hg clone https://github.com/msgpack/msgpack-c
cd msgpack-c
AFL_USE_ASAN=1 CC=afl-gcc cmake \
    -DCMAKE_INSTALL_PREFIX=/path/to/build \
    -DCMAKE_C_FLAGS=-m32 \
    .
AFL_USE_ASAN=1 CC=afl-gcc make
make install
```

It's going to install msgpack binaries to /path/to/build you specified in -DCMAKE\_INSTALL\_PREFIX option. You need to specify a real path here. You might notice that it uses -m32 option which tells GCC to build a 32-bit binaries. At first, I tried to run fuzzing with with 64-bit binaries, but AFL and ASan complained about memory, but it worked fine with 32-bit.

Once&nbsp;we built msgpack, we need to create a couple of files with serialized data for fuzzing. I used the following program for that:

```
#include 
#include 
#include 

int main(int argc, char **argv) {
    if (argc < 2) {
        fprintf(stdout, "no case specified\n");
        return -1;
    }

    if (strcmp(argv[1], "1") == 0) {
        /* creates buffer and serializer instance. */
        FILE *fp = fopen("./1.ser", "w+");
        msgpack_packer pk;
        msgpack_packer_init(&pk, fp, msgpack_fbuffer_write);

        /* serializes ["Hello", "MessagePack"]. */
        msgpack_pack_array(&pk, 2);
        msgpack_pack_bin(&pk, 5);
        msgpack_pack_bin_body(&pk, "Hello", 5);
        msgpack_pack_bin(&pk, 11);
        msgpack_pack_bin_body(&pk, "MessagePack", 11);

        fclose(fp);
    } else if (strcmp(argv[1], "2") == 0) {
        /* creates buffer and serializer instance. */
        FILE *fp = fopen("./2.ser", "w+");
        msgpack_packer pk;
        msgpack_packer_init(&pk, fp, msgpack_fbuffer_write);

        msgpack_pack_array(&pk, 3);
        msgpack_pack_int(&pk, 1);
        msgpack_pack_true(&pk);
        msgpack_pack_str(&pk, 7);
        msgpack_pack_str_body(&pk, "example", 7);

        fclose(fp);
    } else if (strcmp(argv[1], "3") == 0) {
        FILE *fp = fopen("./3.ser", "w+");
        msgpack_packer pk;
        msgpack_packer_init(&pk, fp, msgpack_fbuffer_write);

        char const buf[] = { 2 };
        char const* str_0 = NULL;
        unsigned int str_0_size = 0;
        char str_1[] = "ololo";
        unsigned int str_1_size = sizeof(str_1);
        unsigned int map_size = 2;

        msgpack_pack_array(&pk, 20);
        msgpack_pack_float(&pk, 0.0);
        msgpack_pack_float(&pk, 1.2);
        msgpack_pack_float(&pk, -1.3);
        msgpack_pack_double(&pk, 0.0);
        msgpack_pack_double(&pk, 1.2);
        msgpack_pack_double(&pk, -1.3);
        msgpack_pack_nil(&pk);
        msgpack_pack_map(&pk, 0);
        msgpack_pack_true(&pk);
        msgpack_pack_false(&pk);
        msgpack_pack_int(&pk, 10);
        msgpack_pack_int(&pk, -10);
        msgpack_pack_int(&pk, 0xffffffff);
        msgpack_pack_v4raw(&pk, str_0_size);
        msgpack_pack_v4raw_body(&pk, str_0, str_0_size);
        msgpack_pack_v4raw(&pk, str_1_size);
        msgpack_pack_v4raw_body(&pk, str_1, str_1_size);
        msgpack_pack_ext(&pk, sizeof(buf), 1);
        msgpack_pack_ext_body(&pk, buf, sizeof(buf));
        msgpack_pack_str(&pk, 7);
        msgpack_pack_str_body(&pk, "example", 7);
        msgpack_pack_bin(&pk, 11);
        msgpack_pack_bin_body(&pk, "MessagePack", 11);
        msgpack_pack_array(&pk, 0);
        msgpack_pack_map(&pk, map_size);
        msgpack_pack_true(&pk);
        msgpack_pack_false(&pk);
        msgpack_pack_int(&pk, 10);
        msgpack_pack_int(&pk, -10);

        fclose(fp);
    } else {
        fprintf(stdout, "unexpected case: %s\n", argv[1]);
    }
}
```

Then, we need to compile it with AFL as well, and finally run it to generate serialized data:

```
AFL_USE_ASAN=1 \
    afl-gcc -m32 -g serialize.c -o serialize \
        -I/path/to/build/include \
        -L/path/to/build/lib \
        -l:libmsgpackc.a
./serialize 1
./serialize 2
./serialize 3
```

Next, we need to create a simple program which uses msgpack API for deserialization. Finally, we are going to run this program while fuzzing. I wrote two programs. First one uses C API:

```
#include 
#include 

static const int BUFFERSIZE = 1024;

int main(int argc, char **argv) {
    if(argc < 2) {
        fprintf(stdout, "no file specified\n");
        return 1;
    }

    char inbuffer[BUFFERSIZE];

    FILE *fp = fopen(argv[1], "rb");
    size_t off = 0;
    size_t read = 0;
    msgpack_unpacked unpacked;
    msgpack_unpacked_init(&unpacked);

    read = fread(inbuffer, sizeof(char), BUFFERSIZE, fp);
    if (msgpack_unpack_next(&unpacked, inbuffer, read, &off) == 2) {
        msgpack_object_print(stdout, unpacked.data);
        fprintf(stdout, "\n");
    } else {
        fprintf(stdout, "failed\n");
    }

    fclose(fp);
    msgpack_unpacked_destroy(&unpacked);

    return 0;
}
```

And second&nbsp;one uses C++ API (it's much shorter, by the way):

```
#include 
#include 
#include 
#include 

int main(int argc, char **argv) {
    if(argc < 2) {
        std::cout << "no file specified" << std::endl;
        return 1;
    }

    std::ifstream ifs(argv[1]);
    std::stringstream buffer;
    buffer << ifs.rdbuf();
    msgpack::unpacked upd = msgpack::unpack(buffer.str().data(), buffer.str().size());
    std::cout << upd.get() << std::endl;
}
```

Now we are good to go and start fuzzing. We should create "in" directory for input data for AFL, and copy all three \*.ser files to there. Here is a command which runs fuzzing for msgpack implementation on C (just replace "deserialize" with "deserialize\_cpp" to run it for C++ implementation):

```
export ASAN_OPTIONS="detect_leaks=0"
/home/artem/tools/afl/afl-fuzz \
    -m 5000 -t 2000 -i in -o out \ 
        ../deserialize @@
```

I was not interested in memory leak, so I passed "detect\_leaks=0" option to AddressSanitizer. But someone may try to check msgpack for memory leaks.

I ran fuzzing on my old laptop, and it took a couple of days to find issues. So, the lesson here is that you have to be patient when you fuzz software. Or, you need faster hardware.

And now the most exciting part.

# Bugs in MessagePack

&nbsp;Fuzzing discovered a couple of integer overflows which lead to buffer overflows.

One problem was in template\_callback\_ext() function. While parsing corrupted data, it's possible that zero is passed to "l" parameter of template\_callback\_ext() function. This leads to assigning a negative value to o-\>via.ext.size which is implicitly being converted to a huge positive number.

```
static inline int template_callback_ext(unpack_user* u, const char* b, const char* p, unsigned int l, msgpack_object* o)
{
    MSGPACK_UNUSED(u);
    MSGPACK_UNUSED(b);
    o->type = MSGPACK_OBJECT_EXT;
    o->via.ext.type = *p;
    o->via.ext.ptr = p + 1;
    o->via.ext.size = l - 1;
    u->referenced = true;
    return 0;
}
```

It may then lead to reading o-\>via.ext.ptr out of its bounds, for example, in msgpack\_object\_bin\_print() function:

```
static void msgpack_object_bin_print(FILE* out, const char *ptr, size_t size)
{
    size_t i;
    for (i = 0; i < size; ++i) {
        if (ptr[i] == '"') {
            fputs("\\\"", out);
        } else if (isprint(ptr[i])) {
            fputc(ptr[i], out);
        } else {
            fprintf(out, "\\x%02x", (unsigned char)ptr[i]);
        }
    }
}
```

Depending on where this function is called, and what "out" is, it may potentially lead to reading confidential data, and sending it to untrusted party. It's a read buffer overflow, similar to Heartbleed.

Next issue was in template\_callback\_array() and template\_callback\_map() functions.&nbsp;An integer overflow may occur there&nbsp;while calculating an amount of memory to be allocated. It may potentially lead to allocating zero bytes to o-\>via.map.ptr:

```
static inline int template_callback_array(unpack_user* u, unsigned int n, msgpack_object* o)
{
    o->type = MSGPACK_OBJECT_ARRAY;
    o->via.array.size = 0;
    o->via.array.ptr = (msgpack_object*)msgpack_zone_malloc(u->z, n*sizeof(msgpack_object));
    if(o->via.array.ptr == NULL) { return -1; }
    return 0;
}
```

As a result, it may lead to a buffer overflow, for example, in template\_callback\_map\_item() and template\_callback\_array\_item() functions:

```
static inline int template_callback_array_item(unpack_user* u, msgpack_object* c, msgpack_object o)
{
    MSGPACK_UNUSED(u);
#if defined( __GNUC__ ) && !defined( __clang__ )
    memcpy(&c->via.array.ptr[c->via.array.size], &o, sizeof(msgpack_object));
#else /* __GNUC__ && ! __clang__ */
    c->via.array.ptr[c->via.array.size] = o;
#endif /* __GNUC__ && ! __clang__ */
    ++c->via.array.size;
    return 0;
}
```

Finally, there were a couple of similar integer overflows in include/msgpack/v2/unpack.hpp

My patches were accepted with minor modifications.

- [https://github.com/msgpack/msgpack-c/pull/547](https://github.com/msgpack/msgpack-c/pull/547)
- [https://github.com/msgpack/msgpack-c/pull/550](https://github.com/msgpack/msgpack-c/pull/550)

Thanks to&nbsp;[Takatoshi Kondo](https://github.com/redboltz)&nbsp;who integrated the patches quickly!

