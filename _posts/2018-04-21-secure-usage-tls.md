---
layout: post
title: An overview of secure usage of TLS
date: 2018-04-21 10:47:04.000000000 +01:00
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
- Web security
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_primary_category: '157'
  _yoast_wpseo_focuskw_text_input: TLS
  _yoast_wpseo_focuskw: TLS
  _yoast_wpseo_linkdex: '58'
  _yoast_wpseo_metadesc: 'The article describes how TLS can be used for establishing
    a secure connection: First, we''ll briefly discuss the protocol works. Next, we''ll
    talk about secure protocol versions and parameters. Finally, we''ll describe how
    TLS can be configures securely.'
  rp4wp_auto_linked: '1'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618017034'

permalink: "/en/security/secure-usage-tls.html"
---
The article describes how TLS protocols can be used for establishing a secure connection: First, we'll briefly discuss the protocol works. Next, we'll talk about secure protocol versions and parameters. Finally, we'll describe how TLS can be configures securely.



## What is SSL/TLS?

SSL stands for Secure Sockets Layer, and TLS stands for&nbsp;Transport Layer Security.

This is a family of cryptographic protocols which provide communications security over an untrusted network. The main goals of the protocols are the following:

- Confidentiality by encrypting data with symmetric ciphers (for example, AES-256 in GCM mode)
- Authentication of server (and client if required) by using&nbsp;using public-key cryptography
- Data integrity by using MAC (message authentication code)

A connection contains two main phases:

- Handshake which allows client and server to:
  - negotiate parameters for a secure connection such as encryption algorithm, authentication scheme, hash function, MAC algorithm
  - authenticate both server and client if necessary
  - securely exchange cryptographic keys
- Secure application data exchange using encryption algorithms and parameters established during handshake

You can find [more details about SSL/TLS protocols in Wikipedia](https://en.wikipedia.org/wiki/Transport_Layer_Security), or refer to the corresponding RFCs for technical details (for example,&nbsp;[RFC 5246](https://www.ietf.org/rfc/rfc5246.txt)&nbsp;which defines TLS 1.2).

## What TLS versions do we have?

At the moment of writing this post, there are several versions:

- SSL 3.0
- TLS 1.0
- TLS 1.1
- TLS 1.2
- Draft of TLS 1.3 which is proposed as the next version of TLS protocol

Even if the differences between versions above don't seem to be huge, the protocols above are not compatible.

## Which protocol versions should be used for a secure connection?

At the moment of writing this post, only TLS 1.2 protocol is considered secure. TLS 1.3 is the next version, and should be considered secure once the protocol specification is approved (unless someone finds a critical vulnerability in the protocol right away after publishing :) Other SSL/TLS versions are considered less secure due to various vulnerabilities and weaknesses in the protocols. For example, discovering the [POODLE attack](https://en.wikipedia.org/wiki/POODLE) resulted to prohibiting SSL 3.0.

Using TLS 1.2+ may not be enough for establishing a secure connection. The following aspects need to be taken into account as well:

- Only secure parameters of a connection such as strong [TLS cipher suites](https://en.wikipedia.org/wiki/Cipher_suite), strong key sizes and strong parameters for key exchange algorithms should be used
- Lower protocol versions should be disabled to prevent protocol [downgrade attacks](https://en.wikipedia.org/wiki/Downgrade_attack)
- Insecure modes and features such as renegotiation and&nbsp;compression should be disabled
- Server and client should use latest version of TLS libraries which contain security patches for discovered vulnerabilities in the protocol implementations like [Heartbleed](https://en.wikipedia.org/wiki/Heartbleed)

## What parameters should be used for a secure connection?

Client and server can use different parameters to establish a connection such as cipher suites, keys lengths, initial parameters for Diffie-Hellman key exchange algorithm.

First, client and server should use a secure cipher suite which:

- doesn't use weak cryptographic algorithms such as RC4, DES, 3DES
- uses an AEAD cipher such as AES in GCM mode
- uses strong hash functions such as SHA-256
- provides&nbsp;[forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy)&nbsp;by using ephemeral Diffie-Hellman key exchange (EDH and ECDHE) with strong initial parameters

Here is a list of good cipher suites for TLS 1.2:

1. ECDHE-RSA-AES256-GCM-SHA384
2. ECDHE-RSA-AES128-GCM-SHA256
3. DHE-RSA-AES256-GCM-SHA384
4. DHE-RSA-AES128-GCM-SHA256

[OWASP provides a good guideline for configuring cipher suites](https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet#Server_Protocol_and_Cipher_Configuration).

Second, a client should require server authentication, and then validate a server certificate chain including:

- correctness of digital signatures
- hostname validation
- certificate expiration
- revocation status
- the chain ends with a trusted certificate

Clients need to make sure that a server certificate uses strong cryptographic algorithms with strong key sizes.

The features above should be provided by a good protocol implementation. It would not be worse to mention again that you should not implement your own cryptographic library. It's not necessary to invent a wheel. Furthermore, implementing a cryptographic algorithms and protocols requires a solid knowledge in cryptography and paying attention to a lot of details. Just a single bug can ruin everything and make the implementation insecure. Instead, it's recommended by experts in cryptography and software security to use well-known cryptographic libraries such as OpenSSL, NSS or JSSE which we're going to discuss soon.

P.S. This is just an overview of secure usage of TLS but not an ultimate guideline.

