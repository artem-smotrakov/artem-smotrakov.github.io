---
layout: post
title: Detecting Jackson deserialization vulnerabilities with CodeQL
date: 2021-08-02 14:46:11.000000000 +01:00
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
- Jackson
- Java
- Security
- Vulnerability
permalink: "/en/security/detecting-jackson-deserialization-vulnerabilities-with-codeql.html"
---
If you use Jackson Databind library and run a security scanner, you might have received quite a lot of alerts about deserialization vulnerabilities. In the past, a new CVE pop up nearly every month when someone discovered a new deserialization gadget that could be used to exploit an application. Fortunately, the project doesn't assign CVEs for new deserialization gadgets anymore. It makes sense because an application that uses Jackson libraries is not vulnerable by default. However, if the application uses Jackson libraries in a certain way, it may be in danger.

Below I'll show how CodeQL can be used to to check whether or not an application is vulnerable to deserialization attacks.

  
  


## What makes an application vulnerable?

Here is a list of conditions that have to be met to make an application vulnerable to deserialization attack:

1. The application deserializes data that comes from an attacker.
2. The application uses one of the libraries that contains a dangerous gadget.
3. The application enables polymorphic type handling.
4. The application uses fields of generic types such as `Object`, `Serializable`, `Cloneable` and so on.

If one of the conditions above is not met, the application is not vulnerable. You can find more details in [Tatu's article](https://cowtowncoder.medium.com/on-jackson-cves-dont-panic-here-is-what-you-need-to-know-54cd0d6e8062) or [my blog post](/en/security/safer-deserialization-with-new-jackson.html). Below is an example of vulnerable code:

```
byte[] data = new byte[1024];
socket.read(data); // data can come from a remote attacker
ObjectMapper mapper = new ObjectMapper();
mapper.enableDefaultTyping(); // this enables polymorphic type handling
Cat cat = mapper.readValue(data, Cat.class); // deserialization happens here
```

## Detecting Jackson deserialization issues with CodeQL

In case you don't know, [CodeQL](https://securitylab.github.com/tools/codeql) is a code analysis engine. It lets you write queries for your code to detect various issues including security ones.

There is already a set of CodeQL queries for many security issues. In particular, the set contains a query for detecting unsafe deserialization. The query covers many serialization frameworks including Kryo, XStream, SnakeYaml and others. I recently updated the query to [support Jackson](https://github.com/github/codeql/pull/5900) as well. Let's have a look at how the query detects deserialization vulnerabilities.

The query tracks data flows from remote data sources to a deserialization sink and checks that the sink triggers unsafe deserialization. It is implemented in `UnsafeDeserializationConfig`:

```
class UnsafeDeserializationSink extends DataFlow::ExprNode {
  UnsafeDeserializationSink() { unsafeDeserialization(_, this.getExpr()) }

  /** Get a call that triggers unsafe deserialization. */
  MethodAccess getMethodAccess() { unsafeDeserialization(result, this.getExpr()) }
}

/**
 * Tracks flows from remote user input to a deserialization sink.
 */
class UnsafeDeserializationConfig extends TaintTracking::Configuration {
  UnsafeDeserializationConfig() { this = "UnsafeDeserializationConfig" }

  override predicate isSource(DataFlow::Node source) { source instanceof RemoteFlowSource }

  override predicate isSink(DataFlow::Node sink) { sink instanceof UnsafeDeserializationSink }
```

The predicate `unsafeDeserialization()` checks in a method call triggers unsafe deserialization with untrusted data. I updated the predicate with the following:

```
ma.getMethod() instanceof ObjectMapperReadMethod and
sink = ma.getArgument(0) and
(
  exists(UnsafeTypeConfig config | config.hasFlowToExpr(ma.getAnArgument()))
  or
  exists(EnableJacksonDefaultTypingConfig config | config.hasFlowToExpr(ma.getQualifier()))
  or
  hasArgumentWithUnsafeJacksonAnnotation(ma)
) and
not exists(SafeObjectMapperConfig config | config.hasFlowToExpr(ma.getQualifier()))
```

Let's have a closer look at these conditions.

The first two lines make sure that `ma` is method call of one of the methods in `ObjectMapper` that deserializes data that comes in the first parameter:

```
ma.getMethod() instanceof ObjectMapperReadMethod and
sink = ma.getArgument(0) and
```

The next block checks whether deserialization is dangerous or not:

```
exists(UnsafeTypeConfig config | config.hasFlowToExpr(ma.getAnArgument()))
or
exists(EnableJacksonDefaultTypingConfig config | config.hasFlowToExpr(ma.getQualifier()))
or
hasArgumentWithUnsafeJacksonAnnotation(ma)
```

If one of the below conditions is met, then deserialization is unsafe:

1. `UnsafeTypeConfig` checks if an attacker can control both data and type of the object being deserialized. CVE-2016-8749 is a nice example.
2. `EnableJacksonDefaultTypingConfig` checks if the default type handing is enabled, for example, by calling `ObjectMapper.enableDefaultTyping()` method.
3. `hasArgumentWithUnsafeJacksonAnnotation()` checks if polymorphic type handling is enabled via `JsonTypeInfo` annotation.

The last line checks that `ObjectMapper` does not have a type validator that can prevent deserialization attacks:

```
not exists(SafeObjectMapperConfig config | config.hasFlowToExpr(ma.getQualifier()))
```

## How to avoid deserialization vulnerabilities with Jackson

1. You must have heard it before but still: don't let an application deserialize data from untrusted sources.
2. Avoid using polymorphic type handling.
3. Configure `ObjectMapper` with a type validator that allows deserialization of trusted classes only. For more details, check out [Tatu's article](https://cowtowncoder.medium.com/jackson-2-10-features-cd880674d8a2) and [my blog post](/en/security/safer-deserialization-with-new-jackson.html).

## Conclusion

If an application uses Jackson libraries, it does not always mean it is vulnerable to deserialization attacks. But you'd better check it. CodeQL now can help you with it.

