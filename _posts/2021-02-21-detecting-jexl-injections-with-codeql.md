---
layout: post
title: Detecting JEXL injections with CodeQL
date: 2021-02-21 12:11:16.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- CodeQL
- EL Injection
- Java
- Security
- Vulnerability
permalink: "/en/security/detecting-jexl-injections-with-codeql.html"
---
In this post, I'll talk about a CodeQL query for detecting JEXL Expression Language injection vulnerabilities. First, I'll give a brief overview of expression languages in general and JEXL in particular. Next, I'll explain what Expression Language injection is and how to prevent it. Then, I'll describe how the CodeQL query works. In addition, I'll show a couple of vulnerabilities that have been found by the query.

(you can also read it on [Medium](https://betterprogramming.pub/expression-language-injections-in-java-e08bd17addf4))

![Detecting JEXL injections with CodeQL]({{ site.baseurl }}/assets/images/2021/02/cowsay_jexl_injections_codeql.png)

  
  


## What is Expression Language?

Expression Language (EL) is a general-purpose programming language mostly used for embedding and evaluating expressions at runtime. Most often, they are interpreted languages. In other words, there is an interpreter that prepares an execution context and runs an expression in it. Depending on a particular application, expressions can be created by developers, admins or even regular users for various purposes. Embedded expressions often help customize and configure an application.

There are multiple implementations of EL. Here are several examples:

- Java Server Pages (JSP) allows using expressions in a number of tags.
- Spring Framework offers Spring Expression Language (SpEL) that is allowed in Spring tags, configurations and other contexts.
- Apache Struts allows using expressions in multiple contexts. It uses Object Graph Navigation Library (OGNL).

Normally, Expression Languages offer many simple operations such as addition, multiplication, string concatenation and so on. Besides that, some EL interpreters allow calling methods on objects that are available in the evaluation context. Furthermore, some interpreters allow accessing any class in the JVM and calling their methods. But that may be quite dangerous.

## What is JEXL?

JEXL stands for Java Expression Language. It is a library that offers API for evaluating JEXL expressions. The project belongs to Apache Software Foundation.

The most interesting fact for us is that JEXL is powerful enough to let us calling classes and methods that are available in the JVM. The following code calls `ProcessBuilder` to start an external process:

 
<script src="https://gist.github.com/artem-smotrakov/2cc44e0688988aec4d4ead5ca7278e5f.js"></script>  


## What is Expression Language injection?

If an application allows expressions to access classes and methods available in the JVM, and if a malicious user can feed (or inject to) the application an arbitrary expression, it often leads to arbitrary code execution. That's called Expression Language Injection vulnerability. The impact of this type of issue is usually high because the attacker can run almost any code inside or even outside the JVM. The likelihood of a successful attack depends on the way how the attacker can inject a malicious expression. In the worst case, an application reads and runs an expression from a request that is received from an unauthenticated remote user, for example, using HTTP. That usually results in Remote Code Execution (RCE). Besides that, applications can get expressions from files, databases, and other places.

Here is an example of vulnerable code. It just takes an expression from an HTTP request and immediately runs it:

 
<script src="https://gist.github.com/artem-smotrakov/cbed722339f8da942a455c3dc8b7383b.js"></script>  


## Preventing JEXL injections

First of all, I'd recommend thinking twice before introducing expressions in an application. It may happen that they are not really necessary, and there is a better (simpler?) design. If introducing expressions is unavoidable, expressions should run in a restrictive sandbox. Fortunately, JEXL offers two ways for implementing a sandbox. The first way is to use the `JexlSandbox` class. The class lets specifying which classes are allowed or prohibited to call from a JEXL expression. Make sure that you pass `false` to the constructor. It means that all method calls are prohibited unless they are explicitly allowed. Here is an example:

 
<script src="https://gist.github.com/artem-smotrakov/5c8709fd6abe7f1a76be9d82fc838200.js"></script>  


The second way is to create a custom sandbox by implementing the `JexlUberspect` interface. The main purpose of this class is introspection, but it may be used for creating sandboxes as well. This way is more flexible because we are not limited by the APIs offered by `JexlSandbox`. Here is an example:

 
<script src="https://gist.github.com/artem-smotrakov/aadd6b7d91bfa330fdafbab3ea22e87d.js"></script>  


## CodeQL query for detecting JEXL injections

[CodeQL](https://securitylab.github.com/tools/codeql) is a code analysis engine. It lets you write queries for your code to detect various issues including security ones. Let's see how it can help us detect JEXL injections.

CodeQL can look for data flows from sources to sinks that we specify. In the case of JEXL injections, the sinks are method calls that trigger evaluating JEXL expressions such as `JexlExpression.evaluate()`, `JexlScript.execute()` and several others. The most interesting data sources are the ones that are available for remote users. For example, network sockets and HTTP requests. A data flow from a remote source to one of the JEXL sinks may be a potential JEXL injection. I've implemented this idea in a CodeQL [query](https://github.com/github/codeql/pull/4965). Let's have a look at its main components.

To search for data flows, I wrote a config for tracking tainted data from remote data sources to JEXL sinks. Here is what it looks like:

 
<script src="https://gist.github.com/artem-smotrakov/2206ef252a8c9a74a71e452c513394a9.js"></script>  


It has three main parts:

- `isSource()` predicate defines data sources. Here it uses the `RemoteFlowSource` class that describes network sockets, HTTP requests and other remote data sources which the CodeQL core library is aware of.
- `isSink()` predicate defines data sinks. Here it uses the `JexlEvaluationSink` class that lists method calls that trigger evaluating JEXL expressions.
- `isAdditionalTaintStep()` predicate tells the CodeQL engine about additional ways how tainted input can be propagated.

Let's have a closer look at `JexlEvaluationSink`:

 
<script src="https://gist.github.com/artem-smotrakov/3e9d52d44b50cd4d07a72b7a86bbd495.js"></script>  


It defines two groups of method invocations that can trigger evaluation of JEXL expressions:

- Methods that evaluate expressions immediately. For example, `JexlExpression.evaluate()` and `JexlEngine.getProperty()`.
- Methods for deferred evaluation of an expression: `JexlExpression.callable()` and `JexlScript.callable()`. Those methods don't evaluate expressions right away. The query assumes that the `Callable.call()` method will be called later that will trigger the evaluation.

The `isAdditionalTaintStep()` predicate defines two additional ways for propagating tainted data:

- The `TaintPropagatingJexlMethodCall` class defines methods that compile JEXL expressions, scripts and templates.
- `returnsDataFromBean()` predicate defines calls to getters on beans that may hold tainted data.

Finally, the query checks whether a sandboxed JEXL engine is used or not. It is implemented in the `isUnsafeEngine()` predicate and a separate data flow config `SandboxedJexlFlowConfig` that checks whether `JexlSandbox` or `JexlUberspect` have been set for the JEXL engine.

There are a couple of known issues with the current query:

- The `returnsDataFromBean()` predicate helps to identify issues when data is stored in complex structures. However, it doesn't take into account how data flows in a bean. As a result, it may potentially cause false positives.
- It would be good if the query checked that the sandbox is implemented property. Unfortunately, I didn't find a reliable solution for that. Currently, the query doesn't make sure if a `JexlUberspect` actually implements a sandbox, or if a `JexlSandbox` is not too permissive. Instead, it just assumes the sandbox is properly implemented. Therefore, it may potentially result in false negatives.

The query found a couple of interesting issues.

## CVE-2021-3396: RCE in OpenNMS

The first finding is an RCE in [OpenNMS/newts](https://github.com/OpenNMS/newts) project ([CVE-2021-3396](https://nvd.nist.gov/vuln/detail/CVE-2021-3396)). The server offers `/measurements/{resource}` [endpoint](https://github.com/OpenNMS/newts/blob/d706eb9e12783b31f4745c232a3cfa212474e68a/rest/src/main/java/org/opennms/newts/rest/MeasurementsResource.java#L62):

```java
@POST
@Path("/{resource}")
@Timed
public Collection<Collection<MeasurementDTO>> getMeasurements(
        ResultDescriptorDTO descriptorDTO,
        @PathParam("resource") Resource resource,
        @QueryParam("start") Optional<TimestampParam> start,
        @QueryParam("end") Optional<TimestampParam> end,
        @QueryParam("resolution") Optional<DurationParam> resolution,
        @QueryParam("context") Optional<String> contextId) {
```

The endpoint accepts POST requests with a number of parameters. Most of them come from the URL path and query string. Besides them, the method  
`getMeasurements()` accepts an instance of `ResultDescriptorDTO`. This parameter doesn't have any annotation, but it comes from a body of  
the POST request. The `ResultDescriptorDTO` class holds an array of JEXL  
expressions. When the endpoint is invoked, those JEXL  
expressions go to a `JexlEngine` and get executed immediately. The engine is not configured with a sandbox. As a result, a remote user can run arbitrary code on the server. The issue has been [fixed in newts 1.5.3](https://www.opennms.com/en/blog/2021-02-16-cve-2021-3396-full-security-disclosure/) by [sandboxing](https://github.com/OpenNMS/newts/pull/49) JEXL expressions.

## RCE in Traccar

The second finding is an [RCE in Traccar project](https://github.com/traccar/traccar/issues/4624). The server has an undocumented [endpoint](https://github.com/OpenNMS/newts/blob/d706eb9e12783b31f4745c232a3cfa212474e68a/rest/src/main/java/org/opennms/newts/rest/MeasurementsResource.java#L62)  
`/attributes/computed/test` that accepts a device id and a structure with an attribute:

```java
@POST
@Path("test")
public Response test(@QueryParam("deviceId") long deviceId, Attribute entity) {
```

The attribute has an `expression` field that contains a JEXL expression. The expression goes directly to a `JexlEngine` that doesn't use a sandbox. This issue may be difficult to exploit because an attacker must be an admin to access the endpoint. Therefore, the project maintainers don't treat the issue as a serious one.

The query can be extended to consider more data sources such as files. That would definitely found more issues. However, such issues are not that interesting because it is much more difficult for an attacker to take advantage of them. For example, Apache JMeter doesn't sandbox JEXL expressions that come from a test plan. Implementing a sandbox is considered a security [enhancement](https://bz.apache.org/bugzilla/show_bug.cgi?id=65151).

## References

- [Expression Language injections (OWASP)](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [Apache JEXL](https://commons.apache.org/proper/commons-jexl/)
- [CodeQL documentation](https://codeql.github.com/docs/)



