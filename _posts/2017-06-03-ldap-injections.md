---
layout: post
title: LDAP injections
date: 2017-06-03 22:30:57.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- LDAP
- Python
- Security
meta:
  _edit_last: '1'
  mask_links: default
  _wpgo_column_layout_save: ''
  _wpgo_column_layout: default
  _syntaxhighlighter_encoded: '1'
  _aioseop_opengraph_settings: a:14:{s:32:"aioseop_opengraph_settings_title";s:0:"";s:31:"aioseop_opengraph_settings_desc";s:0:"";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}
  _aioseop_keywords: LDAP, injection, attack, blind, security, java, python, exploit,
    example,
  _aioseop_description: 'LDAP injections: what is LDAP injection, example of vulnerable
    application, example of an exploit for blind LDAP injection, how to prevent LDAP
    injections.'
  _aioseop_title: LDAP injections
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: ''
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_focuskw_text_input: LDAP injections
  _yoast_wpseo_focuskw: LDAP injections
  _yoast_wpseo_metadesc: Everybody knows about SQL injections. It's like a celebrity
    in the world of software security. But there are much more many different types
    of injection attacks which may feel jealous about popularity of SQL injections.
    That's not fair. Let's try to feel the gap, and talk about LDAP injections.
  _yoast_wpseo_linkdex: '87'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:1;s:7:"replies";i:1;s:7:"authors";i:2;s:14:"recent_authors";a:2:{i:0;O:8:"stdClass":3:{s:20:"comment_author_email";s:25:"artem.smotrakov@gmail.com";s:14:"comment_author";s:5:"Artem";s:7:"user_id";s:1:"1";}i:1;O:8:"stdClass":3:{s:20:"comment_author_email";s:15:"qwdqd@jfhda.com";s:14:"comment_author";s:6:"wdqwed";s:7:"user_id";s:1:"0";}}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617978452'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/security/ldap-injections.html"
---
Everybody knows about SQL injections. It's like a celebrity in the world of software security. But there are much more many different types of injection attacks which may feel jealous about popularity of SQL injections. That's not fair. Let's try to feel the gap, and talk about LDAP injections.

[Версия на русском.](/fun-ru/security-ru/ldap-injections-ru.html)

<!--more-->

## What is LDAP?

LDAP stands for Lightweight Directory Access Protocol. It's a client-server binary protocol which lets clients access directory services. LDAP normally runs over TCP/IP, but it's also much better to use TLS.&nbsp;LDAP is defined in [RFC 2251](https://www.ietf.org/rfc/rfc2251.txt).

To make a&nbsp;long story short, LDAP server contains an attribute-based database. LDAP defines a way how clients can update and search data in this database.

There are several [implementations of LDAP](https://en.wikipedia.org/wiki/List_of_LDAP_software) client and servers, for example:

- OpenLDAP
- Java has an LDAP client API (JNDI)

## What is an LDAP injection?

LDAP injection is an injection attack in which&nbsp;an attacker can insert malicious LDAP statements in to the original LDAP query used by an application. As a result, an attacker may be able to send malicious LDAP requests to the LDAP server which may lead to security implications such as reading or updating sensitive information. LDAP injections usually occur because an application fails to properly sanitize untrusted data which may come from an adversary.

## An example of LDAP injection

Let's assume that we have an internal LDAP server behind the firewall. This LDAP server is used by an application for user authentication. The application requests user's credentials, and then checks if they are valid by searching for a record in the LDAP database.

For demo purposes, we can use a local LDAP server based on [Ldaptor](https://ldaptor.readthedocs.io/en/latest/)&nbsp;(it requires [Twisted](https://github.com/twisted/twisted)&nbsp;which can be installed with pip). Ldaptor team kindly provides&nbsp;[an example of LDAP server](https://ldaptor.readthedocs.io/en/latest/quickstart.html#ldap-server-quick-start)&nbsp;which I borrowed:

<script src="https://gist.github.com/artem-smotrakov/0316503009bd1d6a6ce39aa1be171235.js"></script>

The server above can be simply started with `python ldapserver.py` command.

Here is an example of vulnerable application:

<script src="https://gist.github.com/artem-smotrakov/b81033aee088c09091399774dd951308.js"></script>

The application takes a username and a password from command line parameters, and then sends a search request to the LDAP server to get&nbsp;a corresponding user account. If a record was found, it grants access. Here is how it runs with valid and invalid credentials:

```
$ javac -d classes LDAPLogin.java 
$ java -classpath classes LDAPLogin bob secret
LDAP query: (&(uid=bob)(userPassword=secret))
Access granted
$ java -classpath classes LDAPLogin bob wrong
LDAP query: (&(uid=bob)(userPassword=wrong))
Access denied
```

The problem here is that the application puts a username and a password "as is" to an LDAP search query. This allows an attacker to modify the query which the application sends to the LDAP server. It can be used, for example, for bypassing authentication. If an attacker doesn't know Bob's password, he can provide `bob)(|(uid=bob` string as a username and `wrong)` as a password which let him in.

```
$ java -classpath classes LDAPLogin "bob)(|(uid=bob" "wrong)"
LDAP query: (&(uid=bob)(|(uid=bob)(userPassword=wrong)))
Access granted
```

You can see that the application sent `(&(uid=bob)(|(uid=bob)(userPassword=wrong)))` request to the LDAP server. LDAP uses "Polish notation" for search queries. This request means "(uid == bob) and (uid=bob or userPassword=wrong)". This statement is always true for the record for Bob's account, so that the&nbsp;search request always returns a record for Bob's account even if provided password is wrong. As a result, the attacker can bypass authentication.

The actual exploit for an LDAP injection may depend on multiple things such as:

- Original LDAP search request
- Version of LDAP server
- Application logic

There are three main types of LDAP search queries:

- A query doesn't use any logical operators. It looks like "(field=value)". Even if we can put anything to `value`, we can't use any logical operator in such search request because Polish notation requires logical operators to go first. The only option here is to try to exploit a vulnerability of LDAP server (for instance, a buffer overflow, if it has it) which can be triggered by a malicious search request.
- AND operator goes first. It looks like `(&(field1=value1)(field2=value2))`. This kind of search request is used by the application above.
- OR operator goes first. It looks like `(|(field1=value1)(field2=value2))`.

It's good to know what type of search query is used by a vulnerable application for successful attack.

An application may also use LDAP request to update data in database. LDAP uses&nbsp;LDAP Data Interchange Format (LDIF) for updating database. LDIF represents a list of commands for adding, deleting and modifying database content. Basically it's&nbsp;a set of records, one record for each&nbsp;command. `ldapserver.py` above contains an example of LDIF data which adds three records to the directory. If an attacker can inject malicious data to LDIF,&nbsp;he can use a couple of CRLF sequences, and then add malicious commands. It's kinda similar to HTTP request splitting.

Version of LDAP software of server side&nbsp;may also matter. Sometimes an attacker can use an LDAP injection in a way that the query&nbsp;to LDAP server results to&nbsp;a couple of&nbsp;LDAP search request. If an LDAP server uses only first search request, and ignore others, then&nbsp;it may help for successful attack. Note that syntax of LDAP search requests doesn't allow commenting out the rest of request like it's possible in SQL.

## Blind LDAP injections

Blind LDAP injections are similar to blind SQL injections. An application may be vulnerable to LDAP injection, but it may not print out all requested fields. This doesn't allow an attacker to simply dump the content of LDAP directory. But if the application shows somehow if LDAP search requests with injected data succeed or not, then this behavior allows an attacker to ask yes/not questions. As a result, it may be possible to implement an efficient bruteforce attack, and extract data from the LDAP database.

Let's consider the following application. It takes a user ID, search for a corresponding record in LDAP directory. If a record was found, it prints out user's phone number. If not, it says that nobody found. The application can use LDAP server above.

<script src="https://gist.github.com/artem-smotrakov/e4d3bac16fa3404d89c9f09b830d8513.js"></script>

Here is how it's supposed to be normally used:

```
$ javac -d classes LDAPInfo.java 
$ java -cp classes LDAPInfo bob
LDAP query: (&(uid=bob)(objectClass=person))
Phone: telephoneNumber: 555-9999
$ java -cp classes LDAPInfo boba
LDAP query: (&(uid=boba)(objectClass=person))
Nobody found!
```

You might notice that LDAPInfo is vulnerable to LDAP-injection attack. Although it prints out only user's phone number, an attacker can still extract data from other fields. For example, an attacker can check if `userPassword` field for Bob's account starts with 'a' letter:

```
$ java -cp classes LDAPInfo "bob)(userPassword=a*"
LDAP query: (&(uid=bob)(userPassword=a*)(objectClass=person))
Nobody found!
```

You can see that if you pass `bob)(userPassword=a*` string as a user name, it results to 'Nobody found!' message. The application built `(&(uid=bob)(userPassword=a*)(objectClass=person))` search query which didn't return anything because `userPassword` for Bob's record doesn't start with 'a'. Now an attacker can enumerate first letters:

```
$ java -cp classes LDAPInfo "bob)(userPassword=a*"
LDAP query: (&(uid=bob)(userPassword=a*)(objectClass=person))
Nobody found!
$ java -cp classes LDAPInfo "bob)(userPassword=b*"
LDAP query: (&(uid=bob)(userPassword=b*)(objectClass=person))
Nobody found!
$ java -cp classes LDAPInfo "bob)(userPassword=c*"
LDAP query: (&(uid=bob)(userPassword=c*)(objectClass=person))
Nobody found!
[...]
$ java -cp classes LDAPInfo "bob)(userPassword=s*"
LDAP query: (&(uid=bob)(userPassword=s*)(objectClass=person))
Phone: telephoneNumber: 555-9999
```

Once we reached letter 'c', the application returned Bob's phone number. That means that Bob's password starts with 's'. Then, an attacker starts searching for second letter with usernames like `bob)(userPassword=sa*`, `bob)(userPassword=sb*`, `bob)(userPassword=sc*` and so on.&nbsp;This allows to implement an efficient bruteforce attack, and extract the password from the database.

Here is a simple script which demonstrates such a bruteforce attack:

<script src="https://gist.github.com/artem-smotrakov/e624953f7843f8a67239e8db332c3333.js"></script>

## How to prevent&nbsp;LDAP injections

Nothing surprising here. Input validation and sanitation help to prevent LDAP injections. Applications should escape all data that comes from untrusted sources and which is used in LDAP queries. OWASP has an article about it (see below) which may be applied not only to Web applications.

Another measure is disabling indexing of fields which may contain sensitive information like passwords. For example, If `userPassword` fieds&nbsp;is not indexed, then a search request with `(userPassword=a*)` statement will result to a error like the following:

```
$ ldapsearch -h ldap.server m -x -b "dc=test,dc=com" "(&(uid=test)(userPassword=a*))"
....
# search result
search: 2
result: 53 Server is unwilling to perform
text: Function Not Implemented, search filter attribute userpassword is not indexed/cataloged

# numResponses: 1
```

Although, an attacker may still be able to extract info from other indexed fields.

Links:

- [RFC 4511: Lightweight Directory Access Protocol (LDAP): The Protocol](https://tools.ietf.org/html/rfc4511)
- [LDAP injection (OWASP)](https://www.owasp.org/index.php/LDAP_injection)
- [LDAP Injection Prevention Cheat Sheet (OWASP)](https://www.owasp.org/index.php/LDAP_Injection_Prevention_Cheat_Sheet)
- [LDAP injection & Blind LDAP injection in Web applications](http://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)
