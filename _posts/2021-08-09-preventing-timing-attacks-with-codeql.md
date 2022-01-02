---
layout: post
title: Preventing a timing attack with CodeQL
date: 2021-08-09 18:23:08.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- CodeQL
- Java
- Security
- Vulnerability
permalink: "/en/security/preventing-timing-attacks-with-codeql.html"
---
A message authentication code (MAC) or a digital signature may be used to authenticate a message and to protect its integrity. When checking a signature, it is better to use constant-time algorithm. Otherwise, an attacker may be able to forge a valid signature for an arbitrary message by running a timing attack. Although it is a pretty sophisticated attack, sometimes it can be a real threat. Let's see how such issues may be detected with CodeQL in Java applications.

![Preventing timing attacks with CodeQL]({{ site.baseurl }}/assets/images/2021/08/jaelynn-castillo-xfNeB1stZ_0-unsplash.jpg)  

_Photo by Jaelynn Castillo_

  
  


## Signing a message

Here is an example scenario that shows how a signature can be used.

A sender and a receiver share a secret key. The sender calculates a signature over a message using the key. Next, the sender sends both the message and the signature. Then, the receiver calculates a signature over the received message and compares the calculated signature with the received one. If the signatures match, the receiver knows that the message was created by the sender who knows the key, and therefore accepts the message. If the signature doesn't match, the receiver rejects the message.

If an attacker doesn't know the key, they can't create a signature for an arbitrary message. Therefore, the attacker's messages are going to be rejected assuming that replay attacks are out of scope.

## Timing attack against signature

The problem occurs when an application doesn't use a constant-time algorithm for validating a signature. Here is an example of vulnerable code:

```java
public boolean validate(HttpRequest request, SecretKey key) throws Exception {
    byte[] message = getMessageFrom(request);
    byte[] signature = getSignatureFrom(request);

    Mac mac = Mac.getInstance("HmacSHA256");
    mac.init(new SecretKeySpec(key.getEncoded(), "HmacSHA256"));
    byte[] actual = mac.doFinal(message);
    return Arrays.equals(signature, actual);
}
```

The method `Arrays.equals()` returns `false` right away when it sees that one of the input's bytes are different. It means that the comparison time depends on the contents of the arrays. This little thing may allow an attacker to forge a valid signature for an arbitrary message byte by byte. If you want to learn more, check out [this video](https://www.coursera.org/lecture/crypto/timing-attacks-on-mac-verification-FHGW1) by Dan Boneh. By the way, if you don't know much about cryptography, but you are interested to learn more, I'd recommend his [free cource on Courcera](https://www.coursera.org/learn/crypto).

## Preventing a timing attack

It is usually a one-line fix: just use `MessageDigest.isEqual()` for validating a signature. The above code may be fixed just by replacing `Arrays.equals()` with `MessageDigest.isEqual()`:

```java
public boolean validate(HttpRequest request, SecretKey key) throws Exception {
    byte[] message = getMessageFrom(request);
    byte[] signature = getSignatureFrom(request);

    Mac mac = Mac.getInstance("HmacSHA256");
    mac.init(new SecretKeySpec(key.getEncoded(), "HmacSHA256"));
    byte[] actual = mac.doFinal(message);
    return MessageDigest.isEqual(signature, actual);
}
```

Even though timing attacks are quite difficult to run, it may be better to stay on the safe side and apply such one-line fixes to make sure timing attacks are not possible. It looks quite unlikely that such a fix can introduce a serious issue. Maybe you're going to notice some performance degradation since `MessageDigest.isEqual()` literally examines all the bytes, but the impact should be pretty small.

Besides `Arrays.equals()` there are many other methods that don't use a constant-time algorithm. For example, `String.equals()`. Here CodeQL comes into play.

## Detecting timing attacks with CodeQL

In case you don’t know, [CodeQL](https://securitylab.github.com/tools/codeql) is a code analysis engine. It lets you write queries for your code to detect various issues including security ones.

Recently, I wrote a couple of [CodeQL queries that](https://github.com/github/codeql/pull/6006) can detect opportunities for timing attacks in Java applications.

The first query `PossibleTimingAttackAgainstSignature.ql` is pretty simple:

```
from DataFlow::PathNode source, DataFlow::PathNode sink, NonConstantTimeCryptoComparisonConfig conf
where conf.hasFlowPath(source, sink)
select sink.getNode(), source, sink, "Possible timing attack against $@ validation.", source,
  source.getNode().(CryptoOperationSource).getCall().getResultType()
```

It looks for the following data flows:

1. A source of a data flow is methods of `MAC`, `Signature` and `Cipher` classes that can produce a signature. Strictly speaking, `Cipher` is used for encryption/decryption but it can be used to implement a custom MAC as well. That's why it is also considered here. Such sources are defined in `CryptoOperationSource` class and its subclasses.
2. A sink is one of the methods that doesn't use a constant time algorithm for comparing inputs. It covers not only `Array.equals()` but many other methods. `NonConstantTimeComparisonSink` defines such sinks.

The above is implemented in `NonConstantTimeCryptoComparisonConfig` for tracking data flows with CodeQL:

```
class NonConstantTimeCryptoComparisonConfig extends TaintTracking::Configuration {
  NonConstantTimeCryptoComparisonConfig() { this = "NonConstantTimeCryptoComparisonConfig" }

  override predicate isSource(DataFlow::Node source) { source instanceof CryptoOperationSource }

  override predicate isSink(DataFlow::Node sink) { sink instanceof NonConstantTimeComparisonSink }
}
```

Strictly speaking, it is not enough for a successful timing attack that just a non-constant-time algorithm is used for validating a signature. Additionally, an attacker has to be able to send to the receiver both a message and a signature. The query `PossibleTimingAttackAgainstSignature.ql` does not check that. But the second query `TimingAttackAgainstSignature.ql` does:

```
from DataFlow::PathNode source, DataFlow::PathNode sink, NonConstantTimeCryptoComparisonConfig conf
where
  conf.hasFlowPath(source, sink) and
  (
    source.getNode().(CryptoOperationSource).includesUserInput() and
    sink.getNode().(NonConstantTimeComparisonSink).includesUserInput()
  )
select sink.getNode(), source, sink, "Timing attack against $@ validation.", source,
  source.getNode().(CryptoOperationSource).getCall().getResultType()
```

The difference is that it calls `includesUserInput()` predicate on both source and sink:

1. `CryptoOperationSource.includesUserInput()` checks whether untrusted data is used to calculate a signature with `MAC`, `Signature` or `Cipher` classes.
2. `NonConstantTimeComparisonSink.includesUserInput()` checks whether untrusted data is used in the comparison procedure.

That makes the query much stricter than the first one, therefore it should produce less false positives. But of course, it is more likely to skip some true positives.

## References

- [MAC](https://en.wikipedia.org/wiki/Message_authentication_code)
- [Digital signature](https://en.wikipedia.org/wiki/Digital_signature)
- [Timing attack](https://en.wikipedia.org/wiki/Timing_attack)

