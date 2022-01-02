---
layout: post
title: Examples of DNS rebinding attacks
date: 2018-01-23 22:48:07.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- DNS
- Security
- Web security
permalink: "/en/security/examples-of-dns-rebinding-attacks.html"
---
<p style="text-align: justify;">DNS rebinding attacks have been known for quite a long time. For example, Stanford Web Security Research Team posted <a href="https://crypto.stanford.edu/dns/">a whitepaper about DNS rebinding attacks</a> in 2007. But even if it's a well-known type of attacks, nowadays you still can find software systems which are vulnerable to DNS rebinding attacks. For example, Google Project Zero recently discovered such <a href="https://bugs.chromium.org/p/project-zero/issues/detail?id=1471&amp;desc=2">problems in Blizzard Update Agent</a> and <a href="https://bugs.chromium.org/p/project-zero/issues/detail?id=1447">BitTorent Transmission Daemon</a>.</p>
<p></p>
<h2>What is Same-Origin Policy?</h2>
<p style="text-align: justify;">Theoretically, Same-Origin Policy (SOP) which is implemented in a web browser is supposed to prevent scripts on client side to load resources from other websites (except a couple of cases which are considered safe enough). In other words, scripts on client side are only allowed to access content on the same host that served the script. One of the key steps here is comparing domain names. Let's assume that a web browser opens <code>http://ostap.com/index.html</code> which contains code on Javascript. The Javascript code then tries to use <code>XMLHttpRequest</code> to download the content of <code>http://hooves.com/secret.html</code> and display it. This attempt should be denied by the web browser if it enforces SOP because SOP allows a script to access only content from <code>ostap.com</code>.</p>
<h2>Why Same-Origin Policy is important?</h2>
<p style="text-align: justify;">Let's imagine that <code>http://ostap.com</code> is a public website which is owned by an adversary. Let's also assume that <code>http://hooves.com</code> website is in a private network which belongs to "Horns and hooves" company. The private network is not accessible from the Internet. Finally, let's assume that the adversary don't have access to the private network, but he still wants to know what <code>http://hooves.com/secret.html</code> is.</p>
<p style="text-align: justify;">Let's imagine that the adversary put an evil client-side script on <code>http://ostap.com/index.html</code> which tries to download the content of <code>http://hooves.com/secret.html</code>. Then, the adversary figured out that Mr. Panikovsky works at "Horns and Hooves" company, and his computer connected to the private network, so that Mr. Panikovsky can access <code>http://hooves.com/secret.html</code> from his browser. Next, the adversary sends an email to Mr. Panikovsky and tricks him to open <code>http://ostap.com/index.com</code> in a hope that the evil script runs in Mr. Panikovsky's web browser, download <code>secret.html</code>, and send it back to the adversary.</p>
<p style="text-align: justify;">But this tricky scenario is going to fail because the web browser which Mr. Panikovsky uses implements Same-Origin Policy. Since the evil script was originally downloaded from <code>ostap.com</code>, then it's not allowed to access content from <code>hooves.com</code>. As a result, the adversary's hopes break. That's why Same-Origin Policy is important.</p>
<h2>What is a DNS rebinding attack?</h2>
<p>The goal of DNS rebinding attack is to overcome restrictions which are enforced by Same-Origin Policy.</p>
<p style="text-align: justify;">First, the adversary registers a domain name such as <code>ostap.com</code> and delegates it to a DNS server which the adversary controls. Next, the adversary configures the DNS server to send responses with short TTL (time-to-live) to prevent the responses from being cached. Finally, the adversary starts an HTTP server which runs a malicious website <code>http://ostap.com</code>.</p>
<p style="text-align: justify;">When Mr. Panikovsky opens the malicious website, the adversary's DNS server responds with the IP address of the HTTP server which hosts the malicious website <code>http://ostap.com</code>. The web server returns a web page which contains Javascript which runs in the user's web browser. The Javascript code then accesses the original website on the original domain ostap.com to download some additional resource <code>http://ostap.com/sectet.html</code>. When the web browser runs the Javascript it sends a new DNS request for the domain (because of short TTL), but the attacker's DNS replies with a new IP address. For example, the attacker's DNS server can reply with an internal IP address such as the IP address of hooves.com. As a result, the web browser actually loads <code>http://hooves.com/secret.html</code> instead of <code>http://ostap.com/secret.html</code> which means that Same-Origin Policy is bypassed.</p>
<pre class="console">      Web browser                            http://ostap.com
    (Mr. Panikovsky)                           (adversary)
           |                                         |
   +----------------+                                |
   | load ostap.com |                                |
   +----------------+                                |
           |                                         |
           |   DNS: resolve ostap.com                |
           |----------------------------------------&gt;|
           |                                         |
           |   DNS: IP of ostap.com (short TTL)      |
           |        1.1.1.1                          |
           |&lt;----------------------------------------|
           |                                         |
           |   HTTP: get http://1.1.1.1/index.html   |
           |----------------------------------------&gt;|
           |                                         |
           |   HTTP: index.html with a script        |
           |&lt;----------------------------------------|
           |                                         |
   +-------------------------------+                 |
   | run the script                |                 |
   | which requests                |                 |
   | http://ostap.com/secret.html  |                 |
   +-------------------------------+                 |
           |                                         |
           |   DNS: resolve ostap.com again          |
           |        because of short TTL             |
           |----------------------------------------&gt;|
           |                                         |
           |   DNS: IP of hooves.com                 |
           |        2.2.2.2                          |
           |&lt;----------------------------------------|
           |                                         |
           |   HTTP: get http://2.2.2.2/secret.html  |
           |----------------------------------------
\>| | | | HTTP: secret.html | | game over, SOP bypassed | | then the script can send | | secret.html to the adversary | | |

This is a pretty simplified description of one possible DNS rebinding attack when an adversary tries to access a resource from a private network. There may be other use cases of DNS rebinding attacks, for example, an attacker can return an IP address of some web server on the Internet for some purpose, or an attacker may want to access a local server running on localhost (see below).

## Example:&nbsp;Blizzard games were vulnerable to DNS rebinding attack

If you play Blizzard's online games you should be probably aware of the recent [security vulnerability in their Blizzard Update Agent](https://bugs.chromium.org/p/project-zero/issues/detail?id=1471&desc=2). The issue has been discovered by Google's Project Zero. Making a long story short, if you play Blizzard games, you may be in a danger.

Diving a bit more into the issue, an attacker can trick a user to open a malicious website which is controlled by the attacker. Next, the attacker triggers a DNS rebinding attack to send malicious requests to a local RPC server which is a part of Blizzard Update Agent installed on user's machine. Then, the attacker can maliciously use Agent's functions for evil. Although it's not very clear what exactly the RPC server allows to do, so the real consequences of a successful attack is not clear. According to the reporter, Blizzard Update Agent "accepts commands to install, uninstall, change settings, update and other maintenance related options" which sounds like there is a chance that an adversary can potentially get a chance to run arbitrary code on user's machine. But it actually depends on what privileges the Agent has on user's machine. Anyway, it sounds pretty dangerous.

The difference between this scenario and the one which was described in the previous section is that an adversary tries to access a service on localhost instead of an IP address in a private network (well, localhost can be also considered as a private network if it's configured to be accessed only from the same host of from specific set of hosts).

Blizzard has already fixed the issue, but the solution doesn't look too good (see the original bug report above). They said they are working on another solution.

## Preventing DNS rebinding attacks

Here is a lesson which can be learnt from this issue. Let's imagine that you have a project which contains a server which provides some kind of HTTP API. Then let's assume that this server is supposed to be accessed only from the same host or from a private network. It even can bind a port to `0.0.0.0/localhost`, or the port can be protected from external connections by a firewall. This kind of assumption may make you think that your server is protected from malicious requests, but it's not actually true. An attacker can still run a DNS rebinding attack to access the server. There is a couple of ways to mitigate this issue which may depend of the actual software system. One of possible ways to mitigate this problem is check if 'Host' header contains 'localhost' or any other allowed hostname. The server should reject a request if 'Host' header contains any unexpected hostname. In other words, a proper white-listing should be implemented for 'Host' header.

Have fun!

Links:

- [DNS rebinding](https://en.wikipedia.org/wiki/DNS_rebinding)
- [Protecting Browsers from DNS Rebinding Attacks](https://crypto.stanford.edu/dns/dns-rebinding.pdf)
- [blizzard: agent rpc auth mechanism vulnerable to dns rebinding](https://bugs.chromium.org/p/project-zero/issues/detail?id=1471&desc=2#maincol)
- [transmission: rpc session-id mechanism design flaw](https://bugs.chromium.org/p/project-zero/issues/detail?id=1447)
