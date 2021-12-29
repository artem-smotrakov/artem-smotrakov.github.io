---
layout: post
title: What's new security features in Java 11?
date: 2018-09-26 10:34:02.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- JEP
- JSSE
- Security
- TLS 1.3
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_primary_category: '157'
  _yoast_wpseo_focuskw: Java 11
  _yoast_wpseo_metadesc: Besides a huge number of small improvements and bug fixes,
    Java 11 contains 17 major enhancements. But let's focus on security related features
    in this post.
  _yoast_wpseo_linkdex: '88'
  rp4wp_auto_linked: '1'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:0;s:7:"replies";i:0;s:7:"authors";i:0;s:14:"recent_authors";a:0:{}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618017049'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/security/whats-new-security-features-in-java-11.html"
---
Java 11 was released on Sep 25th, 2018. This is the first long-term support release produced under the six-month cadence release model. Besides a huge number of small improvements and bug fixes, the new release contains [17 major enhancements](http://openjdk.java.net/projects/jdk/11/)&nbsp;including:

- several updates in the Hotspot and garbage collectors
- new HTTP client
- Unicode 10
- deprecating&nbsp;Nashorn JavaScript Engine and Pack200 tool
- removing&nbsp;the Java EE and CORBA Modules
- local-variable syntax for Lambda parameters
- launch single-file source-code programs
- and finally several security features

Although all these features are pretty cool, let's focus on security in this post.

<!--more-->

## Transport Layer Security (TLS) 1.3

TLS 1.3 specification was published in August 2018, and Java 11 already supports a minimal implementation of the new protocol. Here is what the JEP authors says:

> The primary goal of this JEP is a minimal interoperable and compatible TLS 1.3 implementation. A minimal implementation should support:
> 
> - Protocol version negotiation
> - TLS 1.3 full handshake
> - TLS 1.3 session resumption
> - TLS 1.3 key and iv update
> - TLS 1.3 updated OCSP stapling
> - TLS 1.3 backward compatibility mode
> - TLS 1.3 required extensions and algorithms
> - RSASSA-PSS signature algorithms (8146293)

There are no new APIs for TLS 1.3, the JEP only introduces several new constants:

> - TLS protocol version name: TLSv1.3
> - javax.net.ssl.SSLContext algorithm name: TLSv1.3
> - TLS cipher suite names for TLS 1.3: TLS\_AES\_128\_GCM\_SHA256, TLS\_AES\_256\_GCM\_SHA384.

[Java Secure Sockets Extension (JSSE)](https://bugs.openjdk.java.net/browse/JDK-8185576)&nbsp;has been significantly re-worked in Java 11. I used to work on security-libs in Java, and I can tell that was not an easy project. But nevertheless, Java security-libs team delivered TLS 1.3 implementation in Java 11. Great job!

Although there seem to be several bugs in the TLS 1.3 implementation which may affect interoperability with other protocol implementations, for example:

- [NPE in SupportedGroupsExtension](https://bugs.openjdk.java.net/browse/JDK-8209916)
- [TLS 1.3 server fails if ClientHello doesn't have pre\_shared\_key and psk\_key\_exchange\_modes](https://bugs.openjdk.java.net/browse/JDK-8210334)
- [TLSv.1.3 interop problems with OpenSSL 1.1.1 when used on the client side with mutual auth](https://bugs.openjdk.java.net/browse/JDK-8210846)

Most of the issues look to be already fixed in the development repository. Hope the fixes will be delivered in the next update release for Java 11.

If you'd like to try TLS 1.3 out with Java 11, here you can find&nbsp;[an example of TLS 1.3 client and server on Java](/en/security/an-example-of-tls-13-client-and-server-on-java.html)

## ChaCha20 and Poly1305 Cryptographic Algorithms

There are several new features around TLS 1.3 implementation in JSSE. This is one of them. ChaCha20 is a stream cipher which is supposed to replace insecure RC4 cipher (disabled in Java), and Poly1305 is a message authentication code (MAC). ChaCha20 may be used in AEAD mode with the Poly1305 authenticator. This AEAD algorithm is usually called&nbsp;ChaCha20-Poly1305 and it's also available in Java 11. Here is what the JEP author says:

> ## Summary
> 
> Implement the ChaCha20 and ChaCha20-Poly1305 ciphers as specified in&nbsp;[RFC 7539](https://tools.ietf.org/html/rfc7539). ChaCha20 is a relatively new stream cipher that can replace the older, insecure RC4 stream cipher.
> 
> ## Goals
> 
> - Provide ChaCha20 and ChaCha20-Poly1305&nbsp;`Cipher`&nbsp;implementations. These algorithms will be implemented in the SunJCE provider.
> - Provide a&nbsp;`KeyGenerator`&nbsp;implementation that creates keys suitable for ChaCha20 and ChaCha20-Poly1305 algorithms.
> - Provide an&nbsp;`AlgorithmParameters`&nbsp;implementation for use with the ChaCha20-Poly1305 algorithm.

TLS 1.3. specification defines a couple of new cipher suites which use ChaCha20-Poly1305 algorithm but unfortunately supporting those new cipher suites was not a goal of this JEP. Hope we'll get them in next Java releases.

You can find more details in [JEP 329](http://openjdk.java.net/jeps/329).

## Key Agreement with Curve25519 and Curve448

This is another JEP related to TLS 1.3 implementation in Java. It adds a couple of new key exchange algorithms based on Curve25519 and Curve448 elliptic curves. Let's take a look what the JEP author says:

> ## Summary
> 
> Implement key agreement using Curve25519 and Curve448 as described in&nbsp;[RFC 7748](https://tools.ietf.org/html/rfc7748).
> 
> ## Goals
> 
> RFC 7748 defines a key agreement scheme that is more efficient and secure than the existing elliptic curve Diffie-Hellman (ECDH) scheme. The primary goal of this JEP is an API and an implementation for this standard. Additional implementation goals are:
> 
> 1. Develop a platform-independent, all-Java implementation with better performance than the existing ECC (native C) code at the same security strength.
> 2. Ensure that the timing is independent of secrets, assuming the platform performs 64-bit integer addition/multiplication in constant time. In addition, the implementation will not branch on secrets. These properties are valuable for preventing side-channel attacks.

It's great to see that the authors explicitly mentioned testing against side-channel attacks. These new algorithms are enabled in TLS 1.3 specification and even earlier protocol versions. More details in [JEP 324](http://openjdk.java.net/jeps/324).

## Nest-Based Access Control

Java allows defining&nbsp;multiple classes in a single source file and even nested classes. But the Java compiler then produces a separate class file for each class. On the language layer, the classes should be able to access each others members (even private ones), but at runtime it's prohibited by the JVM. To overcome this, the java compiler generates synthetic methods (they also are usually called bridge methods) which provide read-write access to the private fields. Such synthetic methods usually have package-level access.

Here is an example. Let's consider the following Java code:  
<script src="https://gist.github.com/artem-smotrakov/b520bea1b807529edf1d6c4948025e85.js"></script>  
If we compile it with JDK 10 and run `javap`, then we'll get something like the following:

```
$ javac -d classes src/com/gypsyengineer/innerclass/field/*.java
$ javap -c -p classes/com/gypsyengineer/innerclass/field/Outer.class
Compiled from "Outer.java"
public class com.gypsyengineer.innerclass.field.Outer {
  private int secret;

  public com.gypsyengineer.innerclass.field.Outer();
    Code:
       0: aload_0
       1: invokespecial #2 // Method java/lang/Object."":()V
       4: aload_0
       5: bipush 10
       7: putfield #1 // Field secret:I
      10: return

  public void check();
    Code:
       0: aload_0
       1: getfield #1 // Field secret:I
       4: ifge 18
       7: getstatic #3 // Field java/lang/System.out:Ljava/io/PrintStream;
      10: ldc #4 // String Oops
      12: invokevirtual #5 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      15: goto 26
      18: getstatic #3 // Field java/lang/System.out:Ljava/io/PrintStream;
      21: ldc #6 // String Okay
      23: invokevirtual #5 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      26: return

  static int access$002(com.gypsyengineer.innerclass.field.Outer, int);
    Code:
       0: aload_0
       1: iload_1
       2: dup_x1
       3: putfield #1 // Field secret:I
       6: ireturn
}
```

Note `access$002` method which we didn't explicitly define in the source code. This is a synthetic method which was added by the Java compiler, and which provides access to the private field. You can find more details in&nbsp;[Accessing private fields with synthetic methods in Java](/en/security/accessing-private-fields-with-synthetic-methods-in-java.html)

Java 11 relaxes access checks for such cases. Here is what the [JEP authors say](http://openjdk.java.net/jeps/181):

> Introduce nests, an access-control context that aligns with the existing notion of nested types in the Java programming language. Nests allow classes that are logically part of the same code entity, but which are compiled to distinct class files, to access each other's private members without the need for compilers to insert accessibility-broadening bridge methods.

If we compile the code above with JDK 11 and run `javap`, then we'll get the following:

```
$ javac -d classes src/com/gypsyengineer/innerclass/field/*.java
$ /home/asmotrakov/tools/jdk/jdk-11b28/bin/javap -c -p classes/com/gypsyengineer/innerclass/field/Outer.class
Compiled from "Outer.java"
public class com.gypsyengineer.innerclass.field.Outer {
  private int secret;

  public com.gypsyengineer.innerclass.field.Outer();
    Code:
       0: aload_0
       1: invokespecial #1 // Method java/lang/Object."":()V
       4: aload_0
       5: bipush 10
       7: putfield #2 // Field secret:I
      10: return

  public void check();
    Code:
       0: aload_0
       1: getfield #2 // Field secret:I
       4: ifge 18
       7: getstatic #3 // Field java/lang/System.out:Ljava/io/PrintStream;
      10: ldc #4 // String Oops
      12: invokevirtual #5 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      15: goto 26
      18: getstatic #3 // Field java/lang/System.out:Ljava/io/PrintStream;
      21: ldc #6 // String Okay
      23: invokevirtual #5 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      26: return
}
```

The `access$002` method is gone, but nevertheless the JVM successfully runs the code.

You can find more details in [JEP 181](http://openjdk.java.net/jeps/181). It's a pretty big update in Java which affected the JVM specification, the Hotspot JVM, the Java compiler, Method Handles API, Reflection API, JVM TI, Java Instrumentation API, JDWP, JDI and even Pack200 (which is deprecated in Java 11 by the way). I hope this update doesn't have any security side effects :)

![]({{ site.baseurl }}/assets/images/2018/09/duke-small-e1537951462321.png)

