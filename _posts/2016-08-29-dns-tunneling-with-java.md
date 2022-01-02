---
layout: post
title: DNS tunneling with Java
date: 2016-08-29 19:18:57.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- DNS
- Java
- Security
permalink: "/en/security/dns-tunneling-with-java.html"
---
DNS tunneling may help you to bypass a firewall if DNS requests are allowed. Or, it can just get you a free Wi-Fi. There are a number&nbsp;standalone tools which allow you to setup a TCP-over-DNS tunnel.&nbsp;Here is a simple implementation of DNS tunneling with pure Java. It's not ready for using in real world, but it shows an idea how DNS tunneling can be implemented. The implementation works with standard JRE, and doesn't require any additional library.

(русская версия -&nbsp;[Java и свет в конце DNS туннеля](http://blog.gypsyengineer.com/fun-ru/dns-tunneling-with-java-ru.html))



## What is DNS tunneling?

The purpose of DNS (Domain Name System) protocol is to convert a domain name to an IP address. DNS requests&nbsp;are usually recursive. In other words, if a DNS server doesn’t know how it can resolve&nbsp;a domain name, it can&nbsp;send a request to other DNS servers. Let's assume that you are in a private network which has&nbsp;a firewall that blocks all connections to the Internet. Or, you just connected to a&nbsp;Wi-Fi spot&nbsp;which doesn't allow you to connect to the Internet either. Another possible situation is that we&nbsp;were able to deliver an exploit (for example, it may be a Java applet) to some computer in a private network. The exploit gives us an access to some&nbsp;machine in the internal network, but we can't establish a usual TCP connection to it because of a corporate firewall. If the&nbsp;firewall blocks usual TCP/UDP connections to machines outside the private network, but allows DNS requests, then it may be possible to establish a DNS tunnel which allows to transfer data outside the private network.

Let's say we have a client inside the private network, and a server outside it. There is a firewall which blocks all connections to the Internet except DNS requests. Now we want to establish a DNS tunnel between those client and server. The server runs its own DNS server which controls a subdomain (let's say "attacker.com"). The private network has its own internal DNS server. The client wants to send data to the server. To do that,&nbsp;the client can encode the data in BASE32, and make a hostname from it. For example, 'hello' string is going to be 'NBSWY3DP', so the client builds 'NBSWY3DP.attacker.com' hostname string. Then the client&nbsp;sends a request to the internal&nbsp;DNS server to resolve this hostname. The internal DNS server&nbsp;realizes that it can't resolve this request, so that it forwards the request to our DNS server which is responsible for subdomains of 'attacker.com'. The firewall is fine with&nbsp;DNS requests. The server decodes the data from BASE32-encoded subdomain, process it, and sends a DNS response which contains data for the client in a TXT field.

## Implementation of DNS tunneling with Java

Standard JRE supports DNS protocol out of the box, and doesn't require any other third-party library. More precisely, it's part of JNDI (Java Naming and Directory Interface). In other words, if an attacker could deliver a Java applet on a target machine, then the chance&nbsp;to successfully setup&nbsp;a DNS tunnel is quite big (well, now it may not be that easy to make a victim run a malicious applet with new Java versions than it used to be before).

Here is an example which shows an idea how DNS tunneling can be implemented with Java:

[https://github.com/artem-smotrakov/java-dns-tunneling](https://github.com/artem-smotrakov/java-dns-tunneling)

It's just a proof of concept, and not ready to use in real world.&nbsp;Here are main points:

1. "src" directory contains Java sources:
  1. DNSTunnelServer.java is a server which runs&nbsp;on attacker's host. It runs a malicious DNS server which is responsible for&nbsp;'attacker.com' domain. It also accepts commands that should be transferred to the victim's machine, and run there.
  2. DNSTunnelClient.java is a client which should be delivered to the victim's machine. The client runs a DNS client which is based on JNDI. It sends requests to the malicious&nbsp;DNS server (through an internal DNS server). DNS requests contain data which need to be delivered to the attacker (for example, it may be user's confidential data, or results of commands executed on user's machine). DNS responses contain commands from attacker which should be executed on victim's machine.
  3. Payload.java is an applet which needs to be delivered to victim's machine. The applet starts a&nbsp;DNSTunnelClient.
  4. CrockfordBase32.java implements BASE32 encoding and decoding.
  5. Command.java&nbsp;just&nbsp;runs a command with ProcessBuilder API.
  6. Server.java is a simple telnet server which listens on a port, waits for commands, and run them. Payload applet can be configured to run this server. It's not related to DNS tunneling, but was used for experiments with ProcessBuilder API.
  7. SimpleHttpServer is a simple HTTP server which can host Payload applet (see also index.html file).
2. "scripts" directory contains some useful scripts:
  1. The scripts require JAVA\_HOME environment variable to be set. The variable should contain a path to JDK 8 installation.
  2. compile.sh compiles sources.
  3. gen.sh packs&nbsp;compiled classes to a jar file, creates a self-signed certificate, and finally signs the jar file with this certificate.
  4. run\_http.sh just starts a&nbsp;SimpleHttpServer
3. manifest.mf is a manifest file for a jar. It contains "Permissions" attribute which is required by new Java versions.

## DNS tunnel testing

I used a couple of VMs for testing:

1. First, I started a usual DNS server&nbsp;(not&nbsp;DNSTunnelServer) on my laptop. This DNS server delegates requests for subdomains of 'attacker.com' to the malicious DNS server on 'attacker' machine.
2. Then, I started one VM (let's call it 'attacker' machine), and I ran a DNSTunnelServer instance there. This DNSTunnelServer is a DNS server for 'attacker.com' domain, and also accepts commands to run on 'victim'&nbsp;machine
3. Next, I started&nbsp;a&nbsp;SimpleHttpServer instance on&nbsp;'attacker' machine. This HTTP server just&nbsp;contains&nbsp;a malicious applet (Payload.jar) on http://attacker.com/index.html
4. Finally, I started another VM (let's call it 'victim' machine), opened a Web-browser there, and loaded http://attacker.com/index.html

As a result, I could type&nbsp;commands in DNSTunnelServer console, and those commands were sent and run on 'victim' machine.

Hope you use latest Java versions. If so, you will need to update Java security settings for Java plugin to be able to run the applet (you probably need to decrease security level, and add 'attacker.com' to the exclude list).

This project requires Java 8 or higher (it uses some new API introduced in Java 8, and fancy stuff like lambdas), but it doesn't look hard to&nbsp;adopt it to run with older Java versions.

