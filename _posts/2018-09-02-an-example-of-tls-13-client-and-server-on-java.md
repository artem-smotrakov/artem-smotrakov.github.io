---
layout: post
title: An example of TLS 1.3 client and server on Java
date: 2018-09-02 23:43:11.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- JSSE
- TLS 1.3
permalink: "/en/security/an-example-of-tls-13-client-and-server-on-java.html"
---
Java 11 supports TLS 1.3 protocol which was published in August 2018. During implementing the new TLS protocol, Java security-libs team significantly [re-worked Java Secure Sockets Extension (JSSE)](https://bugs.openjdk.java.net/browse/JDK-8185576). I used to work on security-libs in Java for 6 years, so I can tell that was not an easy task for sure. But nevertheless, Java security-libs team delivered TLS 1.3 implementation in Java 11. Great job!

But TLS 1.3 implementation in Java 11 doesn't not support all the features of the new TLS protocol. Here is what JSSE supports (see more details in [JEP 332](http://openjdk.java.net/jeps/332)):

- Protocol version negotiation
- Full handshake for both client and server sides
- Session resumption
- Key and IV update
- Updated OCSP stapling
- Backward compatibility mode
- Required extensions and algorithms
- Two new cipher suites: TLS\_AES\_128\_GCM\_SHA256 and TLS\_AES\_256\_GCM\_SHA384
- RSASSA-PSS signature algorithms
- Both SSLSocket and SSLEngine

And here is what are not supported:

- 0-RTT data
- Post-handshake authentication
- Signed certificate timestamps (SCT)
- ChaCha20/Poly1305 cipher suites ([targeted to Java 12](https://bugs.openjdk.java.net/browse/JDK-8140466))
- x25519/x448 elliptic curve algorithms ([targeted to Java 12](https://bugs.openjdk.java.net/browse/JDK-8171279))
- edDSA signature algorithms ([targeted to Java 12](https://bugs.openjdk.java.net/browse/JDK-8166596))

Java 11 doesn't introduce new public classes and methods for TLS 1.3. It just adds a couple of new constants for the new protocol name, cipher suites, ets. And this is actually great because it makes it very easy to switch to the new TLS version. You just need to configure JSSE to use new protocol and ciphers, and the rest of the code should not change. Depending on an application, migration to the new version may not even require any modification of the application code. For example, if an application is configured by JSSE system properties such as `https.protocols`&nbsp;and `jdk.tls.client.protocols`. (well, if third-parties you'l like to talk to with TLS 1.3 don't support it, then the migration may not be that easy).

Here is an example of TLS 1.3 client and server in Java. As you may notice, it's just a regular example of SSLSocket-based client and server except it uses new constants "TLSv1.3" and "TLS\_AES\_128\_GCM\_SHA256".

<script src="https://gist.github.com/artem-smotrakov/bd14e4bde4d7238f7e5ab12c697a86a3.js"></script>

