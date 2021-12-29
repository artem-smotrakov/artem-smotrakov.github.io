---
layout: post
title: Safer deserialization in Spring Security OAuth2
date: 2019-11-16 19:32:24.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Deserialization
- Java
- Security
- Spring
meta:
  _edit_last: '1'
  _yoast_wpseo_primary_category: '157'
  _yoast_wpseo_content_score: '60'
  _yoast_wpseo_focuskw: deserialization
  _yoast_wpseo_metadesc: By default, Spring Security OAuth2 uses deserialization in
    an unsafe way. Let's discuss how to mitigate vulnerabilities if an application
    uses the library.
  _yoast_wpseo_linkdex: '68'
  rp4wp_auto_linked: '1'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617978907'

permalink: "/en/security/safer-deserialization-in-spring-security-oauth.html"
---
<!-- wp:paragraph -->

The Java standard library provides the `ObjectInputStream` class which offers a convenient way for deserializing Java objects. Unfortunately, this way is not safe by default. Using this class may open the doors for Java deserialization attacks which in the worse case may result in arbitrary code execution.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

I recently discovered that Spring Security OAuth2 library may be vulnerable to such an attack. Fortunately, there is one strong pre-requisite for a successful attack which may be difficult to meet for an adversary. Nevertheless, I thought it might be better to make the library a bit safer, and the project maintainers kindly accepted the contribution. Here are the details.

<!-- /wp:paragraph -->

<!-- wp:image {"id":3656,"sizeSlug":"large","className":"noborder"} -->

![Safer deserialization in Spring Security OAuth 2.4.0]({{ site.baseurl }}/assets/images/2019/11/cowsay_happy_deserialization.jpg)

<!-- /wp:image -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## The issue

<!-- /wp:heading -->

<!-- wp:paragraph -->

Spring Security OAuth2 can store authentication info and user details to a SQL or Redis database. Before storing data to the database, the library serialize it with the default Java serialization mechanism offered by the`ObjectOutputStream`. Then, after reading the data from the database, the library unsafely deserializes it with the `ObjectInputStream` class. See `SerializationUtils` class:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
public static T deserialize(byte[] byteArray) {
         ObjectInputStream oip = null;
         try {
             oip = new ConfigurableObjectInputStream(
                     new ByteArrayInputStream(byteArray),
                     Thread.currentThread().getContextClassLoader());
             @SuppressWarnings("unchecked")
             T result = (T) oip.readObject();
```

<!-- /wp:code -->

<!-- wp:paragraph -->

The `ConfigurableObjectInputStream` class, which is provided by the Spring Framework, just wraps the `InputObjectStream` class without adding any security check.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

It means that if an attacker is able to put malicious data into the database, then he can in the worse case execute arbitrary code, and as a result, compromise the whole application. However, there are a couple of requirements for a successful exploit.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

First, the attacker has to build a deserialization gadget using classes that are available in the application's JVM. Most probably it should not be a big problem since the Java standard library provides many dangerous classes. Plus, an application can load many libraries which may also help to build a deserialization gadget.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Second, to take advantage of this deserialization flaw, the attacker has to find a way how he can put malicious data into the database. For example, he can try to find an SQL injection. However, it may be not that easy, so that this requirement may be difficult to meet.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

I found the problem by reviewing the code. Then, I reported it to the Pivotal Security Team. But they decided not to consider it as a vulnerability in the library because of the second requirement above. However, they welcomed a patch that adds a defense-in-depth measure to prevent such deserialization attacks.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## The solution

<!-- /wp:heading -->

<!-- wp:paragraph -->

To prevent Java deserialization vulnerabilities, an application has to restrict a set of classes which may be deserialized. One of the best ways is implementing a whitelist of allowed classes. There are three ways of how such a protection mechanism can be implemented:

<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->

1. Use the filtering API introduced in JEP 290.
2. Use the `ValidatingObjectInputStream` class from Apache Commons IO.
3. Override the `ObjectInputStream.resolveClass()` method, and implement the whitelisting there.

<!-- /wp:list -->

<!-- wp:paragraph -->

The first way would require an application to use the Java versions which have JEP 290. The second way would add an additional dependency to Spring Security OAuth. I decided to follow the third way since it doesn't introduce any additional dependency:

<!-- /wp:paragraph -->

<!-- wp:list -->

- Added a new `SaferObjectInputStream` class which checks if classes are allowed for deserialization.
- Defined a whitelist of classes as `java.lang.*`, `java.util.*` and `org.springframework.security.*`.
- [Updated](https://github.com/spring-projects/spring-security-oauth/pull/1703) the `RedisTokenStore` class to use the `SaferObjectInputStream` with the whitelist.
- [Updated](https://github.com/spring-projects/spring-security-oauth/pull/1760) the JDBC classes to use the `SaferObjectInputStream` with the whitelist.

<!-- /wp:list -->

<!-- wp:paragraph -->

The update was released in Spring Security OAuth2 `2.3.7` but unfortunately, the solution caused a problem. It turned out that the whitelist is too strict. If an application stores custom implementations of tokens or user details, then the new version of the library fails to deserialize them due to the restrictive whitelist. Furthermore, there was no way how a user could modify the whitelist to make the application work again. The issue was [reported](https://github.com/spring-projects/spring-security-oauth/issues/1759#issuecomment-543076614) by several users.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

I fixed the problem in `2.4.0` by introducing a new API which allows a user to specify a custom whitelist. The default whitelist still contains `java.lang.*`, `java.util.*` and `org.springframework.security.*` classes. At first, I updated the library to apply the whitelist by default. But then, the project maintainer asked me to make it an opt-in option. The reason was that `2.4.0` is a minor release, so that it should not introduce any problem for an application when it migrates to the new version.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Here is what I did to [make deserialization great again](https://github.com/spring-projects/spring-security-oauth/pull/1784):

<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->

1. Added a new `SerializationStrategy` interface with two implementations: `DefaultSerializationStrategy` and `WhitelistedSerializationStrategy`.
2. The `DefaultSerializationStrategy` uses unsafe deserialization.
3. The `WhitelistedSerializationStrategy` allows specifying a whitelist of classes which are allowed for deserialization. If no classes specified, the strategy uses the default whitelist: `java.lang.*`, `java.util.*` and `org.springframework.security.*`.
4. Added a new static `SerializationStrategy` field to the `SerializationUtils` class. The default strategy is unsafe `DefaultSerializationStrategy`.
5. The strategy can be overridden by calling a new `SerializationUtils.setSerializationStrategy()` method, or by specifying the strategy in the `META-INF/spring.factories` file.

<!-- /wp:list -->

<!-- wp:paragraph -->

Now a user can enable the default whitelist with the following code:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
SerializationUtils.setSerializationStrategy(new WhitelistedSerializationStrategy());
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Or, the user can implement his own serialization strategy, for example, by extending the `WhitelistedSerializationStrategy` class:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
package org.custom.impl.oauth2;

public class CustomSerializationStrategy
    extends WhitelistedSerializationStrategy {

        private static final List<String> ALLOWED_CLASSES 
                = new ArrayList<String>();
        static {
            ALLOWED_CLASSES.add("java.lang.");
            ALLOWED_CLASSES.add("java.util.");
            ALLOWED_CLASSES.add("org.springframework.security.");
            ALLOWED_CLASSES.add("org.custom.impl.oauth2.");
        }

        CustomSerializationStrategy() {
            super(ALLOWED_CLASSES);
        }
    }
}
```

<!-- /wp:code -->

<!-- wp:paragraph -->

And here is how the user can specify the `WhitelistedSerializationStrategy` strategy in `META-INF/spring.factories` file:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
org.springframework.security.oauth2.common.util.SerializationStrategy = \
org.springframework.security.oauth2.common.util.WhitelistedSerializationStrategy
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Of course, `META-INF/spring.factories` allows using a custom serialization strategy as well.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Conclusion

<!-- /wp:heading -->

<!-- wp:paragraph -->

By default, Spring Security OAuth2 library still uses deserialization in an unsafe way which may make an application vulnerable. Although the discussed deserialization flaw may be difficult to exploit, it may be better to enable the `WhitelistedSerializationStrategy` to be on the safe side.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## References

<!-- /wp:heading -->

<!-- wp:list {"ordered":true} -->

1. [Spring Security OAuth](https://github.com/spring-projects/spring-security-oauth)
2. [ObjectInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/ObjectInputStream.html)
3. [JEP 290](https://openjdk.java.net/jeps/290)
4. [ValidatingObjectInputStream](https://commons.apache.org/proper/commons-io/javadocs/api-2.5/org/apache/commons/io/serialization/ValidatingObjectInputStream.html)

<!-- /wp:list -->

