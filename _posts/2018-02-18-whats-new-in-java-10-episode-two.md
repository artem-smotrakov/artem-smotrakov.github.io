---
layout: post
title: 'What''s new in Java 10: Episode 2'
date: 2018-02-18 21:39:56.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tech
tags:
- Hotspot
- Java
- JEP
- JVM
- Open Source
meta:
  _edit_last: '1'
  _aioseop_opengraph_settings: 'a:15:{s:32:"aioseop_opengraph_settings_title";s:37:"What&#039;s
    new in Java 10: Episode 2";s:31:"aioseop_opengraph_settings_desc";s:194:"Since
    Java 10 is coming, it&#039;s time to have a look at the list of main updates targeted
    to Java 10. Here is a digest of the main features which are planned to be delivered
    in Java 10. Enjoy!";s:32:"aioseop_opengraph_settings_image";s:71:"/wp-content/uploads/2016/08/java_logo.png";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}'
  _wp_old_slug: whats-new-in-java-10-episode-2
  _aioseop_keywords: java,jdk,10,new,features,jep,targeted,enhancement,proposal,jvm,runtime,compilation,compiler,root,certificates,javah
  _aioseop_description: Since Java 10 is coming, it's time to have a look at the list
    of main updates targeted to Java 10. Here is a digest of the main features which
    are planned to be delivered in Java 10. Enjoy!
  _aioseop_title: 'What''s new in Java 10: Episode 2'
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: '258'
  _yoast_wpseo_focuskw_text_input: Java 10
  _yoast_wpseo_focuskw: Java 10
  _yoast_wpseo_linkdex: '77'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_metadesc: Java 10 contains quite a lot of enhancements in the JVM.
    Besides updates to the Java Language and JVM, Java 10 contains another update
    which together with the six-month release model has been bothering the Java community
    for several months. Here is a digest of the rest of the main features in Java
    10.
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618017070'

permalink: "/en/tech/whats-new-in-java-10-episode-two.html"
---
Java 10 is coming in March 2018. This release contains quite a lot of enhancements in the JVM. But it looks like JDK users are mostly interested in one particular update in the Java Language - [type inference to declarations of local variables with initializers](http://openjdk.java.net/jeps/286). Besides updates to the Java Language and JVM, Java 10 contains another update which together with the&nbsp;[six-month release model](https://mreinhold.org/blog/forward-faster#Proposal) has been bothering the Java community for several months.

Here is a digest of the rest of the main features in Java 10 which weren't covered in the [previous post](/en/tech/what-is-new-in-java-10-episode-one.html). Enjoy!

<!--more-->

## Time-Based Release Versioning

Just to make it easy and clear from the very beginning, here is how [JEP 322](http://openjdk.java.net/jeps/322)&nbsp;defines the new format of Java version format:

[1-9][0-9]\*((\.0)\*\.[1-9][0-9]\*)\*

I should probably switch to the next topic because it's hard to add anything else here. But let me try. It allows the version to be of arbitrary length, but the first four elements are assigned specific meanings. Here is what the JEP authors say:

> `$FEATURE.$INTERIM.$UPDATE.$PATCH`
> 
> $FEATURE — The feature-release counter, incremented for every feature release regardless of release content. Features may be added in a feature release; they may also be removed, if advance notice was given at least one feature release ahead of time. Incompatible changes may be made when justified. (Formerly $MAJOR.)
> 
> $INTERIM — The interim-release counter, incremented for non-feature releases that contain compatible bug fixes and enhancements but no incompatible changes, no feature removals, and no changes to standard APIs. (Formerly $MINOR.)
> 
> $UPDATE — The update-release counter, incremented for compatible update releases that fix security issues, regressions, and bugs in newer features. (Formerly $SECURITY, but with a non-trivial incrementation rule.)
> 
> $PATCH — The emergency patch-release counter, incremented only when it's necessary to produce an emergency release to fix a critical issue. (Using an additional element for this purpose minimizes disruption to both developers and users of in-flight update releases.)
> 
> The fifth and later elements of version numbers are reserved for use by downstream consumers of the JDK code base. The fifth element may be used to, e.g., identify implementor-specific patch releases.

This version format is suitable for the [six-month release model](https://mreinhold.org/blog/forward-faster#Proposal)&nbsp;which caused a lot of discussions in the Java community. Here is what the JEP owners say about it:

> Under the six-month release model the elements of version numbers vary as follows:
> 
> $FEATURE is incremented every six months: The March 2018 release is JDK 10, the September 2018 release is JDK 11, and so forth.
> 
> $INTERIM is always zero, since the six-month model does not include interim releases. We reserve it here for flexibility, so that a future revision to the release model could include such releases and say that JDK $N.1 and JDK $N.2 are compatible upgrades of JDK $N. As examples, the JDK 1.4.1 and 1.4.2 releases were, in essence, interim releases, and would have been numbered 4.1 and 4.2 under this scheme.
> 
> $UPDATE is incremented one month after $FEATURE is incremented, and every three months thereafter: The April 2018 release is JDK 10.0.1, the July release is JDK 10.0.2, and so forth.

The new release versioning also caused the following changes:

- new version strings
- added two new system properties `java.version.date` and `java.vendor.version`
- updated&nbsp;Runtime.Version API

The JEP owners put out a lot of information about the new release versioning. You can find it in [JEP 322](http://openjdk.java.net/jeps/322).

## Remove the Native-Header Generation Tool (javah)

How many of us have ever run `ls ${JAVA_HOME}/bin`? You may be surprised by the result. The&nbsp;`bin`&nbsp;directory contains a bunch of tools which come with the JDK. One of them is `javah`&nbsp;which is used to generate&nbsp;JNI header files if you define native methods in your code. How many of us use JNI?

Generating JNI header files has been supported by `javac` since Java 8, so `javah`&nbsp;looks a bit redundant. By the way, there is [Project Panama](http://openjdk.java.net/projects/panama/) which may replace JNI entirely and add more interesting stuff.

The JEP authors don't say much about removing `javah`. There is nothing much to say though. See [JEP 313](http://openjdk.java.net/jeps/313).

## Additional Unicode Language-Tag Extensions

Do you know what a language tag is? Most of us have probably seen it before. For example, `en-GB` and `en-US` are language tags. As you can see, a language tag may contain several elements which are called subtags. Here `en` means English. `GB` and `US` mean a region, so it's Great Britain and United States accordingly. In other words, `en-GB` and `en-US` mean British and American English. Language tags are defined in the BCP 47 documents and several RFCs.

Pretty simple, huh? But as usual it's not enough for humans. A language tag may contain more than two subtags. One possible type of a subtag is called an "extension subtag" which attaches additional information to a language tag. This additional information doesn't have to identify a language itself. For example, it can encode locale information such as calendar and timezone. There are much more many locale attributes which are defined in the Common Locale Data Repository (CLDR). Actually there is a dedicated type of extension subtag for carrying locale data which is called "Extension U". Here are a couple of examples:

- `th-TH-u-ca-buddhist` means&nbsp;Thai (th) in Thailand (TH) with the Buddhist calendar (buddhist)
- `th-TH-u-ca-buddhist-nu-thai` means Thai (th) in Thailand (TH) with the Buddhist calendar (buddhist) and&nbsp; the Thai numbering system (nu-thai)

What about Java? Let's just refer to what the JEP owners say:

> Support for BCP 47 language tags was was initially added in Java SE 7, with support for the Unicode locale extension limited to calendars and numbers. This JEP will implement more of the extensions specified in the latest LDML specification, in the relevant JDK classes.

More precisely, the following extensions are going to be supported by Java 10:

- cu (currency type)
- fw (first day of week)
- rg (region override)
- tz (time zone)

Be aware that the JEP is updating several methods in the&nbsp;`java.text`, `java.time` and `java.util` packages. You can find some details in [JEP 314](http://openjdk.java.net/jeps/314), although not many.

## Heap Allocation on Alternative Memory Devices

It turns out that there are alternative memory devices whose architecture is different from DRAM. For example, there is&nbsp;[NV-DIMM](https://en.wikipedia.org/wiki/NVDIMM)&nbsp;which can even survive from an unexpected power loss, and&nbsp; retain the data even when electrical power is removed. And if such memory exists then someone should allocate&nbsp;the Java object heap on it, right? Starting from Java 10, the Hotspot JVM is going to do that. Here is what the JEP owners say:

> With the availability of cheap NV-DIMM memory, future systems may be equipped with heterogeneous memory architectures. One example of such technology is Intel's 3D XPoint. Such an architecture, in addition to DRAM, will have one or more types of non-DRAM memory with different characteristics.
> 
> This JEP targets alternative memory devices that have the same semantics as DRAM, including the semantics of atomic operations, and can therefore be used instead of DRAM for the object heap without any change to existing application code. All other memory structures such as the code heap, metaspace, thread stacks, etc., will continue to reside in DRAM.

The JEP says some operating systems already expose non-DRAM memory through the file system, for example&nbsp;[NTFS DAX mode](https://channel9.msdn.com/events/build/2016/p470) and [ext4 DAX](https://lwn.net/Articles/618064). The JEP is going to provide a way to allocate the heap in such memory. In other words, it's going to be possible to allocate the heap on a hard drive. It will be probably slower, but on the other hand it provides other features such as heap persistence. The JEP is going to introduce a new option `-XX:AllocateHeapAt=<path>` which&nbsp;takes a path to the file system and uses memory mapping to allocate the object heap.

The JEP owners also say the existing heap related flags such as `-Xmx`, `-Xms`, etc, and garbage-collection related flags would continue to work as before. Phew!

Refer to [JEP 316](http://openjdk.java.net/jeps/316) for details, although unfortunately there isn't much to read.

## Experimental Java-Based JIT Compiler

Are you familiar with runtime compilation? If yes, you should probably skip this paragraph. After you wrote your amazing Java code, you need to compile it to be able to run it. You normally ask `javac`&nbsp;to do that. After that you get a bunch of .class files which contain Java bytecode. Then you usually call `java`&nbsp;to run the bytecode in the JVM. What happens next? The JVM has an interpreter which translates Java bytecode instructions to platform-specific native CPU instructions which finally can be run on a CPU. But the JVM also contains its own compiler which can compile Java bytecode into native code. It's called runtime compilation. Compiled code can then run faster. The JVM can still use the interpreter to run a Java program, but it can also compile some bytecode at runtime. To understand which bytecode should be compiled at runtime, a compiler may gather statistics to understand which Java bytecode is used the most. The most used bytecode then can be compiled to native instructions to make your favorite Java program run faster. A runtime compiler can also try to optimize code during compilation. That also helps to make a Java program run faster. And surprisingly a runtime compiler sometimes may even deoptimize some previously compiled code. There are a couple of strategies for runtime compilers. One of them is called JIT compilation which stands for just-in-time compilation. Basically it means that compilation happens during execution of a program. The bottom line is that a runtime compiler is a pretty important component of a The JVM, and it's usually a pretty complex component.

Currently the HotSpot JVM has two runtime compilers which are called C1 and C2. Making a long story short, C1 is relatively fast and simple compiler which doesn't do much optimization, and is good for applications which require a quick startup. C2 does much more optimization, and it is better for long-running server applications.&nbsp;Both of them are JIT compilers, and both of them are implemented in C/C++/ASM. By the way, C2 even contains its own language for describing optimizations. C2 is a pretty heavy and complex component of the Hotspot JVM.

But there is at least one more JIT compiler for Java. It's called&nbsp;[Graal](https://github.com/oracle/graal). The interesting fact is that Graal is written in Java to compile Java bytecode. The JEP which we're talking about here enables Graal&nbsp;to be used as an experimental JIT compiler on the Linux-x64 platform. Here is what the JEP owners say:

> Enable Graal to be used as an experimental JIT compiler, starting with the Linux/x64 platform. Graal will use the JVM compiler interface (JVMCI) introduced in JDK 9. Graal is already in the JDK, so enabling it as an experimental JIT will primarily be a testing and debugging effort.

In case you want to take part in this experiment, you can run your Java application with the&nbsp;`-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler` options. I think that the Hotspot/Graal teams will probably appreciate your participation if you let them know about your feedback and issues you find.

Refer to [JEP 317](http://openjdk.java.net/jeps/317) for details, although again it doesn't have much detail.

## Root Certificates

Are you familiar with public key certificates and Certificate Authorities? If yes, then the next couple of paragraphs will probably not be interesting for you. In asymmetric cryptography, we have a pair of keys. More precisely,&nbsp; they are the public and private keys. A public key stays public, so everybody has access to it. But a private key must be secret, and only its owner should be able to access it. The keys may be used, for example, for encryption and digital signatures. A public key certificate, or a digital certificate, contains the following:

- a public key
- information about the key
- information about key's owner (the owner of a certificate is also called the subject)
- digital signature of the elements above (it's basically just an array of bytes)

Who signs a certificate? Let's assume that we have a public key `PublicKey1` and a private key `PrivateKey1`. To sign our certificate for `PublicKey1`, we need another pair of private and public keys. Let's call them `PublicKey2` and `PrivateKey2`. More precisely, `PrivateKey2` is used for creating a signature, and `PublicKey2` is used for verifying a signature. `PublicKey1` together with info about it and its owner are signed with `PrivateKey2`. The signature has to be included in the certificate. Now we have a signed certificate `C1` for `PublicKey1`. The signature can be verified by `PublicKey2`. As you can imagine, there should be a certificate which contains&nbsp;`PublicKey2`. But who signs this certificate for `PublicKey2`? We need a new pair of public and private keys. Here we come to a chain of certificates.

The purpose of signing a certificate is to provide a way for a user to make sure that a public key from a certificate `C1` really belongs to the subject. A user can take a public key from the next certificate `C2` in a chain, and use it to validate the signature. But then, we need to validate the signature of certificate `C2`. So then we use certificate `C3` in the chain ... But it can't continue infinitely. When do we stop? Here we come to trusted certificates. We can have a set of certificates which we trust. Such certificates are also called root certificates. We stop validating a certificate chain which we reach one of trusted certificates.

Normally a certificate for a public key is issued by an organization which is called Certificate Authority (CA). Digicert, Comodo, VeriSign are examples of well-known Certificate Authorities. CAs have their own certificates which they use for signing other certificates.

Why do we care about certificate chains? For example, they are widely used in TLS communications. When you see a tiny green lock in your favorite web browser on the left of the URL, that means your browser successfully validated the certificate chain of the web site you are visiting. Web browsers usually contain a set of trusted root certificates.

The good news is that Java also contains storage for root certificates. It is the&nbsp;`cacerts` file which can be found in `${JAVA_HOME}/lib/security`. It's just a Java keystore. The bad news is that `cacerts` in OpenJDK builds is empty by default. But it's finally going to be fixed in Java 10. Basically, the `cacerts` keystore in OpenJDK builds is going to contain certificates from the Oracle JDK. Here is what the JEP owners say:

> The cacerts keystore will be populated with a set of root certificates issued by the CAs of Oracle's Java SE Root CA Program. As a prerequisite, each CA must sign the Oracle Contributor Agreement (OCA), or an equivalent agreement, to grant Oracle the right to open-source their certificates. Below are the CAs that have signed the required agreement and, for each, a list of the root certificates (identified by the Distinguished Name) that will be included. This list includes a majority of the CAs that are currently members of Oracle's Java SE Root CA Program. Those that do not sign an agreement will not be included at this time. Those that take longer to process will be included in the next release.

And then the JEP provides a long list of certificate subjects which are going to be available in OpenJDK builds by default. It sounds pretty easy just to copy the `cacerts` keystore from the Oracle JDK build to the OpenJDK build. But actually there is a lot of work behind this simple update because it needs a confirmation from each and every CA that they don’t mind open-sourcing those certificates in the OpenJDK repositories. Even if those certificates are already public.

You can find the details in [JEP 319](http://openjdk.java.net/jeps/319).

## Conclusion

Phew! We're done with all main updates in Java 10! If you are interested, I talked a bit about the first half of them in my [previous post](/en/tech/what-is-new-in-java-10-episode-one.html). In total, [Java 10 contains 12 pretty big updates](http://openjdk.java.net/projects/jdk/10/). It's going to be the next short-term Java release which should go live in Mar 2018. Let's hope it will be in time, and won't have many regressions.

P.S. I would appreciate a lot if you let me know in case I wrote anything dumb and stupid :)

