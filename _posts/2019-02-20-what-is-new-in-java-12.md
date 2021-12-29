---
layout: post
title: What's new in Java 12
date: 2019-02-20 08:13:58.000000000 +00:00
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
meta:
  _edit_last: '1'
  _yoast_wpseo_primary_category: '258'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_focuskw: Java 12
  _yoast_wpseo_metadesc: 'Let''s take a look what is inside Java 12: enhancements
    in garbage collection, JVM Constants API, default CDS archives and new switch
    statements.'
  _yoast_wpseo_linkdex: '75'
  rp4wp_auto_linked: '1'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:1;s:7:"replies";i:0;s:7:"authors";i:1;s:14:"recent_authors";a:1:{i:0;O:8:"stdClass":3:{s:20:"comment_author_email";s:27:"preetiagarwal1634@gmail.com";s:14:"comment_author";s:6:"preeti";s:7:"user_id";s:1:"0";}}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618017066'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/tech/what-is-new-in-java-12.html"
---
<!-- wp:paragraph -->

Let's take a look what is inside Java 12. The new Java release contains less major enhancements than the previous version: [8](https://openjdk.java.net/projects/jdk/12/) JEPs in Java 12 vs [17](https://openjdk.java.net/projects/jdk/11/) JEPs in Java 11. As you of course remember, JEP stands for JDK Enhancement Proposal. Java 11 also had more closed entries in Jira: ~[2700](https://bugs.openjdk.java.net/browse/JDK-8218034?jql=project%20%3D%20JDK%20AND%20resolution%20in%20(Fixed%2C%20Delivered)%20AND%20fixVersion%20%3D%20%2211%22) in Java 11 vs ~[2400](https://bugs.openjdk.java.net/browse/JDK-8219012?jql=project%20%3D%20JDK%20AND%20resolution%20in%20(Fixed%2C%20Delivered)%20AND%20fixVersion%20%3D%20%2212%22) in Java 12. But it's only mid of Feb 2019, maybe they can deliver 300 Jira entries by Mar 19th 2019 when Java 12 is planned to be released. Now let's take a closed look what is in Java 12.

<!-- /wp:paragraph -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## JEP 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)

<!-- /wp:heading -->

<!-- wp:paragraph -->

I noticed that recent Java releases contain many updates in JVM especially in garbage collection (GC). JEP 189 is another update in this area. The JEP adds a new garbage collection algorithm which is named Shenandoah. As you can see in the JEP title, the main feature of the new garbage collection algorithm is reducing garbage collection pause times. Here is what the JEP authors say:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> ... reduces GC pause times by doing evacuation work concurrently with the running Java threads. Pause times with Shenandoah are independent of heap size, meaning you will have the same consistent pause times whether your heap is 200 MB or 200 GB.

<!-- /wp:quote -->

<!-- wp:paragraph -->

And here is another interesting quote from the JEP about its goals:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> Shenandoah is an appropriate algorithm for applications which value responsiveness and predictable short pauses. The goal is not to fix all JVM pause issues. Pause times due to reasons other than GC like Time To Safe Point (TTSP) issues or monitor inflation are outside the scope of this JEP.

<!-- /wp:quote -->

<!-- wp:paragraph -->

If you want to learn more about the new GC, go ahead and read [JEP 189](https://openjdk.java.net/jeps/189). You can also try out the new garbage collection algorithm - just use the following JVM options:

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

`-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC`

<!-- /wp:paragraph -->

<!-- wp:heading -->

## **JEP 230: Microbenchmark Suite**  

<!-- /wp:heading -->

<!-- wp:paragraph -->

This JEP is for those who take part in JDK development. JEP 230 adds a suite of microbenchmarks to the JDK source code. So, if you are planning just to download Java 12 and run your applications on it, then most probably this JEP is not going to be interesting for you. But in case you want to learn more about performance testing in Java and Java Microbenchmark Harness (JMH), then you are more than welcome to read [JEP 230](https://openjdk.java.net/jeps/230).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 325: Switch Expressions (Preview)

<!-- /wp:heading -->

<!-- wp:paragraph -->

I believe most of developers will find this JEP more interesting than the previous one. JEP 325 introduces "simplified" switch statements. To make a long story short, let me just give you a couple of examples.

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
static void howMany(int k) {
    switch (k) {
        case 1 -> System.out.println("one");
        case 2 -> System.out.println("two");
        case 3 -> System.out.println("many");
    }
}
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

In other words, if a label is matched, then only the expression to the right of an arrow label is executed. No fall through, so you don't need break statements. Here is another example:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
static boolean isWeekend(String day) {
    return switch (day) {
        case "mon", "tue", "wed", "thu", "fri" -> false;
        case "sat", "sun" -> true;
        default -> throw new IllegalArgumentException("oops!");
    };
}
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

The example above uses multiple case labels and shows that the expression to the right of an arrow may be a return value. But you can also have a full block if you need to put more logic inside. Furthermore, the break statement now can take an argument which becomes the value of the enclosing switch expression. Here is an example:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
static boolean isWeekend(String day) {
    return switch (day) {
        case "mon", "tue", "wed", "thu", "fri" -> false;
        case "sat", "sun" -> true;
        default -> {
            if (day.startsWith("s")) {
                System.out.println("looks like weekend");
                break true;
            }

            System.out.printf("unknown day: %s%n", day);
            break false;
        }
    };
}
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Note that a switch expression must complete with a value, or throw an exception.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

You might have noticed the "preview" word in the JEP title. It means this is a preview language feature which is not enabled by default. If you want to play with the new syntax, you have to give `--enable-preview --release 12` options to the Java compiler and `--enable-preview` option to the `java` command:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
$ javac -d classes --enable-preview --release 12 Test.java
$ java -classpath classes --enable-preview Test
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

The authors of this JEP did a good job and described the new feature really well. Here is what they said:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> ... These changes will simplify everyday coding ...

<!-- /wp:quote -->

<!-- wp:paragraph -->

That's nice. You can find all the details in [JEP 325](https://openjdk.java.net/jeps/325).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 334: JVM Constants API

<!-- /wp:heading -->

<!-- wp:paragraph -->

A Java class file has a constant pool which stores the operands for bytecode instructions in the class. The content of the constant pool describes either runtime artifacts such as classes and methods, or simple values such as strings and integers. All these entries may serve as operands for the `ldc` bytecode instruction which loads constants to be used by other instructions, or these entries may also be used in the static argument list of a bootstrap method for the `invokedynamic` bytecode instruction. At runtime, executing an `ldc` or `invokedynamic` instruction causes resolving the constant into a value of one of the standard Java types such as Class, String, or int.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

JEP 334 introduces an API to model constants in the constant pool. This API may be useful for programs and libraries which deal with bytecode instructions and manipulate class files. So unlike the previous JEP, this update won't probably simplify everyday coding for most Java developers.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The new API can be found in [java.lang.constant](http://cr.openjdk.java.net/~iris/se/12/build/latest/api/java.base/java/lang/constant/package-summary.html) package. And you can always find more details in [JEP 334](https://openjdk.java.net/jeps/334).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 340: One AArch64 Port, Not Two

<!-- /wp:heading -->

<!-- wp:paragraph -->

Java can run on ARM. Furthermore, the JDK currently contains even two 64-bit ARM ports. More precisely, and if you are familiar with the structure of the JDK code repositories, the main sources for these ports are in the src/hotspot/cpu/arm and open/src/hotspot/cpu/aarch64 directories. It may look a bit redundant. JEP 340 removes all of the sources related to the arm64 port, and keeps only the 32-bit ARM port and the 64-bit aarch64 port. Here is what the JEP authors say:

<!-- /wp:paragraph -->

<!-- wp:quote {"className":"is-style-default"} -->

> Removing this port will allow all contributors to focus their efforts on a single 64-bit ARM implementation, and eliminate the duplicate work required to maintain two ports

<!-- /wp:quote -->

<!-- wp:paragraph -->

Most likely this update won't be interesting for Java users. The JEP authors probably knew that and didn't put much details about [JEP 340](https://openjdk.java.net/jeps/340).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 341: Default CDS Archives

<!-- /wp:heading -->

<!-- wp:paragraph -->

CDS stands for Class-Data Sharing. CDS allows a set of classes to be pre-processed into a shared archive file. Why do we need it? The JVM does a lot of magic while it’s loading classes. The JVM parses the class, stores it into an internal structure, performs some checks on it, resolves links and symbols, etc. Then, the class is ready to work. All these steps take some time. Furthermore, each JVM instance usually loads the same system classes like String, Integer, URLConnection, etc which are included to Java by default. All these classes require memory. When we have a shared archive which contains pre-processed classes, it can be memory-mapped at runtime. As a result, it can reduce startup time and memory footprint.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Currently, a JDK image includes a default class list in the lib directory. This default class list is generated at build time. If we want to take advantage of CDS with the default class list provided in the JDK, we have to run `java -Xshare:dump` as an extra step.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

All the above was just an introduction. Now, let's see what JEP 341 does. The JEP modifies the JDK build to run `java -Xshare:dump` after linking the image. The resulting CDS archive will be created in the `lib/server` directory which means it is part of the JDK. As a result, the CDS feature will be enabled automatically when an application starts (remember that `-Xshare:auto` was enabled by default for the server VM in JDK 11). In case you would like to disable CDS, you need to use `-Xshare:off` option.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

More details in [JEP 341](https://openjdk.java.net/jeps/341).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## **JEP 344: Abortable Mixed Collections for G1**  

<!-- /wp:heading -->

<!-- wp:paragraph -->

The next major enhancement is about garbage collection. You might know about G1 garbage collector in the Hotspot JVM. You might also know that G1 pauses application threads (stop-the-world) when it starts collecting garbage. G1 can be configured with a max pause time goal, for example, by setting `-XX:MaxGCPauseTimeMillis` option. Then, G1 tries to meet garbage collection pause time goals using several techniques.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Unfortunately sometimes the G1 collector doesn't meet the pause time goal. In particular, it can often happen during mixed collections when G1 tries to collect both young and old heap regions.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

JEP 344 tries to make the G1 collector smarter, and updates it to be able to abort mixed collections if they exceed the pause goal. Here is what the JEP authors say:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> If G1 discovers that the collection set selection heuristics repeatedly select the wrong number of regions, switch to a more incremental way of doing mixed collections: split the collection set into two parts, a mandatory and an optional part. The mandatory part comprises parts of the collection set that G1 cannot process incrementally (e.g., the young regions) but can also contain old regions for improved efficiency. [...]  
> After G1 finishes collecting the mandatory part, G1 starts collecting the optional part at a much more granular level, if there is time left. The granularity of collection of this optional part depends on the amount of time left, at most down to one region at a time. After completing collection of any part of the optional collection set, G1 can decide to stop the collection depending on the remaining time.  
> As the predictions get more accurate again, the optional part of a collection are made smaller and smaller, until the mandatory part once again comprises all of the collection set. [...] If the predictions becomes inaccurate again, then the next collections will consist of both a mandatory and optional part again.

<!-- /wp:quote -->

<!-- wp:paragraph -->

You can find more details in [JEP 344](https://openjdk.java.net/jeps/344) and [G1 documentation](https://docs.oracle.com/en/java/javase/11/gctuning/garbage-first-garbage-collector.html#GUID-ED3AB6D3-FD9B-4447-9EDF-983ED2F7A573).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 346: Promptly Return Unused Committed Memory from G1

<!-- /wp:heading -->

<!-- wp:paragraph -->

And one more enhancement for the G1 collector. Currently G1 only returns memory from the Java heap when a full GC happens or during a concurrent cycle. But both event may happen not very often in G1. As a result, the memory may not be returned to the operating system for long time.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

JEP 346 updates the G1 garbage collector to detect inactivity of the application (idle) and return Java heap memory to the operating system when idle. Here is what the JEP authors say:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> To accomplish the goal of returning a maximum amount of memory to the operating system, G1 will, during inactivity of the application, periodically try to continue or trigger a concurrent cycle to determine overall Java heap usage. This will cause it to automatically return unused portions of the Java heap back to the operating system.

<!-- /wp:quote -->

<!-- wp:paragraph -->

The authors found a very good motivation for this kind of update. Here is what they say about the fact that the G1 collector doesn't return memory often enough:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> This behavior is particularly disadvantageous in container environments where resources are paid by use. Even during phases where the VM only uses a fraction of its assigned memory resources due to inactivity, G1 will retain all of the Java heap. This results in customers paying for all resources all the time, and cloud providers&nbsp;not being able to fully utilize their hardware.

<!-- /wp:quote -->

<!-- wp:paragraph -->

I am wondering if G1 could have an option like `-XX:PriceOfMegabyte`, so that it could also report how much money it saved because of [JEP 346](https://openjdk.java.net/jeps/346).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Conclusion

<!-- /wp:heading -->

<!-- wp:paragraph -->

Most of major enhancements in Java 12 are more or less in JVM area. Plus, Java 12 has one interesting update in the Java language but unfortunately it's not enabled by default. Hope they can finalize the improvements for the switch statement and release them in Java 13.

<!-- /wp:paragraph -->

<!-- wp:image {"id":2956,"align":"center","width":334,"height":396} -->

![Java]({{ site.baseurl }}/assets/images/2019/02/duke-small-1-861x1024.png)

<!-- /wp:image -->

