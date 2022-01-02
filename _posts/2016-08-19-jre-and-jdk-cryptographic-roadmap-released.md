---
layout: post
title: Java Cryptographic Roadmap
date: 2016-08-19 20:50:40.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- Security
permalink: "/en/security/jre-and-jdk-cryptographic-roadmap-released.html"
---
The Java team has published the “Oracle JRE and JDK Cryptographic Roadmap” at&nbsp;[http://java.com/en/jre-jdk-cryptoroadmap.html](http://java.com/en/jre-jdk-cryptoroadmap.html)&nbsp;which contains upcoming changes for&nbsp;crypto algorithms and protocols supported in Oracle's JRE and JDK.

![Java Cryptographic Roadmap]({{ site.baseurl }}/assets/images/2016/08/java_logo-300x300.png)

&nbsp;

JRE and JDK Cryptographic Roadmap describes&nbsp;the planned changes for crypto algorithms and protocols supported in Oracle's JRE and JDK. This roadmap contains the most up-to-date information about&nbsp;the planned changes to all supported Java versions which currently are&nbsp;JDK 8, JDK 7, JDK 6, and JRockit 6 (R28). It also contains information about Early Access builds of next feature release which currently is JDK 9.

## Upcoming planned updates

According to&nbsp;"Planned Changes", next update is planned to Jan 2017. It's going to restrict key sizes for EC and DSA algorithms for JDK&nbsp;8, 7, 6, and JRockit 6 (R28):

1. For SSL/TLS, disable EC for keys of less than 256 bits. Elsewhere, disable EC certificates with keys less than 224 bits. This has been already done in JDK&nbsp;9 b127 early access build in Jul 2016.
2. Increase the minimum key length for DSA certificates to 1024 bits. This has already done in JDK&nbsp;9 b107 in Feb 2016.

This planned update in Jan 2017 is also going to add TLS 1.1 and 1.2 protocols to the client list of default-enabled protocols. It's going to affect only JDK&nbsp;7, 6, and JRockit 6 (R28) because it's already done in JDK 8 and 9.

Next two planned updates in 2015 are smaller:

1. Disable SHA-1 in certificate chains anchored by roots included by default in Oracle's JDK 8, 7, 6, and&nbsp;JRockit 6 (R28). This update is planned&nbsp;to Apr 2017. This has been already done in JDK&nbsp;9&nbsp;b128 in July 2016.
2. For SSL/TLS, increase the minimum key length for Diffie-Hellman&nbsp;to 1024 bits. This update is planned to&nbsp;second half of 2017.&nbsp;This has already done in JDK&nbsp;9 b109 in Mar 2016.

## How to test upcoming updates

There is [a page](http://java.com/en/configure_crypto.html)&nbsp;which&nbsp;contains instructions for testing, enabling and disabling changes to Oracle's JDK and JRE announced on the Oracle JRE and JDK Cryptographic Roadmap. The roadmap also&nbsp;contains links to this page in "How to test/enable change" column.

Most of changes can be enabled or disabled by editing properites in "java.security" file. This file is located in "${java.home}/conf/security/java.security" directory in JDK9, and "${java.home}/lib/security/java.security" in older releases.

But it's a little bit more complicated with SSL/TLS protocols. The page contains four options for&nbsp;changing the default client-side TLS protocol version in the JDK.

First option&nbsp;is setting "jdk.tls.client.protocols" system property. The system property can be specified as a command line option when an application is starting which doesn't require modification of application code. Or, the system property can be set with System.setProperty() method, but this should be done before JSSE is initialized because JSSE reads system properties only once while initialization. In other words, this system property has to be updated before SSL/TLS related classes are used first time in your application.

Three other options require modification of application's code which may be not very convenient:

1. Passing&nbsp;protocol version to&nbsp;SSLContext.getInstance() method while creating an SSLContext.
2. Setting protocol version for&nbsp;SSLSocket and SSLEngine instances with setEnabledProtocols() method.
3. Setting protocol version with in&nbsp;SSLParameters instance with setProtocols() method, and them passing this SSLParameters to SSLSocket or SSLEngine objects.

That's pretty much it. Have fun!

