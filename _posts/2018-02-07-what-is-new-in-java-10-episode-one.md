---
layout: post
title: 'What''s new in Java 10: Episode 1'
date: 2018-02-07 21:42:04.000000000 +00:00
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
- JVM
- Open Source
meta:
  _edit_last: '1'
  _aioseop_opengraph_settings: 'a:15:{s:32:"aioseop_opengraph_settings_title";s:37:"What&#039;s
    new in Java 10: Episode 1";s:31:"aioseop_opengraph_settings_desc";s:206:"Since
    Java 10 is coming, it&#039;s time to have a look at the JEPs (Java Enhancement
    Proposal) targeted to Java 10. Here is a digest of the main features which are
    planned to be delivered in Java 10. Enjoy!";s:32:"aioseop_opengraph_settings_image";s:71:"/wp-content/uploads/2016/08/java_logo.png";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}'
  _aioseop_keywords: java,jdk,10,new,features,jep,targeted,enhancement,proposal,garbage,collector,g1,jvm,cds,appcds,local,variable
  _aioseop_description: Since Java 10 is coming, it's time to have a look at the JEPs
    (Java Enhancement Proposal) targeted to Java 10. Here is a digest of the main
    features which are planned to be delivered in Java 10. Enjoy!
  _aioseop_title: 'What''s new in Java 10: Episode 1'
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: '258'
  _yoast_wpseo_focuskw_text_input: Java 10
  _yoast_wpseo_focuskw: Java 10
  _yoast_wpseo_metadesc: Since Java 10 is coming, it's time to have a look at the
    JEPs (Java Enhancement Proposal) targeted to Java 10. Here is a digest of the
    main features which are planned to be delivered in Java 10. Enjoy!
  _yoast_wpseo_linkdex: '71'
  _yoast_wpseo_content_score: '30'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:10;s:7:"replies";i:4;s:7:"authors";i:8;s:14:"recent_authors";a:5:{i:0;O:8:"stdClass":3:{s:20:"comment_author_email";s:27:"preetiagarwal1634@gmail.com";s:14:"comment_author";s:6:"preeti";s:7:"user_id";s:1:"0";}i:1;O:8:"stdClass":3:{s:20:"comment_author_email";s:27:"preetiagarwal1634@gmail.com";s:14:"comment_author";s:5:"preet";s:7:"user_id";s:1:"0";}i:2;O:8:"stdClass":3:{s:20:"comment_author_email";s:19:"isharoyhr@gmail.com";s:14:"comment_author";s:4:"isha";s:7:"user_id";s:1:"0";}i:3;O:8:"stdClass":3:{s:20:"comment_author_email";s:23:"mridulatw0310@gmail.com";s:14:"comment_author";s:7:"Mridula";s:7:"user_id";s:1:"0";}i:4;O:8:"stdClass":3:{s:20:"comment_author_email";s:25:"artem.smotrakov@gmail.com";s:14:"comment_author";s:5:"Artem";s:7:"user_id";s:1:"1";}}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618072037'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/tech/what-is-new-in-java-10-episode-one.html"
---
Java 10 should be released in Mar 2018. It's going to be the next short-term release after Java 9 which was released in Sep 2017.

After I had left the Java Team at Oracle in the end of 2017, and moved to another side of Atlantic Ocean, I made a New Year's promise that I'll keep myself updated about changes in Java at least for a year.

Since Java 10 is coming, it's time to have a look at the JEPs (Java Enhancement Proposal) [targeted to Java 10](http://openjdk.java.net/projects/jdk/10). Here is a digest of the main features which are planned to be delivered in Java 10. Enjoy!

&nbsp;

<!--more-->

## Local-Variable Type Inference

This is a new language feature which is supposed to enhance the Java language to extend type inference to declarations of local variables with initializers. Here is what the JEP says:

> We seek to improve the developer experience by reducing the ceremony associated with writing Java code, while maintaining Java's commitment to static type safety, by allowing developers to elide the often-unnecessary manifest declaration of local variable types. This feature would allow, for example, declarations such as:
> 
> ```
> var list = new ArrayList<String>(); // infers ArrayList<String>
> var stream = list.stream(); // infers Stream<String>
> ```
> 
> This treatment would be restricted to local variables with initializers, indexes in the enhanced&nbsp;for-loop, and locals declared in a traditional&nbsp;for-loop; it would not be available for method formals, constructor formals, method return types, fields, catch formals, or any other kind of variable declaration.

It basically is going to introduce in Java language, well, not really a keyword or a reserved word, but something which is called "var". Here is what the JEP owners say about it:

> The identifier&nbsp;`var`&nbsp;is not a keyword; instead it is a&nbsp;_reserved type name_&nbsp;or a&nbsp;_context-sensitive keyword_. This means that code that uses&nbsp;`var`&nbsp;as a variable, method, or package name will not be affected; code that uses&nbsp;`var`&nbsp;as a class or interface name will be affected (but these names are rare in practice, since they violate usual naming conventions).

This update to the Java language looks much smaller than for example lambdas, but still someone might have been waiting for such language sugar for long time. Some of you may still live in hope of deliverance of `public static void main(String[] args)` ... Should I file a JEP for it?

The JEP owners kindly remind us to use our brain while writing code:

> Like any other language feature, local variable type inference can be used to write both clear and unclear code; ultimately the responsibility for writing clear code lies with the user.

Please refer to [JEP 286](http://openjdk.java.net/jeps/286) for details.

## Consolidate the JDK Forest into a Single Repository

It's hard to say this is really a new feature in Java, but it's still tracked by a JEP, so it's considered as a "feature". This JEP re-organizes the OpenJDK sources which probably is going to be interesting only for those who take part in JDK development. In this JEP, they basically move stuff around, consolidate some Mercurial repositories, do some renaming, etc. So, it's pretty much unlikely that it's going to affect JDK users. Well, let's at least hope so.

Here is what the JEP owners say about this project:

> Combine the numerous repositories of the JDK forest into a single repository in order to simplify and streamline development.

Please refer to [JEP 296](http://openjdk.java.net/jeps/296) in case you occasionally found yourself in&nbsp;[this list](http://openjdk.java.net/census).

## Garbage-Collector Interface

Again this one doesn't really look like a feature for Java and JDK users. The main purpose of this JEP is to make it easier to implement and plug-in a new garbage collector in the JVM (when I was writing this, I missed the word 'collector' by accident ... now I have been thinking for a couple of day what if it was a sign?..)

Let's see what the JEP owners say about it:

> Improve the source code isolation of different garbage collectors by introducing a clean garbage collector (GC) interface.

Seems like the current garbage collector interface looks pretty dirty. And here is the goals the JEP owner would like to achieve:

> - Better modularity for HotSpot internal GC code
> - Make it simpler to add a new GC to HotSpot without perturbing the current code base
> - Make it easier to exclude a GC from a JDK build

This update in Java may be useful for ones who contribute to the JVM. Please refer to [JEP 304](http://openjdk.java.net/jeps/304) for details.

## Parallel Full GC for G1

Not sure if many of us love cleaning up our homes. At least I feel like most of people don't like to do that too often. Imagine if we had to do a full cleanup every weekend. It may take several hours of your well-deserved weekend. Sounds awful to me. I personally try to avoid this as often as I can, and I feel like most of people do the same. But unfortunately our places finally become dirty, and a good cleanup is unavoidable. But we're lazy, so we look for help from our wifes, girlfriends, boyfriends and sometimes even pets.

The Hotspot JVM behaves the same. It has the G1 garbage collector. This garbage collector tries to avoid expensive full collections, but sometimes it's unavoidable, and the garbage collector starts a full collection. The problem is that the current G1 runs a full collection in a single thread. This JEP updates G1 to use multiple threads for a full collection, and allows us to play with it by specifying the&nbsp;`-XX:ParallelGCThreads` option.

By the way, G1 is the default garbage collector in the JDK 9, so this JEP is going to affect everybody who has switched to 9 and uses the default Hotspot configuration. By the way, how many of you care about it?

Unfortunately&nbsp;[JEP 307](http://openjdk.java.net/jeps/307) doesn't have much details.

Let's hope that garbage is going to be collected in time.

## Application Class-Data Sharing

Do you know what CDS is? If so, you can skip this paragraph. CDS stands for Class-Data Sharing. Yes, it's basically about Java class files. CDS allows a set of classes to be pre-processed into a shared archive file. But what does "pre-processed" mean? Making a long story short, the JVM does a lot of magic when it's loading a class. The JVM parses the class, stores it into an internal structure, performs some checks on it, resolve and link the symbols, etc. Only after that, the class is ready to work. As you can imagine, all these steps take time. Furthermore, each JVM instance usually loads the same system classes like String, Integer, URLConnection, etc which are included to Java by default. As you can imagine, all these classes require memory. So, when we have a shared archive which contains pre-processed classes, it can then be memory-mapped at runtime. As a result, it can reduce startup time, and memory footprint if multiple JVMs share the same archive file.

Okay, that's cool, but CDS has been actually there since Java 5. Now let's take a closer look at the JEP name - "Application Class-Data Sharing". Besides "Class-Data Sharing" it also contains the word "Application". Currently CDS only allows the bootstrap class loader to load archived classes. Application CDS (aka AppCDS) extends CDS, and allows other class loaders to load archived classes. What other class loaders do we have? Here it is:

- the built-in system class loader (aka the application class loader, the word "system" has been always confusing me!)
- the built-in platform class loader
- custom class loaders

The JEP owners mention analysis of large-scale enterprise applications and even serverless cloud services which shows that AppCDS is going to help a lot. Here is what they say:

> Analysis of the memory usage of large-scale enterprise applications shows that such applications often load tens of thousands of classes into the application class loader. Applying AppCDS to these applications will result in memory savings of tens to hundreds of megabytes per JVM process.
> 
> Analysis of serverless cloud services shows that many of them load several thousand application classes at start-up. AppCDS can allow these services to start up quickly and improve the overall system response time.

Sounds promising. Let's see how it works.

The JEP contains a lot of good information about the project. The JEP owners definitely put a lot of work on it. They also kindly provide examples how AppCDS can be used. Please refer to [JEP 310](http://openjdk.java.net/jeps/310)

## Thread-Local Handshakes

Do you know what a JVM safepoint is? Maybe it's better not to know at all ... Anyway, a safepoint in the JVM means that "we stop the world". All Java threads are suspended during a safepoint.&nbsp;Suspending all Java threads are necessary to make sure that only we have access to data structures in the JVM to safely work with them. Safepoints may be used due to several reasons, for example garbage collection, code deoptimization, biased lock revocation, etc.

Making a long story short, this JEP is trying to&nbsp;reduce the number of global safepoints. Here is what the JEP owners say:

> Introduce a way to execute a callback on threads without performing a global VM safepoint. Make it both possible and cheap to stop individual threads and not just all threads or none.

It's relatively easy, isn't it? But the following should blow up your head unless you have been working on virtual machines for last 30-40 years (please do not let kids and impressionable pets read it because it can easily affect their mental health):

> A handshake operation is a callback that is executed for each JavaThread while that thread is in a safepoint safe state. The callback is executed either by the thread itself or by the VM thread while keeping the thread in a blocked state. The big difference between safepointing and handshaking is that the per thread operation will be performed on all threads as soon as possible and they will continue to execute as soon as it’s own operation is completed. If a JavaThread is known to be running, then a handshake can be performed with that single JavaThread as well.
> 
> In the initial implementation there will be a limitation of at most one handshake operation in flight at a given time. The operation can, however, involve any subset of all JavaThreads. The VM thread will coordinate the handshake operation through a VM operation which will in effect prevent global safepoints from occurring during the handshake operation.
> 
> The current safepointing scheme is modified to perform an indirection through a per-thread pointer which will allow a single thread's execution to be forced to trap on the guard page. Essentially, at all times there will be two polling pages: One which is always guarded, and one which is always unguarded. In order to force a thread to yield, the VM updates the per-thread pointer for the corresponding thread to point to the guarded page.

It's crazy ...&nbsp; but if it's not enough for you, please refer to&nbsp;[JEP 312](http://openjdk.java.net/jeps/312)

Here is a follow-up post which covers [the rest of the main updates in Java 10](/en/tech/whats-new-in-java-10-episode-two.html).

P.S. I would appreciate a lot if you let me know in case I wrote anything dumb and stupid :)

