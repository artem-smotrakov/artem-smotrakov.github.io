---
layout: post
title: New Switch Expressions in Java 14
date: 2020-03-07 22:35:55.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tech
tags:
- Java
meta:
  _edit_last: '1'
  _yoast_wpseo_primary_category: '258'
  _yoast_wpseo_focuskw: switch expressions
  _yoast_wpseo_linkdex: '73'
  _yoast_wpseo_content_score: '60'
  _yoast_wpseo_metadesc: 'The new version of Java contains one major update to the
    Java language: new switch expressions. Let''s see how the new switch expressions
    can be used, what kind of advantages they offer, and what can go wrong.'
  rp4wp_auto_linked: '1'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617984850'

permalink: "/en/tech/new-switch-expressions-in-java-14.html"
---
<!-- wp:paragraph -->

Java 14 is going to be released on March 17, 2020. The new version of Java contains one major update to the Java language: new switch expressions. Let's see how the new switch expressions can be used, what kind of advantages they offer, and what can potentially go wrong. In the end, you are going to find a tricky question about the switch expressions.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

(the article has been published on [Medium](https://medium.com/better-programming/a-look-at-the-new-switch-expressions-in-java-14-ed209c802ba0))

<!-- /wp:paragraph -->

<!-- wp:image {"id":3836,"sizeSlug":"large","className":"noborder"} -->

![New Switch Expressions in Java 14]({{ site.baseurl }}/assets/images/2020/03/cowsays-New-switch-expressions-in-Java-14.jpg)

<!-- /wp:image -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## The classic switch statement

<!-- /wp:heading -->

<!-- wp:paragraph -->

The current design of the `switch` statement in Java follows languages such as C and C++. It works only as a statement and supports fall through semantics by default. Here is an example of the classic `switch` statement with an enum:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/d0ba379fa43f132e4c0fede6b51cf1ad.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

You might have noticed many `case` and `break` statements in the example above. Those statements introduce some visual noise and make the code unnecessarily verbose. This visual noise may then mask mistakes such a missing `break` statement which would mean accidental fall through.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## The new switch expressions

<!-- /wp:heading -->

<!-- wp:paragraph -->

Java 14 extends `switch` so that it can be used as either a statement or an expression. In particular, the new Java introduces the following:

<!-- /wp:paragraph -->

<!-- wp:list -->

- A new form of switch label `case ... ->` where only the code to the right of the label is going to be executed if the label is matched. The code to the right of a `case ... ->` label may be an expression, a block, or a throw statement.
- A new `yield` statement to yield a value which becomes the value of the enclosing switch expression.
- Multiple constants per case which are separated by commas.

<!-- /wp:list -->

<!-- wp:paragraph -->

With the new Java 14, it's possible to use both traditional `case … :` labels and new `case … ->` labels. It is important that the traditional labels still support fall through by default, but the new ones don't.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Let's see how the example above can be re-written with the new switch expressions:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/cd1dcb699e93f146b7578b09b229aad3.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

This simple example can be compiled and run with just a single command (thanks to JEP 330 which allowed launching single-file source-code programs since Java 11):

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
$ java WhoIsWho.java
Mozart was a composer
Dali was a painter
Dostoevsky was a writer
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

You can see that several `case` statements have been merged, and the code doesn't use `break` statement any more. As a result, the `print()` method became much shorter and looks nicer. The following example shows a multiline `default` block which uses the new `yield` statement to yield a value. Note several new constants in the `Person` enum which are not covered by any `case` label in the switch expression:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/46b60dc054c511160646e8415114ec8b.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

Here is what it prints out:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
$ java WhoIsWho.java
Mozart was a composer
Dali was a painter
Oops! I don't know about Einstein
Einstein was a …
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

The next example shows how factorial may be implemented with the new switch expressions:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/e5578afa01ff85b266e31b6952151ed1.js"></script>  
<!-- /wp:html -->

<!-- wp:heading -->

## Important details

<!-- /wp:heading -->

<!-- wp:paragraph -->

There are several important things to know about the new switch expressions.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The first thing to know is that the cases of a switch expression must be exhaustive. In other words, for all possible values, there must be a matching switch label. Let's just add a new element to the enum and see what's going to happen:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/ca95ffad3301e3279464dbc9e81369b8.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

Compilation will fail right away with the following error message:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
$ java InvalidWhoIsWho.java
InvalidWhoIsWho.java:19: error: the switch expression does not cover all possible input values
         String title = switch (person) {
                        ^
1 error
error: compilation failed
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Adding a simple `default` case makes the Java compiler happy:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/3bbbe7ed7f16e4134817ad024defd4be.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

In general, unless an enum is used and the cases of a switch expression cover all constants, a `default` clause is required in the switch expression.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The second thing to remember is that a switch expression must either complete normally with a value or by throwing an exception. Let's take a look at the following code:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/be2a7fad8484bd6ab53b9c65fe018454.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

If we try to compile this code, the Java compiler will immediately complain:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
InvalidSwitchExpressionWithoutDefault.java:8: error: the switch expression does not cover all possible input values
        return switch (n) {
               ^
1 error
error: compilation failed
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Again, adding a `default` case makes it work:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/013a55bb127d81303dc6ab5a7f1cf430.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

The third important thing to keep in mind is that `yield` is now a restricted identifier. In particular, it means that classes named "yield" become illegal:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
$ cat YieldClassName.java 
class yield {}
$ javac YieldClassName.java
YieldClassName.java:1: error: 'yield' not allowed here
class yield {
      ^
  as of release 13, 'yield' is a restricted type name and cannot be used for type declarations
1 error
error: compilation failed
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

However, it is allowed to use "yield" as a variable or a method name:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
$ cat ValidUseOfYieldWord.java 
public class ValidUseOfYieldWord {
    void yield() {
        int yield = 0;
    }
}
$ javac ValidUseOfYieldWord.java && echo ok || echo failed
ok
```

<!-- /wp:preformatted -->

<!-- wp:heading -->

## Conclusion

<!-- /wp:heading -->

<!-- wp:paragraph -->

Here is what the authors say about the new switch expressions:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> These changes will simplify everyday coding

<!-- /wp:quote -->

<!-- wp:paragraph -->

Let's see.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Bonus

<!-- /wp:heading -->

<!-- wp:paragraph -->

What do you think is going to happen? Here are the options:

<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->

1. Compilation error.
2. Runtime error.
3. "Oops" is printed out.
4. "OopsOops" is printed out
5. "Forty-two" is printed out.

<!-- /wp:list -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/eb018ebf2f321753eb71e5383bde5c7c.js"></script>  
<!-- /wp:html -->

<!-- wp:heading -->

## References

<!-- /wp:heading -->

<!-- wp:list -->

- [JEP 361: Switch Expressions](https://openjdk.java.net/jeps/361)
- [OpenJDK 14 schedule and the list of enhancements](https://openjdk.java.net/projects/jdk/14/)

<!-- /wp:list -->

<!-- wp:paragraph -->

<!-- /wp:paragraph -->

