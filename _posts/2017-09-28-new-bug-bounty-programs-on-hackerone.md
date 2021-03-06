---
layout: post
title: New bug bounty programs on HackerOne for open source libraries
date: 2017-09-28 11:52:59.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Fuzzing
- Open Source
- Security
permalink: "/en/security/new-bug-bounty-programs-on-hackerone.html"
---
There are a couple of new bug bounty programs on HackeOne for popular open source libraries:

- libcap
- ImageMagick
- libpng
- GraphicsMagick
- curl
- tcpdump

They just started on last week (Sep 22nd, 2017). You can find the rules, scope and other details on&nbsp;[HackerOne](https://hackerone.com/ibb-data)

Those are well-known tools and libraries, and they have already gotten quite much attention from the security community. So, looks like it's going to be challenging to discover new issues there. Looking for a challenge? This may be a good one for sure. By the way, minimum bounty is $500. Not too much, but you also are going to get some credit for making the world better.

The libraries are mostly written in C/C++, so you may want to start with fuzzing. Although, if you search for fuzzing results for the libs above, you are going to find that security researches put some effort on it. On the other hand, it's never worse to try even harder. Someone can also contribute to&nbsp;[Google's OOS-fuzz project](https://github.com/google/oss-fuzz), and add support for fuzzing those libraries.&nbsp;OSS-fuzz already has&nbsp;[libpng and curl](https://github.com/google/oss-fuzz/tree/master/projects), but seems like there may be some room for&nbsp;libcap,&nbsp;ImageMagick,&nbsp;GraphicsMagick and tcpdump.

Good luck!

