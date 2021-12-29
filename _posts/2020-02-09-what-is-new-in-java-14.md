---
layout: post
title: What is new in Java 14?
date: 2020-02-09 19:53:36.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tech
tags:
- GC
- Java
- JEP
- OpenJDK
meta:
  _edit_last: '1'
  _yoast_wpseo_primary_category: '258'
  _yoast_wpseo_content_score: '60'
  _yoast_wpseo_focuskw: Java 14
  _yoast_wpseo_metadesc: 'Let’s take a closer look at the major updates in Java 14:
    new switch expressions, better NullPointerExceptions, improvements in garbage
    collection, JFR event streaming and more.'
  _yoast_wpseo_linkdex: '73'
  rp4wp_auto_linked: '1'
  _stcr@_tech.meshter@gmail.com: 2020-03-24 21:42:48|Y
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617976825'

permalink: "/en/tech/what-is-new-in-java-14.html"
---
<!-- wp:paragraph -->

Java 14 is going to be released on Mar 17th, 2020. Besides ~2400 bug fixes and small enhancements, the new version of Java contains 16 major enhancements which are also called JEPs (Java Enhancement Proposals).

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Let’s take a closer look at the major updates in Java 14: new switch expressions, better NullPointerExceptions, improvements in garbage collection, JFR event streaming and more.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

(the article has been published on [Medium](https://medium.com/better-programming/whats-new-in-java-14-a472ec291c05))

<!-- /wp:paragraph -->

<!-- wp:image {"id":3745,"sizeSlug":"large","className":"noborder"} -->

![What is new in Java 14]({{ site.baseurl }}/assets/images/2020/02/cowsay-what-is-new-in-Java-14.jpg)

<!-- /wp:image -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## Switch Expressions

<!-- /wp:heading -->

<!-- wp:paragraph -->

This update to the Java language was already available in Java 12 and 13 but only as a preview language feature which means it was not enabled by default. Finally, the new switch expressions are released in Java 14. Making a long story short, Java 14 introduces a new simplified form of a switch block with `case L -> ...` labels. The new switch expressions may help to simplify code in some cases. Here are a couple of examples.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Let's assume that we have a enum that describes weekdays. Then, we can write the following code using the new switch expressions:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
switch (day) {
    case MONDAY -> System.out.println("Aweful");
    case TUESDAY, WEDNESDAY -> System.out.println("Okay");
    case THURSDAY -> System.out.println("Good");
    case FRIDAY -> System.out.println("Great");
    case SATURDAY, SUNDAY -> System.out.println("Awesome");
}
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Here we just used a single expression for each `case`. Note that the `switch` block doesn't use any `break` statement which makes it much shorter. The next example shows how the new switch expressions can return a value:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    case THURSDAY, SATURDAY -> 8;
    case WEDNESDAY -> 9;
};
```

<!-- /wp:code -->

<!-- wp:paragraph -->

It's also possible to write multi-line blocks and return a value with a new keyword `yield`:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
int result = switch (s) {
    case "Foo" -> 1;
    case "Bar" -> 2;
    default -> {
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
    }
};
```

<!-- /wp:code -->

<!-- wp:paragraph -->

There are several important things that you need to keep in mind when you use the new switch expressions. For example, the cases of new switch expressions have to be exhaustive. It means that for all possible values there has to be a matching switch label. Or, since `yield` is now a keyword, a class with a name `yield` now becomes illegal in Java 14. If you'd like to learn more about the new switch expressions, you're welcome to read [JEP 361](https://openjdk.java.net/jeps/361). The authors provided quite a lot of useful info about the new switch expressions.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Helpful NullPointerExceptions

<!-- /wp:heading -->

<!-- wp:paragraph -->

The JVM throws a `NullPointerException` (NPE) when code tries to dereference a null reference. All Java developers have seen them before. For example, the following code may result in an NPE:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
foo.bar = 10;
```

<!-- /wp:code -->

<!-- wp:paragraph -->

The NPE is going to look like the following:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
Exception in thread "main" java.lang.NullPointerException
    at App.main(App.java:17)
```

<!-- /wp:code -->

<!-- wp:paragraph -->

The exception message contains a filename and a line where the null dereference happened. For the `foo.bar = 10;` statement, it's not too difficult to figure out that the NPE was thrown because the `foo` variable was null. Unfortunately, sometimes it not clear what exactly causes an NPE. For example, if either `a`, `b` or `c` is null, then an NPE is going to be thrown:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
a.b.c.d = 42;
```

<!-- /wp:code -->

<!-- wp:paragraph -->

However, no matter which field was null, the NPE is going to look the same. It doesn't give any clue which filed was actually null. Here is another example. If one of the nested arrays is null, it results in an NPE:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
a[i][j][k] = 99;
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Again, no matter which array was null, the NPE is going to look the same.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Java 14 addresses this problem and makes NPEs more friendly. Now the JVM can figure out which variable was null, and then it lets the user know about it in the exception message. For example, a null dereference in the line `foo.bar = 10;` is going to result in the following NPE:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
Exception in thread "main" java.lang.NullPointerException: 
        Cannot assign field "bar" because "foo" is null
    at App.main(App.java:17)
```

<!-- /wp:code -->

<!-- wp:paragraph -->

A null dereference in the like `a.b.c.d = 41;` is going to result in the following NPE if `a.b` was null:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
Exception in thread "main" java.lang.NullPointerException: 
        Cannot read field "c" because "a.b" is null
    at App.main(App.java:17)
```

<!-- /wp:code -->

<!-- wp:paragraph -->

The new info in NullPointerExceptions may be very helpful in analyzing its root cause and make the developer's life a bit easier. By the way, the improvement has been available in SAP's JVM since 2006. Unfortunately, it took 14 years to finally bring it into OpenJDK.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

If you're interested in the details, the authors of the [JEP 358](https://openjdk.java.net/jeps/358) provided a lot of info about the new feature.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Packaging Tool (Incubator)

<!-- /wp:heading -->

<!-- wp:paragraph -->

Currently, a Java application is usually distributed as a simple JAR file. However, it's not very convenient especially for a user of the application. It would be much better if the Java application were an installable package such as MSI on Windows or DMG on Mac. This would allow Java applications to be distributed, installed, and uninstalled in a way that is familiar to users.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[JEP 343](https://openjdk.java.net/jeps/343) introduces the `jpackage` tool which packages a Java application into a platform-specific package that includes all of the necessary dependencies. Here is a list of supported package formats:

<!-- /wp:paragraph -->

<!-- wp:list -->

- DEB and RPM on Linux
- PKG and DMG on macOS
- MSI and EXE on Windows

<!-- /wp:list -->

<!-- wp:paragraph -->

Here is an example of how the new tool can be used:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
$ jpackage --name myapp --input lib --main-jar main.jar \
  --main-class myapp.Main
```

<!-- /wp:code -->

<!-- wp:paragraph -->

It takes the `lib/main.jar` file and produces a package in the format most appropriate for the system on which it is run. The entry point is the `myapp.Main` class.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The JEP's authors provided quite a lot of useful [information about the new tool](https://openjdk.java.net/jeps/343).

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Although the `jpackage` tool is available in JDK 14, it's delivered as an incubator module which means that the functionality is not guaranteed to be stable and may be revised in a future release.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Better garbage collection

<!-- /wp:heading -->

<!-- wp:paragraph -->

Java 14 contains multiple enhancements in garbage collection.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[JEP 345](https://openjdk.java.net/jeps/345) improves the G1 garbage collector by implementing NUMA-aware memory allocation. By the way, NUMA stands for Non-Uniform Memory Access. This feature has been implemented in the parallel garbage collector for a long time. Now it can be enabled in the G1 as well by running java with a new `+XX:+UseNUMA` command-line option. This should improve G1 performance on large machines.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[JEP 363](https://openjdk.java.net/jeps/363) removes the Concurrent Mark Sweep (CMS) garbage collector which was deprecated a couple of years ago. Goodbye CMS!

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[JEP 364](https://openjdk.java.net/jeps/364) and [JEP 365](https://openjdk.java.net/jeps/365) make the Z garbage collector (ZGC) available on macOS and Windows. ZGC is a concurrent garbage collector which was added to the JVM a couple of years ago. ZGC tries to reduce pause times for garbage collections and can handle heaps of size ranging from a few hundred megabytes to multi terabytes. Previously, the collector could run only on Linux.

[JEP 366](https://openjdk.java.net/jeps/366) deprecates the combination of the Parallel Scavenge and Serial Old garbage collection algorithms. This combination had to be enabled by the user with the `-XX:+UseParallelGC -XX:-UseParallelOldGC` command-line options. The authors believe that the combination is uncommon but requires a significant amount of maintenance effort. In fact, the option `-XX:UseParallelOldGC` is now deprecated. A warring is going to be displayed if the deprecated modes are used.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JFR Event Streaming

<!-- /wp:heading -->

<!-- wp:paragraph -->

JDK Flight Recorder (JFR) is an event recorder that is built into the JVM. It captures diagnostic and profiling data about the JVM itself, and the application running in the JVM. JFR used to be a proprietary tool but it was open-sourced in 2018 Java released as part of OpenJDK 11.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

To consume the data provided by JFR, a user has to start a recording, stop it, dump the contents to disk and then parse the recording file. This works quite well for application profiling but not for monitoring purposes.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

In Java 14, the JFR allows users to subscribe to events asynchronously. Users can now register a handler which is going to be invoked in response to the arrival of an event. The `RecordingStream` class provides a uniform way to filter and consume events. Here is an example provided by the JEP's authors:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
try (var rs = new RecordingStream()) {
  rs.enable("jdk.CPULoad").withPeriod(Duration.ofSeconds(1));
  rs.enable("jdk.JavaMonitorEnter").withThreshold(Duration.ofMillis(10));
  rs.onEvent("jdk.CPULoad", event -> {
    System.out.println(event.getFloat("machineTotal"));
  });
  rs.onEvent("jdk.JavaMonitorEnter", event -> {
    System.out.println(event.getClass("monitorClass"));
  });
  rs.start();
}
```

<!-- /wp:code -->

<!-- wp:paragraph -->

More info can be found in [JEP 349](https://openjdk.java.net/jeps/349).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Language preview features

<!-- /wp:heading -->

<!-- wp:paragraph -->

Java 14 contains several updates to the Java language with are not yet available by default.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

First, [JEP 305](https://openjdk.java.net/jeps/305) extends the `instanceof` operator with a binding variable. Here is an example:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
if (obj instanceof String s) {
    // can use s here
}
```

<!-- /wp:code -->

<!-- wp:paragraph -->

If `obj` is an instance of `String`, then it is cast to `String` and assigned to the binding variable `s`.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Second, [JEP 359](https://openjdk.java.net/jeps/359) introduces records to the Java language. A record has a name and a state description. The state description declares the components of the record. A record may also have a body. Here is a short example:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
record Point(int x, int y) {}
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Third, after collecting feedback for Java 13, [JEP 368](https://openjdk.java.net/jeps/368) adds a couple of new escape sequences for the text blocks which were previously introduced in Java 13 as a language preview feature.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Unfortunately, these three updates are still only available as preview language features that are not enabled by default. To enable the new syntax, you’ll have to run the Java compiler with `--enable-preview --release 14` options, and then launch `java` with `--enable-preview` option:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
$ javac -d classes --enable-preview --release 14 Test.java
$ java -classpath classes --enable-preview Test
```

<!-- /wp:code -->

<!-- wp:heading -->

## The rest

<!-- /wp:heading -->

<!-- wp:paragraph -->

What else has changed in Java 14?

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[JEP 370](https://openjdk.java.net/jeps/370) introduces an API to allow Java applications to safely and efficiently access foreign memory outside of the Java heap. Sounds scary. The new API should become an alternative to the `java.nio.ByteBuffer` and `sun.misc.Unsafe` classes. This feature is provided as an incubating module.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[JEP 353](https://openjdk.java.net/jeps/352) adds new file mapping modes so that the `FileChannel` API can be used to create `MappedByteBuffer` instances that refer to non-volatile memory (NVM).

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The Pack200 tool was deprecated in Java 11. Now [JEP 367](https://openjdk.java.net/jeps/367) removed the tool and its API.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

In case you know about Solaris and SPARC, [JEP 362](https://openjdk.java.net/jeps/362) drops support for the Solaris/SPARC, Solaris/x64, and Linux/SPARC platforms. In the future, the ports on these platforms most likely are going to be removed from OpenJDK.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Conclusion

<!-- /wp:heading -->

<!-- wp:paragraph -->

Comparing to 5 JEPs in Java 13, the new Java 14 delivers much more many major enhancements. The updates touch various areas. Most likely, the most interesting updates for Java developers are going to be the new switch expressions and the enhanced NullPointerExceptions. Don't forget to try out the new language preview features and provide your feedback to the JDK developers. Enjoy the new Java 14!

<!-- /wp:paragraph -->

<!-- wp:heading -->

## References

<!-- /wp:heading -->

<!-- wp:list -->

- [OpenJDK 14 schedule and the list of enhancements](https://openjdk.java.net/projects/jdk/14/)

<!-- /wp:list -->

