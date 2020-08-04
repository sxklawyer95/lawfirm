---
title: docker introduction
tags:
  - Docker
date: 2019-12-03
---

## Containers

During the 1990s and early 2000s, the standard way to deploy applications to the internet was to get a server instance, copy the code or binary onto the instance, and then start the program. This worked great for a while, but soon complications began to arise:

- Code that worked on the developer's machine might not work on the server.
- Programs that ran perfectly on a server instance might fail upon applying the latest patch to the server's OS.
- For every new instance added as part of a service, various installation scripts had to be run so that we can bring the new instance to be on par with all the other instances. This can be a very slow process.
- Extra care had to be taken to ensure that the new instance and all the software versions installed on it are compatible with the APIs being used by our program.
- It was also important to ensure that all config files and important environment variables were copied to the new instance; otherwise, the application might fail with little or no clue.
- Usually the version of the program that ran on local system versus test system versus production system were all configured differently, and this meant that it was possible for our application to fail on one of the three types of systems. If such a situation occurred, we would end up having to spend extra time and effort trying to figure out whether the issue is specific to one particular instance, one particular system, and so on.

**Containers** try to solve this problem using OS-level virtualization.

All programs and applications are run in a section of memory known as **user space**. This allows the operating system to ensure that a program is not able to cause major hardware or software issues.

The real advantage of containers is that they allow us to run applications in isolated user spaces, and we can even customize the following attributes of user spaces:

- Connected devices such as network adapters and TTY
- CPU and RAM resources
- Files and folders accessible from host OS

## Docker

![docker-container-versus-vm](https://sherlockblaze.com/resources/img/daily/2019-12-03/docker-container-versus-vm.png)

The biggest advantage of a VM is that we can run different types of OSes on a system, for example, Windows, FreeBSD, and Linux. In the case of Docker, we can run any flavor of Linux, and the only limitation is that it has to be Linux.

The biggest advantage of Docker containers is that since it runs natively on Linux as a discrete process making it lightweight and unaware of all the capabilities of the host OS.

![docker-architecture](https://sherlockblaze.com/resources/img/daily/2019-12-03/docker-architecture.png)

- **Dockerfile**: It consists of instructions on how to build an image that runs our program.
- **Docker client**: This is a command-line program used by the user to interact with Docker daemon.
- **Docker daemon**: This is the Daemon application that listens for commands to manage building or running containers and pushing containers to Docker registry. It is also responsible for configuring container networks, volumes, and so on.
- **Docker images**: Docker images contain all the steps necessary to build a container binary that can be executed on any Linux machine with Docker installed.
- **Docker registry**: The Docker registry is responsible for storing and retrieving the Docker images. We can use a public Docker registry or a private one. Docker Hub is used as the default Docker registry.
- **Docker Container**: The Docker container is different from the Container we have been discussing so far. A Docker container is a runnable instance of a Docker image. A Docker container can be created, started, stopped, and so on.
- **Docker API**: The Docker client we discussed earlier is a command-line interface to interact with Docker API. This means that the Docker daemon need not be running on the same machine as does the Docker client. The default setup that we will be using throughout the book talks to the Docker daemon on the local system using UNIX sockets or a network interface.