---
layout: post
title: Fuzzing and code coverage analysis
date: 2018-06-13 19:31:55.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Fuzzing
- picotls
- Security
- TLS
- TLS 1.3
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '90'
  _yoast_wpseo_primary_category: '157'
  _oembed_b4feb9b14bcc99590e6d53501a681afa: "{{unknown}}"
  _oembed_95a95033c4b79d6b2f122d49143fbb8b: "{{unknown}}"
  _oembed_27e9c6708601ed072f2ac07a2f1823a7: "{{unknown}}"
  _oembed_3bad1234a550d185dc39a8c130bd7bc1: "{{unknown}}"
  _oembed_cb87415dce53ae085e186b3633c32a42: "{{unknown}}"
  _yoast_wpseo_focuskw_text_input: fuzzing
  _yoast_wpseo_focuskw: fuzzing
  _yoast_wpseo_metadesc: Code coverage analysis is used in testing to discover untested
    pieces of an application. Gathering code coverage data may also be useful for
    fuzzing. Basically it may help to figure out which parts of a program were not
    reached during fuzzing. This info can then be used to improve the fuzzer.
  _yoast_wpseo_linkdex: '69'
  rp4wp_auto_linked: '1'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:1;s:7:"replies";i:1;s:7:"authors";i:2;s:14:"recent_authors";a:2:{i:0;O:8:"stdClass":3:{s:20:"comment_author_email";s:25:"artem.smotrakov@gmail.com";s:14:"comment_author";s:5:"Artem";s:7:"user_id";s:1:"1";}i:1;O:8:"stdClass":3:{s:20:"comment_author_email";s:24:"igor.ignatyev@oracle.com";s:14:"comment_author";s:4:"Igor";s:7:"user_id";s:1:"0";}}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617986433'

permalink: "/en/security/fuzzing-code-coverage-analysis.html"
---
Code coverage analysis is used in software testing to discover untested pieces of an application. Gathering code coverage data may also be useful for fuzzing. Basically it may help to figure out which parts of a program were not reached during fuzzing. This info can then be used to improve the fuzzer.

Let's try to gather some code coverage data during fuzzing. As an example, we're going to test [picotls](https://github.com/h2o/picotls) with [tlsbunny](https://github.com/artem-smotrakov/tlsbunny). Picotls is an implementation of TLS 1.3 protocol written in C, and tlsbunny&nbsp;is a framework for building negative tests and fuzzers for TLS 1.3 implementations. We're going to use [gcov](https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/Gcov.html) for gathering code coverage data, and [lcov](http://ltp.sourceforge.net/coverage/lcov.php) for creating a report.



## Instrumenting an application with gcov

Gcov works together with gcc compiler. It can be easily enabled by passing `-fprofile-arcs -ftest-coverage` flags to gcc. You only need to make sure that you passed the flags to all gcc invocations during building your application. Typically, the flags can be added to `CFLAGS` or similar environment variables.

Picotls uses CMake for building itself. The following simple script clones and builds picotls:  
<script src="https://gist.github.com/artem-smotrakov/54bd586cde7d6a99b69f781835873e23.js"></script>  
Here we also enabled debug info, disabled some optimizations, and turned on AddressSanitizer which can help to catch various memory corruption issues.

For testing, we are going to use `cli`&nbsp;program which can start a simple TLS 1.3 server based on picotls. The sources of `cli`&nbsp;can be found at `t/cli.c`.

Gcov normally dumps code coverage data when a program finishes. `cli` is a very simple server which doesn't provide a graceful way to stop itself. We just have to kill the process. But in this case Gcov won't be able to dump code coverage data. The following patch for `t/cli.c`&nbsp;helps to overcome this problem:

<script src="https://gist.github.com/artem-smotrakov/ada29fd193cf3805c484fe3359c88b97.js"></script>

The patch adds a&nbsp; handler for `SIGINT`&nbsp;signal. The handler calls `__gcov_flush()`&nbsp;function which dumps gathered code coverage data. Now we can just stop `cli`&nbsp;process by `Ctrl-C`, and code coverage data will be dumped by the signal handler.

Don't forget to re-build picotls after applying the patch.

## Initializing lcov

Gcov doesn't provide a good human-readable report. Once your instrumented application finishes, gcov is going to store data to&nbsp;`.gcno`&nbsp;files for each source file. The data is stored in a binary format.&nbsp;&nbsp;To make it a bit more readable, you can call `gcov`&nbsp;for a source file you are interested in, for example:

```
$ gcov -l -o CMakeFiles/cli.dir/t/cli.c.gcno t/cli.c
```

The command above creates several `.gcov`&nbsp;files which contain code coverage info in more readable format:

```
$ tail -10 cli.c##cli.c.gcov
        -: 420:
        1: 421: if (resolve_address((struct sockaddr *)&sa, &salen, host, port, family, SOCK_STREAM, IPPROTO_TCP) != 0)
    #####: 422: exit(1);
        -: 423:
        1: 424: if (is_server) {
        1: 425: return run_server((struct sockaddr *)&sa, salen, &ctx, file, &hsprop);
        -: 426: } else {
    #####: 427: return run_client((struct sockaddr *)&sa, salen, &ctx, host, file, &hsprop);
        -: 428: }
        -: 429:}
```

You may have noticed that gcov marked uncovered lines with `#####`. Technically, you can use `grep` to find all untested lines:

```
$ find . -name "*.gcov" | xargs grep "#####"
```

But luckily we have lcov which can transform `.gcov`&nbsp;files in to an HTML report . Once we built our application with gcov, we can initialize lcov:

```
$ lcov --zerocounters --directory .
$ lcov --no-external --capture --initial \
	--directory . --output-file base.info
```

The commands above create a "baseline" that contains zero coverage.

## Running a picotls server and starting a fuzzer

Now it's time to run fuzzing. First, let's start a local picotls server:

```
$ echo "I am a picotls server" > message
$ ./cli -c certs/server_cert.pem -k certs/server_key.pem -i message 0.0.0.0 20101
```

You can get certificates and keys in [tlsbunny repository](https://github.com/artem-smotrakov/tlsbunny/tree/master/certs). Then, let's run a simple fuzzer for TLSPlaintext structure:

```
$ git clone https://github.com/artem-smotrakov/tlsbunny-and-docker
$ cd tlsbunny-and-docker
$ git checkout 561df07499ced89bf0e659754b0b3428a4ffd2c2
$ javac -d classes -classpath src/main/java \
      src/main/java/com/gypsyengineer/tlsbunny/tls13/test/picotls/client/FuzzyClient.java
$ java -classpath classes \
       -Dtlsbunny.port=20101 \
       -Dtlsbunny.threads=1 \
       -Dtlsbunny.target.filter=tls_plaintext \
                com.gypsyengineer.tlsbunny.tls13.test.picotls.client.FuzzyClient
```

Fuzzing is not going to be fast because the fuzzer talks to the server via a socket. Furthermore, the fuzzer can set&nbsp;`TLSPlaintext.length` field greater than the actual size of content of the TLSPlaintext structure. As a result, the server can wait for additional data which in the end results to a timeout.

After the fuzzer is finished, we can stop the server:

```
$ kill -SIGINT `pidof cli`
```

## Generating an HTML report with lcov

Once fuzzing is finished, we can ask lcov to generate an HTML report for us:

```
$ mkdir cc_report
$ lcov --no-external --capture \
	--directory . --output-file test.info
$ lcov --add-tracefile base.info --add-tracefile test.info \
    --output-file total.info
$ genhtml --ignore-errors source total.info --legend --title picotls \
    --output-directory=cc_report
```

First two `lcov` calls prepare code coverage data for the report and merge it with the baseline we previously created. Finally, `genhtml` creates an HTML report which will be stored in `cc_report`&nbsp;directory.

The report showed an expected result: such a simple fuzzer didn't cover much. Uncovered functions in the report may give an idea which TLS structures may be also fuzzed to improve coverage. For example, `server_handle_certificate_verify` function from `lib/picotls.c` was not covered which means that we may consider implementing a fuzzer for client authentication.

## Conclusion

Code coverage info may be helpful in fuzzing. But if you were able to get 100% coverage in your code coverage report, it doesn't mean that you have tested everything, and your software is free of bugs. Reaching all code during testing may not be enough. It would be really nice to reach all code with all possible memory states. Unfortunately, a code coverage report doesn't show this kind of information.

Traditionally, fuzzing is used for testing applications written in C/C++ to catch possible memory corruption issues which most likely are going to have security implications. But it doesn't mean that fuzzing can't be used for testing applications written in other languages. Even if a language (such Java) doesn't allow to access memory directly, fuzzing can still uncover bugs. For example, fuzzing may uncover an unexpected runtime exception which occurs during processing incorrect data.&nbsp;No matter which language is used, a good application should properly handle incorrect data and react with an expected action, for example, by throwing a documented exception. An unexpected behavior in processing incorrect data may still have security implications.

## Links

Here are a couple of useful articles which helped me a lot:

- [gcov/lcov - C & Linux based Dynamic Code Coverage Tools](http://techvolve.blogspot.com/2014/03/gcovlcov-c-linux-dyanmic-code-coverage.html)
- [LCOV Code Coverage](https://wiki.documentfoundation.org/Development/Lcov)

Enjoy!

