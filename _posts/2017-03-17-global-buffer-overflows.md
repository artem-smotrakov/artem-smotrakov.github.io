---
layout: post
title: Global buffer overflow
date: 2017-03-17 17:48:35.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Security
meta:
  _edit_last: '1'
  mask_links: default
  _wpgo_column_layout_save: ''
  _wpgo_column_layout: default
  _syntaxhighlighter_encoded: '1'
  _aioseop_opengraph_settings: a:14:{s:32:"aioseop_opengraph_settings_title";s:0:"";s:31:"aioseop_opengraph_settings_desc";s:0:"";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}
  _aioseop_keywords: security,global,buffer,overflow,stack,heap,example,mitigation,exploit
  _aioseop_description: 'There are a lot of articles about buffer overflows on stack
    and heap, but not that many about global buffer overflows: examples, exploitation,
    and mitigation.'
  _aioseop_title: Global buffer overflows
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: ''
  _yoast_wpseo_content_score: '60'
  _yoast_wpseo_focuskw_text_input: buffer overflow
  _yoast_wpseo_focuskw: buffer overflow
  _yoast_wpseo_linkdex: '86'
  _yoast_wpseo_metadesc: There are a lot of articles, posts, and even books which
    describe a stack buffer overflow. There are a little less stuff about heap buffer
    overflows. But there is one more thing which you can overflow - buffers in global
    memory. Although all of those types of issues are very similar, let me try to
    fill this little gap with global buffer overflows.
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:0;s:7:"replies";i:0;s:7:"authors";i:0;s:14:"recent_authors";a:0:{}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617984221'
permalink: "/en/security/global-buffer-overflows.html"
---
<p style="text-align: justify;">There are a lot of articles, posts, and even books which describe a stack buffer overflow. There are a little less stuff about heap buffer overflows. But there is one more thing which you can overflow - buffers in global memory. Although all of those types of issues are very similar, let me try to fill this little gap with global buffer overflows.</p>
<p><a href="/ru/security-ru/global-buffer-overflow-ru.html">Русская версия</a></p>
<p><!--more--></p>
<h2>Where do global buffers live?</h2>
<p style="text-align: justify;">There are two places where global and static variables can be located:</p>
<ul style="list-style-type: square;">
<li>segment for initialized data</li>
<li>segment for uninitialized data (BSS segment)</li>
</ul>
<p style="text-align: justify;">Those variables which don't have an explicit initializer go to BSS segment, and are initialized automatically with zeros.</p>
<p>Here is how memory looks like:</p>
<pre class="console">         high address
+-----------------------------+
| command line arguments and  |
| environment variables       |
+-----------------------------+
|           stack             |
+-------------+---------------+
|             |               |
|             V               |
|                             |
|                             |
|                             |
|                             |
|             ^               |
|             |               |
+-------------+---------------|
|            heap             |
+-----------------------------+
|   uninitialized data (BSS)  |
|    (initialized to zero)    |
+-----------------------------+
|       initialized data      |
+-----------------------------+
|            text             |
+-----------------------------
+ low address

## An example of a global buffer overflow

Here is a very simple example of a global buffer overflow:

<script src="https://gist.github.com/artem-smotrakov/6d0c774621afcfc3b2b1c72bf7cca549.js"></script>

## How can I exploit a global buffer overflow?

The answer is the usual - it depends. More precisely, it depends on what kind of data is stored in memory which you are able to overwrite or overread. Here&nbsp;are a couple of examples.

### Overwriting sensitive data which may lead to security implications

Here is a simple program which takes a pass phrase, and prints a secret phrase if the password is correct:

<script src="https://gist.github.com/artem-smotrakov/218e8b185c5e2d2a1d73610bc657e317.js"></script>

`strcpy` call overflows `buffer` if the password is more than 15 symbols (`strcpy` adds \0 to the end of the string). As a result, we can overwrite `access` flag:

(Let's use Python to generate some strings)

```
$ gcc -g gbo.c -o gbo
$ ./gbo wrong
access denied
$ ./gbo `python3 -c "print('x' * 16 + 'y')"`
this is a secret
```

It happens because both `buffer` and `access` variables are located in the segment for uninitialized data, and `access` follows `buffer` in the memory. But actual location of those variables may depend because the order of global variables in memory is undefined. So the code above may not be vulnerable.  
There is an interesting detail - the issue will disappear if we initialize `access` flag with a value when we declare it:

```
char access = 'n';
```

In this case, compiler puts `access` to the segment for initialized data which normally goes before the segment for uninitialized data. As a result, overflowing `buffer` won't overwrite `access` flag because the address of `buffer` is higher.

### Overwriting objects on the heap, or just crashing the application

Normally the heap starts somewhere after data and BSS segments. But the actual address may vary. Let's consider the following code:

<script src="https://gist.github.com/artem-smotrakov/9638c9a39e6968f19481d8ccf67aef5a.js"></script>

It defines a global buffer `buffer` and a buffer `allocated` on the heap. Then, it copies "test" string to `allocated` buffer. Next, it prints out addresses and content of the buffers. Finally, it copies first command line option to the global buffer, and prints out `allocated` again.  
On my laptop with Linux, the following command causes a segfault:

```
$ ./gbo `python3 -c "print('x' * 2**12)"`
buffer address = 0x601070
allocated address = 0x773010
allocated address - buffer address = 1515424
allocated (before) = test
Segmentation fault (core dumped)
```

Here we are trying to put a string of 4096 symbols 'x' to `buffer` which overflows it. `allocated` address is 0x773010, but it changes every time when the program runs because of dynamic memory allocation. Note that the difference between addresses is much bigger (1515424) than 4096. As a result, it crashes because we're trying to write to an invalid address. It doesn't look to be possible to overwrite objects on the heap if we can overflow a global buffer. But it may be possible just to crash an application.

### Overwriting a function pointer in global memory

A function pointer contains an address to a function. A function pointer can be used to call a function. Pretty straightforward. Here is a simple example with overwriting a function pointer:

<script src="https://gist.github.com/artem-smotrakov/b5ec30cd841cac298d60fbf39e43a5b7.js"></script>

The program is similar to the example above with `access` flag, but instead of setting a flag it plays with a function pointer. First, the function pointer is not initialized. Next, it puts an address of `do_something` to `func` pointer. If specified password is correct, it puts an address of `print_secret` function to `func` pointer. Finally, it calls a function which address was placed to `func`.

`strcpy` call overflows `buffer` if specified command line option is more than 15 symbols (`strcpy` adds \0 to the end of the string). Since both `func` and `buffer` are not initialized, they live in the same data segment for uninitialized data. As a result, we can overwrite&nbsp;function pointer `func`:

```
$ gcc -g gbo.c -o gbo
$ ./gbo `python3 -c "print('w' * 256)"`
Segmentation fault (core dumped)
```

We just overwrote the function pointer `func` with 0x77777777 address (0x77 is ASCII code of 'w'). Then, the program tried to call a function on this address. Since the function pointer points to an invalid address, it resulted to a segfault. But just a crash is not fun. It's more interesting to make the program run what we want. Let's make it call `print_secret` function even if we pass a wrong password. First, we should figure out what the address of `print_secret` function is. GDB can help us here:

```
$ gdb --args ./gbo test
Reading symbols from ./gbo...done.
(gdb) break gbo.c:36
Breakpoint 1 at 0x40068c: file gbo.c, line 36.
(gdb) run
Starting program: /home/artem/tmp/gbo test

Breakpoint 1, main (argc=2, argv=0x7fffffffdcf8) at gbo.c:36
36 func();
(gdb) p func
$1 = (void (*)(void)) 0x7777777777777777
(gdb) p print_secret 
$2 = {void (void)} 0x400607 
(gdb) quit
```

Now we know the address of `print_secret` function - it is 0x400607. Then we need to pass such a string to the program, so that it overwrites `func` pointer with 0x400607. We need to take into account the following:

- We need to write 16 bytes first to fill out `buffer`
- We need to remember that we are on 64-bit system (in my case), so we need to write 8 bytes to overwrite `func` pointer
- We need to remember that we are on little-endian system (in my case), so 0x400607 should be reversed - 0x070640

The following command makes the program to call `print_secret` function without passing a correct password:

```
$ ./gbo `python3 -c "print('w' * 16 + '\x00\x00\x00\x00\x00\x07\x06\x40')"`
this is a secret
```

### Reading sensitive data

Everybody knows about Heartbleed bug in OpenSSL. This is just a great example of buffer overread vulnerability which means that an attacker can read a buffer out of its bounds ("overflow" sounds a bit confusing here). In case of Heartbleed, an attacker could read sensitive data from the heap. But it's also possible to read data from global buffers which may also contain sensitive information. Here is a very simple example of a vulnerable program:

<script src="https://gist.github.com/artem-smotrakov/7fc2d7390f99974b6a663d77d198a739.js"></script>

The program above accepts a number of symbols which should be printed out. It copies specified number of bytes from global buffer `public` to local buffer `buffer`. Then, it prints out all strings in `buffer`. If a number of requested symbols are more than length of `public`, then the program will read `public` buffer out of its bounds. It results to reading `secret` buffer which follows `public` in the data segment for uninitialized data. As a result, the content of `secret` is printed out as well.

## Mitigation

Same rules apply. As you can see, there is no much difference between overflows in stack, heap and global memory. Developers should be careful while working with memory. Using a brain and being careful should help to avoid introducing memory corruption issues. Appropriate deadlines and not being in a rush should also help (dear managers, you can help your developers here). Code review, tools for static and dynamic analysis should help to catch issues in time.

