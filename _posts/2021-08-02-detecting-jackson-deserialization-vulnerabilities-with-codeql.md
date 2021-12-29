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
meta:
  _edit_last: '1'
  _yoast_wpseo_primary_category: '157'
  _yoast_wpseo_focuskw: deserialization vulnerabilities with Jackson
  _yoast_wpseo_linkdex: '71'
  _yoast_wpseo_estimated-reading-time-minutes: '4'
  _yoast_wpseo_content_score: '30'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1627217672'
  _yoast_wpseo_metadesc: How CodeQL can help with detecting deserialization vulnerabilities
    in an application that uses Jackson serialization framework.
  rp4wp_auto_linked: '1'
  _wp_old_date: '2021-07-25'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/security/detecting-jackson-deserialization-vulnerabilities-with-codeql.html"
---
<!-- wp:paragraph -->

If you use Jackson Databind library and run a security scanner, you might have received quite a lot of alerts about deserialization vulnerabilities. In the past, a new CVE pop up nearly every month when someone discovered a new deserialization gadget that could be used to exploit an application. Fortunately, the project doesn't assign CVEs for new deserialization gadgets anymore. It makes sense because an application that uses Jackson libraries is not vulnerable by default. However, if the application uses Jackson libraries in a certain way, it may be in danger.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Below I'll show how CodeQL can be used to to check whether or not an application is vulnerable to deserialization attacks.

<!-- /wp:paragraph -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## What makes an application vulnerable?

<!-- /wp:heading -->

<!-- wp:paragraph -->

Here is a list of conditions that have to be met to make an application vulnerable to deserialization attack:

<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->

1. The application deserializes data that comes from an attacker.
2. The application uses one of the libraries that contains a dangerous gadget.
3. The application enables polymorphic type handling.
4. The application uses fields of generic types such as `Object`, `Serializable`, `Cloneable` and so on.

<!-- /wp:list -->

<!-- wp:paragraph -->

If one of the conditions above is not met, the application is not vulnerable. You can find more details in [Tatu's article](https://cowtowncoder.medium.com/on-jackson-cves-dont-panic-here-is-what-you-need-to-know-54cd0d6e8062) or [my blog post](/en/security/safer-deserialization-with-new-jackson.html). Below is an example of vulnerable code:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
byte[] data = new byte[1024];
socket.read(data); // data can come from a remote attacker
ObjectMapper mapper = new ObjectMapper();
mapper.enableDefaultTyping(); // this enables polymorphic type handling
Cat cat = mapper.readValue(data, Cat.class); // deserialization happens here
```

<!-- /wp:code -->

<!-- wp:heading -->

## Detecting Jackson deserialization issues with CodeQL

<!-- /wp:heading -->

<!-- wp:paragraph -->

In case you don't know, [CodeQL](https://securitylab.github.com/tools/codeql) is a code analysis engine. It lets you write queries for your code to detect various issues including security ones.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

There is already a set of CodeQL queries for many security issues. In particular, the set contains a query for detecting unsafe deserialization. The query covers many serialization frameworks including Kryo, XStream, SnakeYaml and others. I recently updated the query to [support Jackson](https://github.com/github/codeql/pull/5900) as well. Let's have a look at how the query detects deserialization vulnerabilities.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The query tracks data flows from remote data sources to a deserialization sink and checks that the sink triggers unsafe deserialization. It is implemented in `UnsafeDeserializationConfig`:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

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

<!-- /wp:code -->

<!-- wp:paragraph -->

The predicate `unsafeDeserialization()` checks in a method call triggers unsafe deserialization with untrusted data. I updated the predicate with the following:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

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

<!-- /wp:code -->

<!-- wp:paragraph -->

Let's have a closer look at these conditions.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The first two lines make sure that `ma` is method call of one of the methods in `ObjectMapper` that deserializes data that comes in the first parameter:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
ma.getMethod() instanceof ObjectMapperReadMethod and
sink = ma.getArgument(0) and
```

<!-- /wp:code -->

<!-- wp:paragraph -->

The next block checks whether deserialization is dangerous or not:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
exists(UnsafeTypeConfig config | config.hasFlowToExpr(ma.getAnArgument()))
or
exists(EnableJacksonDefaultTypingConfig config | config.hasFlowToExpr(ma.getQualifier()))
or
hasArgumentWithUnsafeJacksonAnnotation(ma)
```

<!-- /wp:code -->

<!-- wp:paragraph -->

If one of the below conditions is met, then deserialization is unsafe:

<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->

1. `UnsafeTypeConfig` checks if an attacker can control both data and type of the object being deserialized. CVE-2016-8749 is a nice example.
2. `EnableJacksonDefaultTypingConfig` checks if the default type handing is enabled, for example, by calling `ObjectMapper.enableDefaultTyping()` method.
3. `hasArgumentWithUnsafeJacksonAnnotation()` checks if polymorphic type handling is enabled via `JsonTypeInfo` annotation.

<!-- /wp:list -->

<!-- wp:paragraph -->

The last line checks that `ObjectMapper` does not have a type validator that can prevent deserialization attacks:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
not exists(SafeObjectMapperConfig config | config.hasFlowToExpr(ma.getQualifier()))
```

<!-- /wp:code -->

<!-- wp:heading -->

## How to avoid deserialization vulnerabilities with Jackson

<!-- /wp:heading -->

<!-- wp:list {"ordered":true} -->

1. You must have heard it before but still: don't let an application deserialize data from untrusted sources.
2. Avoid using polymorphic type handling.
3. Configure `ObjectMapper` with a type validator that allows deserialization of trusted classes only. For more details, check out [Tatu's article](https://cowtowncoder.medium.com/jackson-2-10-features-cd880674d8a2) and [my blog post](/en/security/safer-deserialization-with-new-jackson.html).

<!-- /wp:list -->

<!-- wp:heading -->

## Conclusion

<!-- /wp:heading -->

<!-- wp:paragraph -->

If an application uses Jackson libraries, it does not always mean it is vulnerable to deserialization attacks. But you'd better check it. CodeQL now can help you with it.

<!-- /wp:paragraph -->

