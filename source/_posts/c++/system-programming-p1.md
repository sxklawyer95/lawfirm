---
title: Getting Started with System Programming
tags:
  - c++
date: 2020-04-26
---

> All the summaries are from the book named [C++ System Programming Cookbook](https://www.amazon.com/System-Programming-Cookbook-system-level-programming/dp/1838646558).



- Learning the Linux fundamentals
  - architecture
  - shell
  - users
  - processes and threads
- Using a makefile to compile and link a program
- Using the **GNU Project Debugger (GDB)** to debug a program
- Handling a Linux bash error
- Handling Linux code error

## Technical Requirements

- Install [docker](www.docker.com).
- Pull the image from Docker Hub: `docker pull kasperondocker/system_programming_cook:latest`
- Run the Docker image with an interactive shell, with the help of the following command: `docker run -it --cap-add sys_ptrace kasperondocker/system_programming_cookbook:latest /bin/bash`
- `cd /BOOK`

## Learning the Linux fundamentals - architecture

Linux is a clone of the Unix operating system, it's a multiuser, multitasking operating system that runs on a wide variety of platforms. The Linux kernel has a monolithic architecture for performance reasons. This means **that it is self-contained in one binary, and all its services run in kernel space.**

The following diagram shows the main Linux building blocks:

![](https://sherlockblaze.com/resources/img/c++/linux-building-blocks.png)

- On the top layer, there are user applications, processes, compilers, and tools. This layer (which runs in a user space) communicates with the Linux kernel (which runs in kernel space) through system calls.
- **System libraries**: These are a set of functions through which an application can interact with the kernel.
- **Kernel**: This component contains the core of the Linux system. Among other things, it has the scheduler, networking, memory management, and filesystems.
- **Kernel modules**: These contain pieces of kernel code that still run in kernel space but are fully dynamic (in the sense that they can be loaded and unloaded with the running system). They typically contain device drivers, kernel code that is specific to a particular hardware module implementing a protocol, and so on. One huge advantage of the kernel modules it that users can load them without rebuilding the kernel.

**GNU** is an operating system that is free software. Indeed, GNU used alone is meant to represent a full set of tools, software, and kernel parts that an operating system needs. The GUN operating system kernel is called the **Hurd.** As the Hurd was not production-ready, GNU typically uses the Linux kernel, and this combination is called the **GNU/Linux operating system.**

So, what are the GNU components on a GNU/Linux operating system? Packages* such as the **GNU Compiler Collection(GCC)**, the **GNU C library**, GDB, the GNU Bash shell, and the **GNU Network Object Model Environment(GNOME)** desktop environment, to mention just a few.

Richard Stallman and the **Free Software Foundation(FSF)** -- of which Stallman is the founder -- authored the **free software definition** to help respect users' freedom. _Free software_ is considered any package that grants users the following for types of freedoms (so-called **essential freedoms**: [https://isocpp.org/std/the-standard](https://isocpp.org/std/the-standard)):

- The freedom to run the program as you wish, for any purpose (Freedom 0)
- The freedom to study how the program works and to change it, so it does your computing as you wish (Freedom 1). Access to the source code is a precondition for this.
- The freedom to redistribute copies so that you can help others (Freedom 2)
- The freedom to distribute copies of your modified versions to other (Freedom 3). By doing this, you can give the whole community a chance to benefit from your changes. Access to the source code is a precondition for this.

The concrete instantiation of these principles is in the GNU/GPL license, which FSF authored. All of the GNU packages are released under the GNU/GPL license.



