---
layout: post
title: Configuring security for REST API in Spring
date: 2018-10-10 15:56:13.000000000 +01:00
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
- Spring
- Web security
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '90'
  _yoast_wpseo_primary_category: '157'
  _oembed_2567fe47d343837597b8185dee4dc49f: "{{unknown}}"
  _oembed_171f377d9f3ebcec8ce0fc188f466c29: "{{unknown}}"
  _oembed_b82d7266fb36916f2a7b14d79e2f286e: "{{unknown}}"
  _oembed_46d4640fb489da6dbf236457d32af9ba: "{{unknown}}"
  _yoast_wpseo_metadesc: This post contains a list of things which may be good to
    pay attention to when you configure or review authentication and authorization
    settings for a RESTful application based on Spring.
  rp4wp_auto_linked: '1'
  _yoast_wpseo_focuskw: Spring
  _yoast_wpseo_linkdex: '67'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617982976'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/security/tips-configuring-security-rest-api-spring.html"
---
In most cases, REST APIs should be accessed only by authorized parties. Spring framework provides many ways to configure authentication and authorization for an application. Another good thing is that the framework usually provides relatively good default settings. But nevertheless, it may be better to understand what's going on rather then rely on the defaults.

This post contains a list of things which may be good to pay attention to when you configure or review authentication and authorization settings for a RESTful application based on Spring (boot) framework. However this is not a comprehensive guideline (if such a guideline even exist) which tells how to configure authentication and authorization for an application based on Spring framework. It's more like a collection of tips and suggestions. Furthermore, any other suggestions and comments are more than welcome.

<!--more-->

## Consider using OAuth2 or JWT

Usually only authorized users or applications should be able to access a RESTful service. A RESTful web application doesn't normally use HTTP sessions. Instead, it performs access checks for each incoming request.

One of the ways to restrict access to a RESTful application is to start using API keys. But OAuth2 or JWT (JSON Web tokens) may sound like a better option. OAuth2 and JWT are frameworks for&nbsp; authentication and authorization which are described in RFCs. The frameworks allow configuring and controlling authentication and authorization processes in a more flexible way than API keys do. But the cost for that is an additional infrastructure and complexity. Fortunately, there are already implementations of these standards, and even services which provide all required infrastructure (for example, [application load balancers in AWS support OAuth2](https://aws.amazon.com/blogs/aws/built-in-authentication-in-alb/)).

Even if an application uses OAuth2 or JWT, we need to make sure that it uses it correctly, for example:

- the application validates tokens and access rights for each request
- the application doesn't expose access tokens (for example, no tokens should be put to logs)
- access tokens have a correct lifetime
- and so on

By the way, [OWASP provides good guidelines for authentication and authorization](https://www.owasp.org/index.php/Top_10-2017_A2-Broken_Authentication).

## Enforce HTTPS

This advice can be found in every guideline/article/book/presentation about securing a web application. Using TLS is important especially if an application uses OAuth2 (or API keys) for authentication since the protocol explicitly relies on TLS for providing confidentiality and integrity.

In Spring, you can use&nbsp;`requiresSecure()`&nbsp;method. Don't forget to enable [HSTS header](https://docs.spring.io/spring-security/site/docs/4.2.5.RELEASE/apidocs/org/springframework/security/config/annotation/web/configurers/HeadersConfigurer.html#httpStrictTransportSecurity--). Here is an example:

<script src="https://gist.github.com/artem-smotrakov/1fafe7d74b93f10c99591e19248204c2.js"></script>

An application may support only plain HTTP and stay behind a reverse proxy which provides TLS. In this case, make sure that the reverse proxy is configured to use HTTPS and HSTS header. The proxy should deny plain HTTP connections, or always redirect to HTTPS.

## Configure session management

Spring framework provides a mechanism to establish an HTTP session. If a RESTful service authenticates each request (which mostly is the case), then it doesn't need any session management mechanism. In this case, it may be better to instruct Spring framework not to create any session:

<script src="https://gist.github.com/artem-smotrakov/bed11936a6632092a079149392246fc8.js"></script>

## Disable login and logout pages

Spring framework provides a login form and a logout page out of the box. This may be useful for a web application with a GUI, but most probably a RESTful application doesn't need these pages. This is the case for sure if the application uses OAuth2, JWT or API tokens for access control. Then, it might make sense to disable these features for the application:

<script src="https://gist.github.com/artem-smotrakov/3b0a1a1381c5486439d4336e6d2ecefa.js"></script>

## Disable Basic HTTP authentication

Most probably a RESTful application doesn't need HTTP Basic authentication, so it may be better to disable it:

<script src="https://gist.github.com/artem-smotrakov/b8c601db7cc389e3b147e3aa89c0d775.js"></script>

## Disable anonymous access

If a RESTful service should be accessed only by authorized clients, then it may be better to disable anonymous access:

<script src="https://gist.github.com/artem-smotrakov/1436cc0997bfb294427983fda9990595.js"></script>

## Authorizing requests by HTTP methods may lead to a problem

`antMatchers()` method allows to specify an HTTP method for which you'd like to configure access. It may look like the following:

<script src="https://gist.github.com/artem-smotrakov/6ac47bbddffe30e63c01e0863c4d1f04.js"></script>

Depending on the rest of the security configuration, it may result to a problem because we missed HEAD method in the configuration above. To avoid such issues, it may be better to specify API endpoints in `antMatchers()`, require all requests to be authenticated, and call `denyAll()` in the end. `authenticated()` and `fullyAuthenticated()` methods may be used to require all requests to be authenticated. The difference is that the last one doesn't take into account remember-me feature. By the way, most likely a RESTful service doesn't need remember-me. Here is what security config may look like:

<script src="https://gist.github.com/artem-smotrakov/97afa6d0d0797a00891fdcbbe1e41d61.js"></script>

## Disable creating a default user with random password

By default, Spring framework creates a default user `user`&nbsp;with a random password. Then, the framework prints out the password to logs. It looks like the following:

```
Using default security password: 94251362-f9cc-4c01-95be-121e5b6e1865
```

Most probably an application doesn't need this used in production especially if it uses OAuth2 or JWT. So, it may be better to disable it even if it doesn't hurt. Overriding `authenticationManagerBean()` method in security configuration helps although this solution may look a bit strange (you can find a bit more details [here](https://stackoverflow.com/a/41856630)):

<script src="https://gist.github.com/artem-smotrakov/f2e7951e530806baf186fbb0e28c0948.js"></script>

## References

- [Securing a Web Application](https://spring.io/guides/gs/securing-web/)
- [Spring Security Architecture](https://spring.io/guides/topicals/spring-security-architecture/)
- [REST Security Cheat Sheet](https://www.owasp.org/index.php/REST_Security_Cheat_Sheet)
