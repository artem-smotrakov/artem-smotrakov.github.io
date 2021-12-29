---
layout: post
title: Running an SSH server in a Docker image as a debugging mode
date: 2018-01-21 22:18:53.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tips and Tricks
tags:
- Docker
- Python
- SSH
meta:
  _edit_last: '1'
  _aioseop_opengraph_settings: a:14:{s:32:"aioseop_opengraph_settings_title";s:0:"";s:31:"aioseop_opengraph_settings_desc";s:0:"";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}
  _aioseop_keywords: docker,python,ssh,remote,debugging,debug,PyCharm
  _aioseop_description: How to implement a switch between your application and SSH
    server in a Docker container for debugging purposes.
  _aioseop_title: Running an SSH server in a Docker image as a debugging mode
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: ''
  _yoast_wpseo_focuskw_text_input: SSH
  _yoast_wpseo_focuskw: SSH
  _yoast_wpseo_metadesc: Here is a couple of notes about running an SSH server in
    a Docker container for application debugging purposes.
  _yoast_wpseo_linkdex: '74'
  _yoast_wpseo_content_score: '30'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:1;s:7:"replies";i:0;s:7:"authors";i:1;s:14:"recent_authors";a:1:{i:0;O:8:"stdClass":3:{s:20:"comment_author_email";s:21:"devangnaraj@gmail.com";s:14:"comment_author";s:12:"Devangna
    Raj";s:7:"user_id";s:1:"0";}}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617984294'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/tips-and-tricks-en/running-an-ssh-server-in-a-docker-image-as-a-debugging-mode.html"
---
I was recently debugging one Python application which ran in a Docker container. At some point, I'd like to debug the app in the container. At first, I was wondering if there is a way to run a Python application with a debug agent like you can do with Java, so that the agent listened in a port for incoming connections from a remote debugger. Unfortunately I didn't find a convenient way how I could remotely debug my Python app. I found an [article](https://medium.com/@furkanpur/remote-python-debug-to-docker-container-over-ssh-by-using-pycharm-44a9b6e82206) which describes how you can debug a Python application remotely with PyCharm IDE and SSH. If I understood correctly, PyCharm can deploy your application to a Docker container via SSH, then do some magic which is called "remote interpreter", so as a result, you can debug the application from your local PyCharm installation. Looks like this feature is available only in a commercial PyCharm version, but I had only a community edition.

<!--more-->

So, I gave up and decided just to connect to an SSH server which is run in a Docker container, and run my Python application manually with a debugger. Yes, hardcore only.

One of the simplest solution is just to have a switch between "production" and "debug" modes. That looked to me as a pretty simple and straightforward solution, but the problem is that you can't run both SSH server and a Python application in the same Docker container. Or, at least I don't know how to do that. If you add multiple CMD instructions to a Dockerfile, only one is going to be executed. It might be possible to have two Dockerfiles. The first one could run the application, and the second one could start an SSH server. But I didn't want to have two Dockerfiles which had a common part which setup environment for the application.

So, if such a switch can't be added to a Dockerfile, let's add a wrapper bash-script which has this logic. We can use an environment variable to switch between modes.

Here is a Dockerfile, `wrapper.sh` and a simple Python application which I used:

<script src="https://gist.github.com/artem-smotrakov/f84a7af8bd4e9ab2d29f5f9076b439e2.js"></script>

The following command builds a Docker image:

`docker build --tag vivaldi .`

Here is how you can run the application in a regular mode:

`docker run vivaldi`

And here is how you can switch to an SSH server which runs in the same Docker container:

`docker run -p 8022:22 -e DEBUG=yes vivaldi`

Here we forward a local port 8022 to 22 port in our container. Then, we can connect to the running container via SSH:

`$ ssh root@localhost -p 8022`

Good luck!

Links:

- [Dockerize an SSH service](https://docs.docker.com/engine/examples/running_ssh_service/#run-a-test_sshd-container)
- [Remote Python Debug to Docker Container over Ssh by using PyCharm](https://medium.com/@furkanpur/remote-python-debug-to-docker-container-over-ssh-by-using-pycharm-44a9b6e82206)
