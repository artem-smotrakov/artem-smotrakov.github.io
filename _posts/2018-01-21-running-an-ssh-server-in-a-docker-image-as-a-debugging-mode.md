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
permalink: "/en/tips-and-tricks-en/running-an-ssh-server-in-a-docker-image-as-a-debugging-mode.html"
---
I was recently debugging one Python application which ran in a Docker container. At some point, I'd like to debug the app in the container. At first, I was wondering if there is a way to run a Python application with a debug agent like you can do with Java, so that the agent listened in a port for incoming connections from a remote debugger. Unfortunately I didn't find a convenient way how I could remotely debug my Python app. I found an [article](https://medium.com/@furkanpur/remote-python-debug-to-docker-container-over-ssh-by-using-pycharm-44a9b6e82206) which describes how you can debug a Python application remotely with PyCharm IDE and SSH. If I understood correctly, PyCharm can deploy your application to a Docker container via SSH, then do some magic which is called "remote interpreter", so as a result, you can debug the application from your local PyCharm installation. Looks like this feature is available only in a commercial PyCharm version, but I had only a community edition.

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
