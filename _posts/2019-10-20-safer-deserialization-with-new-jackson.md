---
layout: post
title: Safer deserialization with new Jackson 2.10
date: 2019-10-20 17:00:44.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Deserialization
- Jackson
- Java
- Open Source
- Security
permalink: "/en/security/safer-deserialization-with-new-jackson.html"
---
New Jackson 2.10 was released on Sep 26th, 2019. Everyone who uses the library and also scans their applications for known vulnerabilities knows about the problem with endless CVEs that have been reporting against Jackson. Let's try to understand what makes an application vulnerable and how the new version of Jackson can help to prevent deserialization vulnerabilities.

![Safer deserialization with new Jackson 2.10]({{ site.baseurl }}/assets/images/2019/10/cowsay-deserialization-is-now-safer-with-new-Jackson-1.jpg)

  
  


## What makes an application vulnerable

A new CVE pops up each time when somebody discovers a new dangerous deserialization gadget. A deserialization gadget is a class that allows executing dangerous code during deserialization. Such a class usually comes from a popular Java library which is likely to be included in the classpath.

It's important to mention that not each and every application which uses Jackson is vulnerable to deserialization attacks. Here is a list of conditions which have to be met to make an attack possible:

1. The application deserializes data that may come from an attacker.
2. The application uses one of the libraries which contains dangerous gadgets.
3. The application enables polymorphic type handling.
4. The application uses fields of generic types such as `Object`, `Serializable`, `Cloneable` and so on.

In other words, if at least one of the conditions above is not met, the application is not vulnerable. Let's take a closed look at each condition.

First, the attacker should be able to feed malicious data to the application, so that the data reaches the deserialization mechanism. Unfortunately, it may be very difficult to identify all possible ways of how malicious data can come to the deserialization procedures. Even if some of the ways are known, then it may be difficult to mitigate it. For example, if a public RESTful application receives JSON via a POST request, and then deserializes it, it may be hard to do anything here besides protecting the endpoint with authentication. But it may also happen that the adversary bypasses the authentication mechanism, for example, due to other vulnerabilities. As a result, in many cases, there is a non-zero likelihood that condition #1 is met.

Second, the application should use one of the libraries which contain deserialization gadgets. Nowadays, a typical application uses a lot of third-party dependencies which bring a lot of classes. The good news here is that not all of the classes are dangerous. Some of them may be already known gadgets which are blocked by Jackson. However, the bad news is that there may be many unknown deserialization gadgets. As a result, in many cases, there is a non-zero likelihood that condition #2 is met.

Third, the application enables polymorphic type handling. This feature supports polymorphism which is brought in by object-oriented programming. Making a long story shorter, the feature allows deserializing multiple implementations of a base class. To make it possible, a serialized object should contain a class name which defines a specific implementation of the base class, for example:

```json
{
    "name": "John Doe",
    "age": 101,
    "phone": {
        "@class": "com.phone.DomesticNumber",
        "areaCode": 0,
        "local": 0
    }
}
```

Here you can see a `@class` field which contains a class name. There are two ways to enable polymorphic type handling:

1. Call one of the `ObjectMapper.enableDefaultTyping()` methods.
2. Use `@JsonTypeInfo(use = Id.CLASS)` annotation.

An application is not vulnerable if it does not enable polymorphic type handling with one of the ways above.

Fourth, the application should use fields of generic types such as `Object`, `Serializable`, `Cloneable` and so on. It's required because during deserialization Jackson checks that a deserialized type can be assigned to a specific field. Here is an example of vulnerable code:

 
<script src="https://gist.github.com/artem-smotrakov/a80ecd6336e4e5c730c4a0ff2fd72500.js"></script>  


The `phone` field is `Object` which means that any object may be assigned to this field during deserialization. If the type of the `phone` field were some custom class `PhoneNumber`, most likely the code would not be vulnerable anymore.

## Examples of vulnerable code

Here you can find a couple of examples of vulnerable code:

1. [In the first example](https://github.com/artem-smotrakov/javahell/tree/unsafe-jackson-example-v2/src/main/java/com/gypsyengineer/jackson/unsafe/one), polymorphic type handling is enabled by using `@JsonTypeInfo(use = Id.CLASS)` annotation in `Person` class. `UnsafePersonDeserialization` class demonstrates unsafe deserialization.
2. [In the second example](https://github.com/artem-smotrakov/javahell/tree/unsafe-jackson-example-v2/src/main/java/com/gypsyengineer/jackson/unsafe/two), polymorphic type handling is enabled by calling `ObjectMapper.enableDefaultTyping()` method. `UnsafeCatDeserialization` class demonstrates unsafe deserialization.

In both examples, [the role of a gadget](https://github.com/artem-smotrakov/javahell/blob/unsafe-jackson-example-v2/src/main/java/com/popular/lib/Exec.java) plays `com.popular.lib.Exec` class which runs a specified command in `setCommand()` method using `Runtime.exec()`.

## How to protect an application

To protect an application, at least one of the conditions mentioned above have to be broken. Unlike the conditions #1 and #2, the conditions #3 and #4 are much easier to avoid in an application. In other words, if an application doesn't enable polymorphic type handling, or at least restrict it by avoiding generic field types, then most likely the application is not vulnerable to deserialization attacks via Jackson.

Jackson also tries to break the condition #2. To block dangerous gadgets, Jackson Databind contains [a list of classes which are not allowed to be deserialized](https://github.com/FasterXML/jackson-databind/blob/ae15ae4492de360168b12aaab55248061d2077bd/src/main/java/com/fasterxml/jackson/databind/jsontype/impl/SubTypeValidator.java#L35). It's usually called a blacklist approach. When a new gadget is discovered, it needs to be added to the blacklist. Then, a new version of Jackson goes out. Since new gadgets keep coming, this approach results in a situation when applications need to update Jackson Databind every month or sometimes even more often.

The new Jackson 2.10 now allows implementing security checks which are specific to a particular application. The following new APIs can be used to implement such security checks:

1. Abstract `PolymorphicTypeValidator` class which can be extended to implement validating classes during deserialization.
2. `BasicPolymorphicTypeValidator` class which extends `PolymorphicTypeValidator` and implements both whitelist and blacklist for validating classes.
3. Added new methods such as `activateDefaultTyping(PolymorphicTypeValidator, ???)` and `polymorphicTypeValidator(...)` which allow specifying validators for classes during deserialization.

Note that unsafe `ObjectMapper.enableDefaultTyping()` methods are now marked as deprecated.

Here is an example of a whitelist implemented with the new API:

 
<script src="https://gist.github.com/artem-smotrakov/eed1df9759e0f9873d4c90ece7149884.js"></script>  


In the code snippet above, an `ObjectMapper` is configured with a validator which allows deserialization only safe classes from `com.gypsyengineer.jackson` package. Note that the new Jackson 2.10 offers multiple ways to define a validation procedure for deserialization.

Full code can be found [here](https://github.com/artem-smotrakov/javahell/tree/unsafe-jackson-example-v2/src/main/java/com/gypsyengineer/jackson/unsafe/one), and here is another [example](https://github.com/artem-smotrakov/javahell/blob/unsafe-jackson-example-v2/src/main/java/com/gypsyengineer/jackson/unsafe/two/SaferCatDeserialization.java).

## Conclusion

The good news is that Jackson 2.10 provides a convenient way for an application to prevent deserialization vulnerabilities. However, there is a couple of not too good news.

First, it's not enough just to increment Jackson version to 2.10. A security check for the deserialization process has to be carefully implemented using the new APIs.

Second, even if an application migrates to the new Jackson 2.10, and then implements a correct security check for its own deserialization procedures, there may be other third-party dependencies that still use Jackson in an unsafe way. If an attacker can reach these unprotected deserialization procedures brought in by third-party dependencies, then the application still may be vulnerable.

It looks like we can say that an application is not vulnerable to deserialization attacks only when all its dependencies migrate to 2.10 and implement security checks for their deserialization procedures.

## References 

1. [Jackson Release 2.10](https://github.com/FasterXML/jackson/wiki/Jackson-Release-2.10)
2. [Jackson 2.10 features](https://medium.com/@cowtowncoder/jackson-2-10-features-cd880674d8a2)
3. [On Jackson CVEs: Don???t Panic ??? Here is what you need to know](https://medium.com/@cowtowncoder/on-jackson-cves-dont-panic-here-is-what-you-need-to-know-54cd0d6e8062)

