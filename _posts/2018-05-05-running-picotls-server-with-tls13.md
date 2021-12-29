---
layout: post
title: Running picotls TLS 1.3 server with AddressSanitizer and Docker
date: 2018-05-05 12:32:43.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tips and Tricks
tags:
- AddressSanitizer
- Docker
- Open Source
- picotls
- TLS
- TLS 1.3
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_primary_category: '243'
  _yoast_wpseo_focuskw_text_input: picotls
  _yoast_wpseo_focuskw: picotls
  _yoast_wpseo_metadesc: I have been experimenting with TLS 1.3 in tlsbunny project.
    This is a framework for building negative tests and fuzzers for TLS 1.3 implementations.
    For example, tlsbunny has several simple fuzzers for TLS structures like TLSPlaintext,
    Handshake, etc. It would not be worse to run those fuzzers against picotls server.
  _yoast_wpseo_linkdex: '52'
  rp4wp_auto_linked: '1'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:1;s:7:"replies";i:0;s:7:"authors";i:1;s:14:"recent_authors";a:1:{i:0;O:8:"stdClass":3:{s:20:"comment_author_email";s:27:"preetiagarwal1634@gmail.com";s:14:"comment_author";s:6:"preeti";s:7:"user_id";s:1:"0";}}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618017078'

permalink: "/en/tips-and-tricks-en/running-picotls-server-with-tls13.html"
---
[Picotls](https://github.com/h2o/picotls) is a TLS 1.3 implementation written in C. At the moment of writing this post, picotls implements [TLS 1.3 draft 26](https://tools.ietf.org/html/draft-ietf-tls-tls13-26).

I have been experimenting with TLS 1.3 in [tlsbunny](https://github.com/artem-smotrakov/tlsbunny) project. This is a framework for building negative tests and fuzzers for TLS 1.3 implementations. For example, tlsbunny has several simple fuzzers for TLS structures like TLSPlaintext, Handshake, ClientHello, etc. It would not be worse to run those fuzzers against picotls server.

<!--more-->

picotls is a tiny TLS 1.3 implementation, so building takes less than a minute. But I'd like to have a couple of features:

- run picotls server in a Docker container
- enable [AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer) to catch possible memory corruptions and leaks

Unfortunately picotls doesn't support AddressSanitizer out-of-the box (which may be a nice improvement), so `-fsanitize=address` option should be passed to the compiler via environment variables and CMake parameters.

The tlsbunny repository contains a Dockerfile which builds picotls and starts a local server. Furthermore, the repository already contains server key and certificate which you will need to start the server. So, the easiest way to build a Docker image with picotls is to clone tlsbunny repository and run `docker build`:

```
$ git clone https://github.com/artem-smotrakov/tlsbunny
$ cd tlsbunny
$ docker build --file src/main/docker/picotls/Dockerfile --tag picotls/server/tls13 .
```

Below you can find the details:

- `create_certs.sh`&nbsp;generates EC keys and certificates. It's a bit redundant but it should not be a problem.
- Dockerfile is a just a Dockerfile.
- A local server starts on `0.0.0.0:20101`. Note that `127.0.0.1`&nbsp;doesn't work because the port can't be exposed outside the container in this case.

<script src="https://gist.github.com/artem-smotrakov/9b65e69b05c8acbd1a8ef2799b39c588.js"></script>

Enjoy!

