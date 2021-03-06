---
layout: post
title: Detecting Jakarta Expression Language injections with CodeQL
date: 2021-04-14 18:10:25.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- EL Injection
- Java
- Security
- Vulnerability
permalink: "/en/security/detecting-jakarta-expression-language-injections-with-codeql.html"
---
Recently I wrote a post about [detecting JEXL injections with CodeQL](/en/security/detecting-jexl-injections-with-codeql.html). JEXL is a library that provides an interpreter for a simple expression language (EL). This time, I'll talk about injections with Jakarta Expression Language, and how they can be found with CodeQL.

![Detecting Jakarta Expression Language injections with CodeQL]({{ site.baseurl }}/assets/images/2021/04/cowsay_jakarta_el_and_codeql-1.png)

  
  


## What is Jakarta Expression Language?

Among other things, Jakarta EE contains a specification for an expression language (EL) and defines API for interpreters. The Jakarta EL is a special-purpose programming language that is mostly used in web applications for embedding and evaluating expressions in web pages. But the interpreter may be simply used anywhere else. There are multiple implementations of these API, for example:

- [JUEL](http://juel.sourceforge.net/)
- [Apache Commons EL](https://commons.apache.org/dormant/commons-el/)

The following example shows how to run an expression with JUEL:

 
<script src="https://gist.github.com/artem-smotrakov/0b2619b967c331d1135f791f3d701c87.js"></script>  


By the way, Jakarta EE used to be Java EE in the past. Originally, the API for the EL was located in the package `javax.el`. After Eclipse Foundation had taken over Java EE, the package was renamed to `jakarta.el`.

## What is Expression Language injection?

The EL is quite powerful. In particular, it allows invocation of methods available in the JVM. That makes it dangerous. If an expression is built using untrusted data, and then evaluated, it may allow the attacker to run arbitrary code in the worst case. That is called Expression Language Injection vulnerability.

The impact is usually high because the attacker can run almost any code inside or even outside the JVM. The likelihood of a successful attack depends on the way how the attacker can inject a malicious expression. In the worst case, an application receives an expression from an unauthenticated remote user. For example, it can read it from an HTTP request. That is likely to results in Remote Code Execution (RCE).

Here is an example of vulnerable code. It just takes an expression from an HTTP request and immediately runs it (again with JUEL):

 
<script src="https://gist.github.com/artem-smotrakov/1a170502a59cb229c3b64759d067b631.js"></script>  


## Preventing Jakarta EL injections

I'd rather avoid using Jakarta EL in applications if possible. If expressions are really necessary, only authorized users should be able to run them. It would be also good to evaluate expressions in a restrictive sandbox, but unfortunately, the Jakarta EL specification doesn't define any API for sandboxing. As a result, the interpreters don't implement any sandbox. Instead of Jakarta EL, you can consider using another EL that allows defining a sandbox. For example, Apache JEXL. [This post has a couple of examples that show how to implement a sandbox with JEXL.](/en/security/detecting-jexl-injections-with-codeql.html)

To prevent running arbitrary code, incoming data can be also checked before including it in an expression. The following example uses a Regex pattern to check whether a user tries to run an allowed expression or not:

 
<script src="https://gist.github.com/artem-smotrakov/e2a76babb90de87d3646b3a501521ed3.js"></script>  


## CodeQL query for detecting Jakarta EL injections

[CodeQL](https://securitylab.github.com/tools/codeql)&nbsp;is a code analysis engine. It lets you write queries for your code to detect various issues including security ones. Let???s see how it can help us detect Jakarta EL injections.

CodeQL can search for data flows from sources to sinks that you specify. In the case of Jakarta EL injections, the sinks are method invocations that evaluate expressions, for example, `ValueExpression.getValue()`, `MethodExpression.invoke()`, `ELProcessor.eval()` and others. The most interesting data sources are the ones that are available for remote users. For example, HTTP requests and network sockets. A data flow from a remote source to one of the Jakarta EL sinks may be a potential EL injection. I???ve implemented this idea in [this CodeQL&nbsp;query](https://github.com/github/codeql/pull/5471). Let???s see how it works.

First, there is a config for tracking tainted data from remote data sources to the Jakarta EL sinks:

 
<script src="https://gist.github.com/artem-smotrakov/cab3d5bbe1c47bc61c4248ab652c44b6.js"></script>  


It has three main components:

- `isSource()` predicate defines data sources. It uses the class `RemoteFlowSource` that describes network sockets, HTTP requests and other remote data sources which the CodeQL core library is aware of.
- `isSink()` predicate defines data sinks. It uses the class `ExpressionEvaluationSink` that lists method calls that evaluate expressions.
- `isAdditionalTaintStep()` predicate tells the CodeQL engine about additional ways how tainted input can be propagated.

The most interesting thing here is `ExpressionEvaluationSink`. Let's have a look at it:

 
<script src="https://gist.github.com/artem-smotrakov/414586d351a3479bf5c862cc566ce815.js"></script>  


First, the class describes methods that evaluate expressions immediately:

- `getValue()` and `setValue()` methods in `ValueExpression`
- `invoke()` methods in `MethodExpression` and `LambdaExpression`
- `eval()`, `getValue()` and `setValue()` methods in `ELProcessor`

Next, the class mentions the method `ELProcessor.setVarialbe()` that doesn't evaluate an expression right away but assigns the expression to a variable. The query assumes that the injected code is likely to be run a bit later when the variable is used.

The predicate&nbsp;`isAdditionalTaintStep()`&nbsp;defines two additional ways for propagating tainted data:

- `TaintPropagatingCall`&nbsp;defines methods calls that create `ValueExpression`, `MethodExpression` and `LambdaExpression` objects.
- `hasGetterFlow()`&nbsp;predicate defines calls to getters on objects that may hold tainted data. This predicate helps to identify issues when data is stored in complex objects. However, it doesn???t take into account how data flows inside those objects. As a result, it may potentially result in false positives.

The query detects a known [RCE in OpenFaces](https://github.com/TeamDev-Archive/OpenFaces/issues/175).

UPDATE: [T](https://github.com/github/codeql/pull/5471)[h](https://github.com/github/codeql/pull/5471)[e query](https://github.com/github/codeql/pull/5471) has been added to the set of [experimental](https://github.com/github/codeql/blob/main/java/ql/src/experimental/Security/CWE/CWE-094/JakartaExpressionInjection.ql) ones.

## References

- [Jakarta Expression Language](https://projects.eclipse.org/projects/ee4j.el)
- [Jakarta Expression Language API](https://javadoc.io/doc/jakarta.el/jakarta.el-api/latest/index.html)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)

