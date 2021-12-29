---
layout: post
title: Detecting dangerous Spring service exporters with CodeQL
date: 2021-03-25 14:12:41.000000000 +00:00
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
- Spring
- Vulnerability
meta:
  _edit_last: '1'
  _yoast_wpseo_primary_category: '157'
  _yoast_wpseo_estimated-reading-time-minutes: '4'
  _yoast_wpseo_content_score: '30'
  _oembed_bae1e08284d58bf516c13d8aa079e566: "{{unknown}}"
  _yoast_wpseo_focuskw: CVE-2016-1000027
  _yoast_wpseo_linkdex: '67'
  _yoast_wpseo_metadesc: How to use CodeQL to detect dangerous Spring service exporters.
    How to make sure that CVE-2016-1000027 doesn't affect application.
  rp4wp_auto_linked: '1'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617986213'

permalink: "/en/security/detecting-dangerous-spring-exporters-with-codeql.html"
---
<!-- wp:paragraph -->

In this blog post, I'll talk about detecting unsafe Spring Exporters with a CodeQL query. First, I'll describe the issue that received CVE-2016-1000027. Next, I'll show what a vulnerable code looks like and how the issue can be mitigated in an application. Then, I'll describe how the CodeQL query works. In addition, I'll show a couple of vulnerabilities that have been found by the query.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

(you can also read it on [Medium](https://infosecwriteups.com/detect-dangerous-spring-service-exporters-with-codeql-c3c800b7b2de))

<!-- /wp:paragraph -->

<!-- wp:image {"id":4072,"sizeSlug":"large","linkDestination":"none","className":"noborder"} -->

![Detecting dangerous Spring Exporters with CodeQL]({{ site.baseurl }}/assets/images/2021/03/codeql_spring.png)

<!-- /wp:image -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## What is a Spring Exporter?

<!-- /wp:heading -->

<!-- wp:paragraph -->

The Spring Framework provides classes for exporting a service bean as an endpoint. Service exporters read data from an incoming request and then pass the data to the underlying bean. The data can contain a serialized object.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

For example, `HttpInvokerServiceExporter` and `SimpleHttpInvokerServiceExporter` classes export a specified service bean as an HTTP endpoint. These exporters extend `RemoteInvocationSerializingExporter` that uses the default Java deserialization mechanism to parse data from an incoming request.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## What is CVE-2016-1000027?

<!-- /wp:heading -->

<!-- wp:paragraph -->

The default Java deserialization mechanism is available via `ObjectInputStream` class. This mechanism is known to be vulnerable. If an attacker can make an application deserialize malicious data, it may result in arbitrary code execution in the worst case.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Spring's `RemoteInvocationSerializingExporter` uses the default Java deserialization mechanism to parse data. As a result, all classes that extend it are vulnerable to deserialization attacks. The Spring Framework contains at least `HttpInvokerServiceExporter` and `SimpleHttpInvokerServiceExporter` that extend `RemoteInvocationSerializingExporter`. These exporters parse data from the HTTP body using the unsafe Java deserialization mechanism.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[The issue was discovered and reported by Tenable](https://www.tenable.com/security/research/tra-2016-20). The problem received [CVE-2016-1000027](https://nvd.nist.gov/vuln/detail/CVE-2016-1000027). The Spring team deprecated the vulnerable classes and added a note to the documentation that warns users about the [issue](https://github.com/spring-projects/spring-framework/issues/24434). They recommended to avoid using the dangerous exporters.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

To be honest, it doesn't seem possible to fix the issue without breaking applications that use the vulnerable exporters. The usual fix for such issues is to check if a serialized object is an instance of an allowed class before deserializing it. This can be done by overriding `ObjectInputStream.resolveClass()` method or by using `ValidatingObjectInputStream` from Apache Commons IO. The key thing here is the default list of allowed classes. If the default list is restrictive, the fix is likely to break applications. If the list is permissive, then the applications are still vulnerable by default, and the users need to explicitly specify the allowed safe classes to get rid of the vulnerability. The Spring team decided not to offer a way for configuring allowed classes for deserializing in the vulnerable exporters.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Once CVE-2016-1000027 was published in NVD, security scanners started reporting this issue against applications that use the Spring Framework. Since the Spring Framework is quite popular these days, many users received such alerts. The problem for many users was that they didn't know what to do with these alerts. The issue couldn't be solved by updating the Spring Framework version in their applications because the Spring Framework did not have a fix for that.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Example of vulnerable code

<!-- /wp:heading -->

<!-- wp:paragraph -->

It is quite easy to make an application vulnerable by using one of the unsafe service exporters. The following example shows a vulnerable HTTP endpoint that is based on `HttpInvokerServiceExporter`:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/8f9d666cc58ccff9dd50d42a3cc1d68d.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

The next example shows how the same endpoint can be defined in an XML config for the Spring Framework:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/5ea4d67c9ee5ecb51cbfc591a383df89.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

For demo purposes, I also wrote a simple [PoC for CVE-2016-1000027](https://github.com/artem-smotrakov/cve-2016-1000027-poc). This is a simple Spring application that uses a vulnerable service exporter. The repository contains a demo exploit for the vulnerable endpoint.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## How to mitigate CVE-2016-1000027

<!-- /wp:heading -->

<!-- wp:paragraph -->

The best way is to stop using `HttpInvokerServiceExporter` and `SimpleHttpInvokerServiceExporter` or any other exporter that is based on `RemoteInvocationSerializingExporter`. Instead, one of the other message formats for API endpoints can be used. For example, JSON. Make sure that the underlying deserialization mechanism is properly configured so that deserialization attacks are not possible.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

If the vulnerable exporters can not be replaced, consider using global deserialization filters introduced in [JEP 290](https://openjdk.java.net/jeps/290).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## CodeQL query for detecting unsafe Spring exporters

<!-- /wp:heading -->

<!-- wp:paragraph -->

[CodeQL](https://securitylab.github.com/tools/codeql) is a code analysis engine. It lets you write queries for your code to detect various issues including security ones. Let's see how it can help us detect unsafe Spring service exporters.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

As shown in the code snippets above, an unsafe service exporter may be defined in two ways:

<!-- /wp:paragraph -->

<!-- wp:list -->

- A method in a Spring configuration class that creates a bean
- A bean in an XML config

<!-- /wp:list -->

<!-- wp:paragraph -->

CodeQL can cover both ways. That’s exactly what we need.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

In a Spring configuration class, we need to look for the following methods:

<!-- /wp:paragraph -->

<!-- wp:list -->

- The class should have one of the annotations that make it a config. For example, `@Configuration` annotation.
- The method should return an instance of a class that extends `RemoteInvocationSerializingExporter`.
- The method should have `@Bean` annotation.

<!-- /wp:list -->

<!-- wp:paragraph -->

The below CodeQL query implements these ideas:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/0889c86b2e33a4fd092ca9e5769c53a4.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

In an XML configuration, we just need to look for `<bean>` elements that use a class that extends `RemoteInvocationSerializingExporter` in the `class` property. The following simple CodeQL query implements this:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/fd504f83eb1559924d358ed173b1a184.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

If a security scanner reports CVE-2016-1000027 against your application, you can use the above CodeQL queries to check if the application is really vulnerable. [These queries have been also added to the CodeQL experimental query pack](https://github.com/github/codeql/pull/5260).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## References

<!-- /wp:heading -->

<!-- wp:list -->

- [Remoting and web services using Spring](https://docs.spring.io/spring-framework/docs/2.0.x/reference/remoting.html)
- [RemoteInvocationSerializingExporter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/remoting/rmi/RemoteInvocationSerializingExporter.html)
- [HttpInvokerServiceExporter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/remoting/httpinvoker/HttpInvokerServiceExporter.html)
- [CodeQL documentation](https://codeql.github.com/docs/)

<!-- /wp:list -->

