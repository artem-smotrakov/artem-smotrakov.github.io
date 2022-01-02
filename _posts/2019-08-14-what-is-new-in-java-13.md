---
layout: post
title: What's new in Java 13
date: 2019-08-14 22:52:10.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tech
tags:
- GC
- Hotspot
- Java
- JEP
- JVM
- Open Source
- OpenJDK
permalink: "/en/tech/what-is-new-in-java-13.html"
---
Java 13 is going to be released on Sep 17th, 2019. Besides ~2300 bug fixes and small enhancements, the new version of Java contains 5 major enhancements which are also called JEPs (Java Enhancement Proposals). Let's take a closer look at these major updates: text blocks, switch expressions, re-implemented the legacy Socket API, updates to ZGC and dynamic CDS archives.

![What is new in Java 13]({{ site.baseurl }}/assets/images/2019/08/duke-small-861x1024.png)

  
  


## JEP 355: Text Blocks (Preview)

Sometimes we need to define a long multi-line string in Java. For example, it can be an HTML page or an SQL query. Here is what it usually looks like:

```java
String html = "<html>\n" +
              " <body>\n" +
              " <p>Hello, world</p>\n" +
              " </body>\n" +
              "</html>\n";
```

You might have noticed a lot of `\n`, `+` and `"` characters which make the code a bit longer and harder to read. Some programming languages such Python allow defining multi-line strings by using triple quotes `"""` to start and end them. Such a language feature may help to make code much nicer. JEP 355 is going to introduce a better way to define multi-line strings in Java. Here is what the JEP says:

> Add text blocks to the Java language. A text block is a multi-line string literal that avoids the need for most escape sequences, automatically formats the string in a predictable way, and gives the developer control over format when desired.

Here is how the example above can be re-written using the text blocks:

```java
String html = """
              <html>
                  <body>
                      <p>Hello, world</p>
                  </body>
              </html>
              """;
```

At runtime, the string above becomes a usual `String` object with the following content. Note that the indentation spaces are removed at runtime:

```
<html>
    <body>
        <p>Hello, world</p>
    </body>
</html>
```

Unfortunately, the new syntax is only a preview language feature that is not available by default. To enable the new syntax, you'll have to run the Java compiler with `--enable-preview --release 13` options, and then launch `java` with `--enable-preview` option. For example:

```
$ javac -d classes --enable-preview --release 13 Test.java
$ java -classpath classes --enable-preview Test
```

Even though the new feature may look simple, JEP 355 discusses quite a lot of important topics around the new text blocks such as line terminators, indentation, incidental white space, escape sequences, concatenation. Furthermore, the JEP adds three new methods: `String::stripIndent()`, `String::translateEscapes()` and `String::formatted(Object… args)`. You're welcome to learn [more about the text blocks in Java](https://openjdk.java.net/jeps/355).

## JEP 354: Switch Expressions (Preview)

This is an update for [JEP 325](https://openjdk.java.net/jeps/325) which introduced new `switch` expressions in JDK 12 as a preview language feature. Making a long story short, JEP 325 added a new simplified form of a `switch` block with `case L ->` labels. [You can find more details in the previous post](/en/tech/what-is-new-in-java-12.html).

After the new syntax was introduced in JDK 12, [a request for feedback was sent to the jdk-dev mailing list](https://mail.openjdk.java.net/pipermail/jdk-dev/2019-April/002770.html). The collected feedback resulted in JEP 354 that makes only one change. In the previous preview version of `switch` expressions it was proposed to add a new form of the `break` statement with a value which would be used to yield a value from a `switch` expression. In the new version of `switch` expressions, this will be replaced with a new `yield` statement. Here is what the JEP says:

> To yield a value from a switch expression, the `break` with value statement is dropped in favor of a `yield` statement.

Here is an example which shows how the new `yield` statement may be used:

```java
int result = switch (s) {
    case "Foo" -> 1;
    case "Bar" -> 2;
    default -> {
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
    }
};
```

Here is another example:

```java
int result = switch (s) {
    case "Foo": 
        yield 1;
    case "Bar":
        yield 2;
    default:
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
};
```

The new `switch` expressions are still available only in preview mode. To enable the new syntax, you'll have to run the Java compiler with `--enable-preview --release 13` options, and then launch `java` with `--enable-preview` option (see the example above).

That's pretty much it. [JEP 354](https://openjdk.java.net/jeps/354) discusses similar topics which were discussed in JEP 325.

## JEP 353: Reimplement the Legacy Socket API

This JEP replaces the underlying implementation of the `java.net.Socket` and `java.net.ServerSocket` APIs. Although someone may not use these APIs directly, this update will likely affect almost everybody who uses JDK because the Java sockets are the main APIs for networking. Here is how the authors of the JEP explain why the update is necessary:

> The java.net.Socket and java.net.ServerSocket APIs, and their underlying implementations, date back to JDK 1.0. The implementation is a mix of legacy Java and C code that is painful to maintain and debug. The implementation uses the thread stack as the I/O buffer, an approach that has required increasing the default thread stack size on several occasions. The implementation uses a native data structure to support asynchronous close, a source of subtle reliability and porting issues over the years. The implementation also has several concurrency issues that require an overhaul to address properly. In the context of a future world of fibers that park instead of blocking threads in native methods, the current implementation is not fit for purpose.

From the description above, it looks like the update should make it easier to develop JDK. Note that the JEP doesn't touch the underlying implementation of `java.net.DatagramSocket`. Most likely JDK users are not going to see a big difference if the update doesn't introduce a regression. Fortunately, the authors know about such a risk, and they won't remove the old implementation. If something goes wrong, the users can switch back to the old implementation by setting `jdk.net.usePlainSocketImpl` system property. However, the old implementation and the system property are going to be removed in one of the next JDK releases. More details can be found on [the official JEP page](https://openjdk.java.net/jeps/353).

## JEP 350: Dynamic CDS Archives

CDS stands for Class-Data Sharing. CDS allows a set of classes to be pre-processed into a shared archive file. Why do we need such an archive? While loading classes, the JVM does quite a lot of things such as parsing the classes, storing them into an internal structure, performing multiple checks on them, resolving links and symbols, etc. Only then, the classes are ready to work. All these steps take some time. Furthermore, a JVM instance usually loads the same set of classes for an application including both application classes and system classes such as `String`, `Integer` and `ArrayList`. All these classes require memory. If we have a shared archive which contains pre-processed classes, it can be memory-mapped at runtime. As a result, it can reduce startup time and memory footprint.

Java versions below 10 allow only system classes to be added to a shared archive. Java 10 extended the CDS feature to allow application classes to be placed in the shared archive as well. [To create such a shared archive with application classes](https://openjdk.java.net/jeps/310), you need to take two steps. First, build a list of classes which are loaded by an application:

```
java -Xshare:off -XX:+UseAppCDS -XX:DumpLoadedClassList=classes.lst \
     -classpath application.jar Application
```

Then, create a shared archive which contains those classes:

```
java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=classes.lst \
     -XX:SharedArchiveFile=application.jsa -classpath application.jar Application
```

JEP 350 extends the application class-data sharing feature to allow the dynamic archiving of classes at the end of Java application execution. It means that the steps above can be combined in a single command:

```
java -XX:ArchiveClassesAtExit=application.jsa \
     -classpath application.jar Application
```

Note that only classes that were loaded during an application’s execution will be added to the archive. In other words, the classes which exist in application.jar but were not loaded during execution will not be included in the archive.

Unfortunately, an archive can't be created automatically when an application runs for the first time. The authors of the JEP explicitly mentioned that:

> A follow-up enhancement to this JEP could perform automatic archive generation during the first run of an application. This would eliminate the explicit archive creation step (step 2 above). The usage of CDS/AppCDS could then be completely transparent and automatic.

In case you're interested, the authors provided quite a lot of [details about JEP 350](https://openjdk.java.net/jeps/350).

## JEP 351: ZGC: Uncommit Unused Memory

[The Z Garbage Collector](https://wiki.openjdk.java.net/display/zgc/Main) (ZGC) is a concurrent garbage collector which tries to reduce garbage collection pauses times. This garbage collector (GC) tries to do all the work while Java threads continue to execute.

When a GC collects garbage, it should normally uncommit and return memory to the operating system. G1 and Shenandoah garbage collectors in Hotspot already do so, but ZGS currently doesn't return memory even if that memory has been unused for a long time. JEP 351 addresses this problem by introducing a new `-XX:ZUncommitDelay` option that tells ZGC when it should return unused memory. [The authors of the JEP explained a bit about how ZGC and the new option works.](https://openjdk.java.net/jeps/351)

## Conclusion

Java 13 doesn't have too many JEPs. I think only text blocks and `switch` expressions are going to be interesting for most of Java users. However, those are only preview language features that are not available by default. I suspect most of the users won't notice the rest of the JEPs integrated to JDK 13. It means that a user should not notice a big difference after migrating to JDK 13. Let's hope that text blocks and `switch` expressions will be finalized and delivered in JDK 14. Meanwhile, you can try those features out and provide your feedback to the JDK developers.

