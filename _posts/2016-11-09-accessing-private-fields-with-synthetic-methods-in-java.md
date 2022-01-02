---
layout: post
title: Accessing private fields with synthetic methods in Java
date: 2016-11-09 22:23:17.000000000 +00:00
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
permalink: "/en/security/accessing-private-fields-with-synthetic-methods-in-java.html"
---
In Java, you can&nbsp;define one class B inside another class A. Class B is called an inner class, and class A&nbsp;is called an outer class. It&nbsp;looks like the following:

<script src="https://gist.github.com/artem-smotrakov/717ff420953f6a112fe1900ef156a2aa.js"></script>

Class A has a private field "secret". This private field can be accessed by both A and B classes. But in some cases, this private field can be accessed by other classes in the same package&nbsp;even if neither A or B provide any accessors. It actually depends on what we have in go() method.

# How does Java compiler compile inner classes?

Let's take a look at the following example:

<script src="https://gist.github.com/artem-smotrakov/b520bea1b807529edf1d6c4948025e85.js"></script>

There are a couple of interesting things about this code.

First, Java compiler compiles&nbsp;these two classes above to&nbsp;two separate classes in the same package. If you ask Java compiler&nbsp;to compile this code, it will create two class files "Outer.class" and "Outer$Inner.class". At runtime, those classes will be considered as separate classes in the same package.

But we know that a private field can be only accessed by a class which owns this field. In our case, Outer class has a private field "secret" which is accessed by Outer$Inner class. When JRE runs this code, Outer$Inner class is going to be considered as a separate class which means that Outer$Inner class is not allowed to access "secret" anymore. But this code works. So&nbsp;the question is how Outer$Inner class can access "secret" field.

This is second interesting thing about this code. Java compiler creates a synthetic method which can be called by Outer$Inner class to modify "secret" field. You can see this synthetic method if you run "javap".&nbsp;You can find full example on GitHub:

[https://github.com/artem-smotrakov/javahell](https://github.com/artem-smotrakov/javahell)

Here is an example how you can run javac and javap:

```
$ git clone https://github.com/artem-smotrakov/javahell
$ cd javahell
$ mkdir -p classes
$ javac -d classes src/com/gypsyengineer/innerclass/field/*.java
$ javap -c -p classes/com/gypsyengineer/innerclass/field/Outer.class
```

It's going to print something like this:

```
Compiled from "Outer.java"
public class com.gypsyengineer.innerclass.field.Outer {
  private int secret;

  public com.gypsyengineer.innerclass.field.Outer();
    Code:
       0: aload_0
       1: invokespecial #2 // Method java/lang/Object."&lt;init&gt;":()V
       4: aload_0
       5: bipush 10
       7: putfield #1 // Field secret:I
      10: return

  public void check();
    Code:
       0: aload_0
       1: getfield #1 // Field secret:I
       4: ifge 18
       7: getstatic #3 // Field java/lang/System.out:Ljava/io/PrintStream;
      10: ldc #4 // String Oops
      12: invokevirtual #5 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      15: goto 26
      18: getstatic #3 // Field java/lang/System.out:Ljava/io/PrintStream;
      21: ldc #6 // String Okay
      23: invokevirtual #5 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      26: return

  static int access$002(com.gypsyengineer.innerclass.field.Outer, int);
    Code:
       0: aload_0
       1: iload_1
       2: dup_x1
       3: putfield #1 // Field secret:I
       6: ireturn
}
```

You can see that&nbsp;Java compiler added a new "access$002" method to Outer.class. This is a synthetic method which assigns a value to integer "secret" field. Here is what "javap" says about Outer$Inner class:

```
Compiled from "Outer.java"
public class com.gypsyengineer.innerclass.field.Outer$Inner {
  final com.gypsyengineer.innerclass.field.Outer this$0;

  public com.gypsyengineer.innerclass.field.Outer$Inner(com.gypsyengineer.innerclass.field.Outer);
    Code:
       0: aload_0
       1: aload_1
       2: putfield #1 // Field this$0:Lcom/gypsyengineer/innerclass/field/Outer;
       5: aload_0
       6: invokespecial #2 // Method java/lang/Object."&lt;init&gt;":()V
       9: return

  public void go();
    Code:
       0: aload_0
       1: getfield #1 // Field this$0:Lcom/gypsyengineer/innerclass/field/Outer;
       4: bipush 42
       6: invokestatic #3 // Method com/gypsyengineer/innerclass/field/Outer.access$002:(Lcom/gypsyengineer/innerclass/field/Outer;I)I
       9: pop
      10: return
}
```

You can see that go() method calls "access$002" in order to modify "secret".

Third interesting thing about this code is that static "access$002" method is accessible by any class in the same package. This means that adding an inner class sometimes may implicitly make a private field accessible by other classes in the same package. But the key word here is "may". Just adding an inner class doesn't make Java compiler create a synthetic method to access a&nbsp;private field. But if an&nbsp;inner class needs to access private fields, then Java compiler will add synthetic methods. Java compiler may add different synthetic methods which actually depends on what an inner class does with private fields. For example, try to replace "secret = 42;" with "secret++;" and run "javap" on&nbsp;compiled classes.

# How can we call synthetic methods?

Java compiler fails if you try to call a synthetic method in your Java code:

```java
Outer.access$002(outer, 0);
```

But a synthetic method can be called with Reflection API:

```java
Outer o = new Outer();
Method m = o.getClass().getDeclaredMethod("access$002", o.getClass(), int.class);
m.invoke(null, o, -1);
```

If you feel comfortable with writing bytecode manually (or, using some tools), then you can just create bytecode which calls synthetic methods.

But success of invocation of a synthetic method also depends on a class loader.

# Accessing synthetic methods with the same class loader

The following code works fine if all classes&nbsp;were loaded with the same class loader:

<script src="https://gist.github.com/artem-smotrakov/61458c68d38e6b890a55ad74947fcb4e.js"></script>

At runtime,&nbsp;both&nbsp;AccessPrivateField and Outer classes are in the same package because they were loaded by the same class loader. As a result, AccessPrivateField can call access$002() method of Outer class. But it behaves differently&nbsp;if we use different classloaders.

# Accessing synthetic methods with different classloaders

The following code throws an exception if we load another AccessPrivateField class again with different classloader:

<script src="https://gist.github.com/artem-smotrakov/bf05dbb8cdf012db5c979f18e7f8453e.js"></script>

Let's assume that Outer class was loaded by classloader #1. First, the code above loads&nbsp;AccessPrivateField (\*) class with different classloader #2. Next, it creates an instance of Outer class which was loaded by classloader #1. Then, it invokes go() method on class&nbsp;AccessPrivateField (\*) which was loaded by classloader #2. Method go() gets a handle of&nbsp;access$002() method of class Outer with Reflection API, and tries to call it. This call fails because at runtime classes&nbsp;AccessPrivateField (\*) and Outer are not in the same package since they were loaded by different classloaders. As a result,&nbsp;IllegalAccessException exception occurs while invoking&nbsp;m.invoke(null, t, -1)

```
Test #2: try to modify a private field with different classloader
         (an exception is expected)
Exception in thread "main" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.gypsyengineer.innerclass.field.AccessPrivateField.test02(AccessPrivateField.java:83)
	at com.gypsyengineer.innerclass.field.AccessPrivateField.main(AccessPrivateField.java:54)
Caused by: java.lang.IllegalAccessException: Class com.gypsyengineer.innerclass.field.AccessPrivateField can not access a member of class com.gypsyengineer.innerclass.field.Outer with modifiers "static"
	at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:102)
	at java.lang.reflect.AccessibleObject.slowCheckMemberAccess(AccessibleObject.java:296)
	at java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:288)
	at java.lang.reflect.Method.invoke(Method.java:491)
	at com.gypsyengineer.innerclass.field.AccessPrivateField.go(AccessPrivateField.java:90)
	... 6 more
```

# Conclusion

It may be better to keep in mind this little "feature" of Java compiler which may allow other classes to access private fields. It also may be better to avoid usage of inner classes. I cannot say that I write lots of Java code everyday, but I cannot remember when I needed an inner class last time. Inner static classes sometimes may be helpful, but they don't cause creating synthetic methods since they are not suppose to have access to members of outer class.

