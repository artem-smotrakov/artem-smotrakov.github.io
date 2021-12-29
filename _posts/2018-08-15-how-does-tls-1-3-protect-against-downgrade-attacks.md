---
layout: post
title: How does TLS 1.3 protect against downgrade attacks?
date: 2018-08-15 20:03:24.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Security
- TLS
- TLS 1.3
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_primary_category: '157'
  _yoast_wpseo_metadesc: One of the problems in older TLS versions is a lack of protection
    against downgrade attacks. Let's see how TLS 1.3 can proctect us.
  _yoast_wpseo_focuskw: TLS 1.3
  _yoast_wpseo_linkdex: '67'
  rp4wp_auto_linked: '1'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:0;s:7:"replies";i:0;s:7:"authors";i:0;s:14:"recent_authors";a:0:{}}
  _yoast_wpseo_estimated-reading-time-minutes: '4'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617979269'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/security/how-does-tls-1-3-protect-against-downgrade-attacks.html"
---
<!-- wp:paragraph -->

TLS 1.3 standard was finally published in August 2018. The authors tried to address the problems which unfortunately exist in older versions of the TLS protocol. One of the problems is a lack of protection against downgrade attacks.

<!-- /wp:paragraph -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## What is a downgrade attack against TLS?

<!-- /wp:heading -->

<!-- wp:paragraph -->

Let's assume that both TLS client and server support modern TLS protocol versions (like TLS 1.2) and cipher suites with strong cryptographic algorithms. The goal of a downgrade attack for TLS is to trick a client and a server to use an older protocol version or insecure parameters for the TLS connections. After a successful downgrade attack, an adversary may try to exploit known (or unknown) flaws in older protocol versions or weak cryptographic algorithms. A downgrade attack often requires the adversary to be able to intercept and modify the network traffic which is usually called man-in-the-middle. This is a quite powerful type of adversary - it seems unlikely that average people are able to implement such an attack.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

There are several well-known examples of downgrade attacks such as POODLE,&nbsp;FREAK and&nbsp;Logjam.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Typically, modern TLS clients and servers support old protocol versions and weak cryptographic algorithms for backward compatibility with older client and servers. One of the simplest and most reliable mitigation for downgrade attacks which can be easily applied to modern TLS implementations is just disabling insecure protocol versions and algorithms. But nevertheless such a simple mitigation may cost too much because of compatibility issues.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## What does TLS 1.3 offer to prevent downgrade attacks?

<!-- /wp:heading -->

<!-- wp:paragraph -->

Here is how the TLS 1.3 standard defines a downgrade protection:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> The cryptographic parameters should be the same on both sides and should be the same as if the peers had been communicating in the absence of an attack

<!-- /wp:quote -->

<!-- wp:paragraph -->

In other words, a good downgrade protection mechanism makes sure that client and server always negotiate the most secure protocol versions and cryptographic parameters even if there is a bad guy in the middle.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

TLS 1.3 provides two measures to prevent downgrade attacks. First, it requires both client and server to send a Finished message which contains a MAC over all previous handshake messages, so that both client and server ensure that the negotiated parameters have not been modified in the middle by an attacker. Here is how exactly [TLS 1.3 standard defines the content of Finished message](https://tools.ietf.org/html/rfc8446#section-4.4.4):

<!-- /wp:paragraph -->

<!-- wp:html -->

```
finished_key =
       HKDF-Expand-Label(BaseKey, "finished", "", Hash.length)

   Structure of this message:

      struct {
          opaque verify_data[Hash.length];
      } Finished;

   The verify_data value is computed as follows:

      verify_data =
          HMAC(finished_key,
               Transcript-Hash(Handshake Context,
                               Certificate*, CertificateVerify*))

      * Only included if present.
```

<!-- /wp:html -->

<!-- wp:paragraph -->

`Certificate` and `CertificateVerify` messages are included only if present. `finished_key`&nbsp;is derived from one of the negotiated handshake secrets.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Second, [TLS 1.3 provides&nbsp;a downgrade protection mechanism which is embedded in the server's random nonce](https://tools.ietf.org/html/rfc8446#section-4.1.3) in `ServerHello` message. If a TLS 1.3 server sees that it's only possible to negotiate an older protocol version, then TLS 1.3 requires to set the last 8 bytes of their `ServerHello.random` field to one of the predefined values:

<!-- /wp:paragraph -->

<!-- wp:list -->

- If negotiating TLS 1.2, then the last 8 bytes have to be&nbsp;`44 4F 57 4E 47 52 44 01`
- If negotiating TLS 1.1 or even older protocol versions, then the last 8 bytes have to be&nbsp;`44 4F 57 4E 47 52 44 00`

<!-- /wp:list -->

<!-- wp:paragraph -->

Then, TLS 1.3 says that a client has to check that the last 8 bytes of received `ServerHello.random` are not equal to either of the values above, and if so, the connection has to be terminated.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

By the way, the first 7 bytes of the byte-sequences above mean "DOWNGRD". In other words, TLS 1.3 specification asks client and server to behave like the following:

<!-- /wp:paragraph -->

<!-- wp:list -->

- I am a server, and I support TLS 1.3. I got a connection from a client which says it only supports TLS 1.2 or lower. That's okay but I am going to put the message "DOWNGRD" to my `ServerHello.random`. That will let the client know that I actually support a higher protocol versions but I was asked to use a lower version.
- I am a client, and I support TLS 1.3. In my `ClientHello` message, I asked the server to use TLS 1.3. But suddenly I got a ServerHello message which says that the server only supports TLS 1.2 or lower. Let me check if `ServerHello.random` contains "DOWNGRD" message, and if so, somebody in the middle is trying to run a downgrade attack against us, and we should stop talking.

<!-- /wp:list -->

<!-- wp:paragraph -->

If an attacker in the middle removed the "DOWNGRD" message from `ServerHello.random`, that would not help much because client and server use `ServerHello.random` in a key exchange process. The server is going to use the original value anyway, so that handshake would fail in this case.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Let's hope that all implementations of TLS 1.3 protocol won't forget to implement those measures against downgrade attacks.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

P.S.: Here is a small homework assignment for you. `ServerHello.random`&nbsp;contains 32 bytes. What would be the probability of randomly picking up "DOWNGRD + 00/01"&nbsp; by accident for a TLS 1.3 connection? Should we worry about it?

<!-- /wp:paragraph -->

