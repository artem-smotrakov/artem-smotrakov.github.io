---
layout: post
title: TLS enhancements in Java 13
date: 2019-10-03 21:11:26.000000000 +01:00
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
- TLS
permalink: "/en/security/tls-enhancements-in-java-13.html"
---

Java 13 was released on Sep 13th, 2019. Although the new Java doesn’t contain major updates in security libraries, nevertheless it has several notable updates in the TLS implementation. Let’s take a closer look at how Java 13 helps to make your TLS connections faster and more secure.


<figure class="wp-block-image noborder"><img src="{{ site.baseurl }}/assets/images/2019/10/cowsay_do_you_know_about_tls_enhancements_in_java_13.jpg" alt="TLS enhancements in Java 13" class="wp-image-3481" /></figure>

<br />
<br />


<h3>Updated the default cipher suites&nbsp;order</h3>


A TLS cipher suite consists of several components: a key exchange algorithm, a digital signature algorithm, a cipher, a message authentication code. For example, the cipher suite <code>TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256</code> uses the following:


<ul>
<li>Elliptic Curve Diffie-Hellman Ephemeral (ECDHE) key exchange method</li>
<li>Elliptic Curve Digital Signature Algorithm (ECDSA)</li>
<li>AES cipher with 128-bit key in Galois/Counter Mode (GCM)</li>
<li>SHA256 hash algorithm</li>
</ul>


In TLS 1.3, the format of a cipher suite slightly changed but it’s not that important for us now because we’re going to focus on key exchange methods.


Each TLS implementation defines a default preference order for cipher suites which may be negotiated during TLS handshake. The TLS implementation in older Java versions preferred key exchange and signature algorithms in the following order:


<ul>
<li>ECDHE-ECDSA</li>
<li>ECDHE-RSA</li>
<li>RSA</li>
<li>ECDH-ECDSA</li>
<li>ECDH-RSA</li>
<li><strong>DHE-RSA</strong></li>
<li><strong>DHE-DSS</strong></li>
</ul>


You can see that two DHE algorithms go first (ECDHE-ECDSA and ECDHE-RSA) but the other two (DHE-RSA and DHE-DSS) are at the end of the list. One of the most important security features of DHE algorithms is forward secrecy which guarantees that session keys will not be compromised even if the private key of the server is compromised. Although the rest of the algorithms above don’t provide this security feature, they are more preferred than DHE-RSA and DHE-DSS.


In Java 13, the order of cipher suites has been changed to prefer algorithms that provide forward secrecy. Here is the new order:


<ul>
<li>ECDHE-ECDSA</li>
<li>ECDHE-RSA</li>
<li><strong>DHE-RSA</strong></li>
<li><strong>DHE-DSS</strong></li>
<li>ECDH-ECDSA</li>
<li>ECDH-RSA</li>
<li><strong>RSA</strong></li>
</ul>


Now the DHE algorithms go first. The new order improves security since the algorithms with forward secrecy have higher priority. Note that the priority of the RSA key exchange mechanism has been decreased. This key exchange algorithm has been deprecated in TLS 1.3.


Furthermore, the TLS implementation in Java 13 now prefers the server’s cipher suites during TLS handshake. When a client and a server are establishing a TLS connection, the client sends a list of supported cipher suites to the server. The order of the cipher suites means which cipher suites the client prefers more. The server has its own list of cipher suites which it supports and prefers. If there are more than one cipher suites in common, then the server has a choice: it can pick up the most preferred cipher suite from the client list or from its own list.


By default, older Java versions preferred the client’s cipher suites but Java 13 changed that, and now it prefers the server’s cipher suites. This way, the server can have more control over the security parameters of TLS connections.


<h3>X25519 and X448 Diffie-Hellman elliptic curve&nbsp;support</h3>


X25519 and X448 are modern elliptic curves that are used in Diffie-Hellman (DH) key agreement schemes. The curves are considered as better alternatives to the NIST's curves after it was discovered that NSA had potentially implemented a backdoor.


Java 13 now supports the x25519 and x448 elliptic curves for TLS versions 1.0, 1.1, 1.2, and 1.3. The TLS implementation in Java supports many elliptic curves. The x25519 now has the highest priority, but x448 goes after several other curves:


<code>x25519, secp256r1, secp384r1, secp521r1, x448, ...</code>


If necessary, the default order can be overridden by setting the <code>jdk.tls.namedGroups</code> system property. 


<h3>Stateless server</h3>


Establishing a TLS connection, which is usually called TLS handshake, requires several steps such as negotiating cryptographic mechanisms, authentication, key exchange and so on. These steps take time. If a TLS server is supposed to handle many connections, the performance of the TLS handshake may become an issue. One of the ways to improve TLS performance is to re-use the TLS sessions which have been established earlier.


Session tickets allow a TLS server to operate stateless. During the first TLS handshake, the TLS server sends internal session information in the form of an encrypted session ticket to a client. When the client connects to the server next time, it sends the session ticket to the server to resume the session. This can significantly improve the performance and scalability of the TLS server since the TLS handshake takes fewer steps. It can also improve memory usage of the TLS server since it has to store less information about TLS sessions.


Java 13 now supports session tickets. This feature is not yet enabled by default. To turn it on, you'll have to set the <code>jdk.tls.client.enableSessionTicketExtension</code> and <code>jdk.tls.server.enableSessionTicketExtension</code> system properties to true.


<h3>Displaying TLS configuration with&nbsp;keytool</h3>


For a long time, the keytool utility has been used for operations with cryptographic keys and keystores. In Java 13, keytool has been enhanced to display information about the TLS configuration. To print the info out, pass -showinfo -tls options to keytool. Currently, the tool shows only enabled TLS versions and cipher suites.


Here is what it looks like:


<pre class="wp-block-preformatted console"> $ keytool -showinfo -tls
 Enabled Protocols
 -----------------
 TLSv1.3
 TLSv1.2
 TLSv1.1
 TLSv1

 Enabled Cipher Suites
 ---------------------
TLS\_AES\_256\_GCM\_SHA384 TLS\_AES\_128\_GCM\_SHA256 TLS\_CHACHA20\_POLY1305\_SHA256 TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384 TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256 TLS\_ECDHE\_ECDSA\_WITH\_CHACHA20\_POLY1305\_SHA256 TLS\_ECDHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384 TLS\_ECDHE\_RSA\_WITH\_CHACHA20\_POLY1305\_SHA256 TLS\_ECDHE\_RSA\_WITH\_AES\_128\_GCM\_SHA256 TLS\_DHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384 ...





### Conclusion





Java 13 doesn't have major updates in security libraries. Besides the updates discussed above, the new Java version contains [159](https://bugs.openjdk.java.net/browse/JDK-8228403?jql=project%20%3D%20JDK%20AND%20resolution%20%3D%20Fixed%20AND%20fixVersion%20%3D%20"13"%20AND%20component%20%3D%20security-libs%20ORDER%20BY%20resolved%20DESC) bugs and minor enhancements in security libs including [50](https://bugs.openjdk.java.net/browse/JDK-8226338?jql=project%20%3D%20JDK%20AND%20resolution%20%3D%20Fixed%20AND%20fixVersion%20%3D%20"13"%20AND%20component%20%3D%20security-libs%20AND%20Subcomponent%20%3D%20javax.net.ssl%20ORDER%20BY%20resolved%20DESC) in the TLS implementation. The Java team constantly works on improving security functionality.





### References





- [JDK-8219545: Update the default enabled cipher suites preference](https://bugs.openjdk.java.net/browse/JDK-8219545)
- [JDK-8219657: Use server cipher suites preference by default](https://bugs.openjdk.java.net/browse/JDK-8219657)
- [JDK-8171279: Support X25519 and X448 in TLS](https://bugs.openjdk.java.net/browse/JDK-8171279)
- [JDK-8211018: Session Resumption without Server-Side State](https://bugs.openjdk.java.net/browse/JDK-8211018)
- [JDK-8219861: Add new keytool -showinfo -tls command for displaying TLS configuration information](https://bugs.openjdk.java.net/browse/JDK-8219861)
- [Forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy)
- [Curve25519](https://en.wikipedia.org/wiki/Curve25519)
- [Curve448](https://en.wikipedia.org/wiki/Curve448)


