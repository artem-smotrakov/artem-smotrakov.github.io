---
layout: post
title: Detecting dangerous RMI objects with CodeQL
date: 2021-06-02 15:46:45.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- CodeQL
- Deserialization
- Java
- Security
- Vulnerability
permalink: "/en/security/detecting-dangerous-rmi-objects-with-codeql.html"
---
Java RMI uses the default Java deserialization mechanism for passing parameters during remote method invocations. In other words, RMI uses `ObjectInputStream` that is a well-known unsafe deserialization mechanism. If an attacker can find and send a deserialization gadget to a vulnerable remote method, in the worst case it can result in arbitrary code execution.

I recently wrote a CodeQL [query](https://github.com/github/codeql/pull/5818) that looks for dangerous remote objects registered in an RMI registry. This post describes the vulnerability and how the query works.

  
  


Luckily, not all RMI methods are vulnerable. Making a long story shorter, to be vulnerable, a remote method has to accept a complex object as a parameter. If it accepts only primitive types, strings and a few other types from the Java core library, then the method is safe. You can find more details in the following articles:

- [Attacking Java RMI services after JEP 290](https://mogwailabs.de/en/blog/2019/03/attacking-java-rmi-services-after-jep-290/)
- [Java RMI for pentesters part two - reconnaissance & attack against non-JMX registries](https://itnext.io/java-rmi-for-pentesters-part-two-reconnaissance-attack-against-non-jmx-registries-187a6561314d)

Here is an example of vulnerable code:

 
<script src="https://gist.github.com/artem-smotrakov/5b9ec83e0b04d05aaf5ac5d808452d78.js"></script>  


`RemoteObject.action()` is vulnerable because it accepts a complex parameter. The vulnerability here can be fixed by specifying a deserialization filter introduced by JEP 290. Here is an example:

 
<script src="https://gist.github.com/artem-smotrakov/886638320b0db2b43a7b22cd2499c86f.js"></script>  


It is also possible to configure a global deserialization filter by calling `ObjectInputFilter.Config.setSerialFilter(ObjectInputFilter)` method or by setting `jdk.serialFilter` system or security property. Make sure that you use Java version that contains JEP 290. I put some more examples of vulnerable code, demo exploits and mitigation in this [rep](https://github.com/artem-smotrakov/ql-fun/tree/master/src/main/java/com/gypsyengineer/ql/fun/java/rmi)[o](https://github.com/artem-smotrakov/ql-fun/tree/master/src/main/java/com/gypsyengineer/ql/fun/java/rmi)[sitory](https://github.com/artem-smotrakov/ql-fun/tree/master/src/main/java/com/gypsyengineer/ql/fun/java/rmi).

Now, let's have a look at the CodeQL [query](https://github.com/github/codeql/pull/5818) that detects such vulnerabilities. The main part is a configuration for tracking data flows from constructing dangerous remote object to registering them in an RMI registry:

 
<script src="https://gist.github.com/artem-smotrakov/7dec100471524152743f143220e8dbc4.js"></script>  


A source of such a data flow is a constructor call for a type that has a vulnerable method. The predicate `hasVulnerableMethod()` checks whether a class has vulnerable methods or not.

A sink is a method call to one of the methods that registers a remote object in an RMI registry. For example, `Registry.bind()` or `Registry.rebind()`.

The predicate `isAdditionalTaintStep()` adds an additional taint-propagation step. If a remote object doesn't extend `UnicastRemoteObject` class, then it has to be exported by calling one of the `UnicastRemoteObject.exportObject()` methods before registering the object in a registry. This operation is covered by the predicate. Plus, this predicate plays a role of a sanitizer because it propagates taint only if `exportObject()` was called without a deserialization filter.

The query detected multiple issues in various open source projects on GitHub. Here are some examples:

- [RMI deserialization vulnerability in PeerUnit/mock-dht](https://github.com/PeerUnit/mock-dht/issues/2)
- [RMI deserialization vulnerability in TubeDB](https://github.com/environmentalinformatics-marburg/tubedb/issues/10#issuecomment-840400709)
- [RMI deserialization vulnerability in Weka](https://github.com/Waikato/weka-trunk/issues/23)

More examples of alerts can be found on [LG](https://lgtm.com/query/5242868053583474640/)[T](https://lgtm.com/query/5242868053583474640/)[M](https://lgtm.com/query/5242868053583474640/). Finally, the query was able to detect [CVE-2016-2170](https://nvd.nist.gov/vuln/detail/CVE-2016-2170) in older Apache OFBiz releases.

