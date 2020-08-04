---
title: How Linux Works(Chapter Two)--Directory Hierarchy Essentials
tags:
  - Linux
date: 2019-03-04
---

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

Let's check the directory hierarchy of linux first.

![Linux directory hierarchy](https://sherlockblaze.com/resources/img/profession/linux/how-linux-works/linux-directory-hierarchy.png)

This picture offers a simplified overview of the hierarchy, showing some of the directories under `/`, `/usr`, and `/var`. Notice that the directory structure under `/usr` contains some of the same directory names as `/`.

Here are the most important subdirectories in `root`:

- `/bin` Contains ready-to-run programs(also known as an executables), including most of the basic Unix commnads such as `ls` and `cp`. Most of the programs in `/bin` are in binary format, having been created been by a C compiler, but some are shell scripts in modern systems.
- `/dev` Contains device files.
- `/etc` This core system configuration directory contains the user password, boot, device, networking, and other setup files. Many items in `/etc` are specific to the machine's hardware. For example, the `/etc/X11` directory contains graphics card and window system configurations.
- `/home` Holds personal directories for regular users. Most Unix installations conform to this standard.
- `/lib` An abbreviation for library, this directory holds library files containing code that executables can use. There are ***two*** types of libraries: **static** and **shared**. **The `/lib` directory should contain only shared libraries**, but other lib directories, such as `/usr/lib`, contain both varieties as well as other auxiliary files.
- `/proc` Provides system statistics through a browsable directory-and-file interface. Much of the `/proc` subdirectory structure on Linux is unique, but many other Unix variants have similar features. The `/proc` directory contains information about currently running processes as well as some kernel parameters.
- `/sys` This directory is similar to `/proc` in that it provides a device and system interface.
- `/sbin` The place for system executables. Programs in `/sbin` directories relate to system management, so regular users usually do not have `/sbin` components in their command paths. Many of the utilities found here will not work if you're not running them as root.
- `/tmp` A storage area for smaller, temporary files that you don't care much about. Any user may read to and write from `/tmp`, but the user may not have permission to access another user's files there. Many programs use this directory as a workspace. If something is extremely important, don't put it in `/tmp` because most distributions clear `/tmp` when then machine boots and some even remove its old files periodically. Also don't  let `/tmp` fill up with garbage because its space is usually shared with something critical.
- `/usr` Although pronounced "user", this subdirectory has no user files. Instead, it contains a large directory hierarchy, including the bulk of the Linux system. Many of the directory names in `/usr` are the same as those in the root directory(like `/usr/bin` and `/usr/lib`), and they hold the same type of files.
- `/var` The variable subdirectory, where programs record runtime information. System logging, user tracking, caches, and other files that system programs create and manage are here.

## Other Root Subdirectories

There are a few other interesting subdirectories in the root directory:

- `/boot` Contains kernel boot loader files. These files pertain only to the very first stage of Linux startup procedure;
- `/media` A base attachment point for removable media such as flash drives that is found in many distributions.
- `/opt` This may contain additional third-party software. Many systems don't use `/opt`

## The `/usr` Directory

The `/usr` directory may look relatively clean at first glance, but a quick look at `/usr/bin` and `/usr/lib` reveals that there's a lot here; `/usr` is where most of the user-space programs and data reside. In addition to `/usr/bin`, `/usr/sbin`, and `/usr/lib`, `/usr` contains the following:

- `/include` Holds header files used by C compiler.
- `/info` Contains GNU info manuals.
- `/local` Is where administrators can install their own software. Its structure should look like that of `/` and `/usr`.
- `/man` Contains manual pages.
- `/share` Contains files that should work on other kinds of Unix machines with no loss of functionality. In the past, networks of machines would share this directory, but a true `/share` directory is becoming rare because there are no space issues on modern disk. Maintaining a `/share` directory is often just a pain. In any case, `/man` , `/info`, and some other subdirectories are often found here.

## Kernel Location

On Linux system, the kernel is normally in `/vmlinuz` or `/boot/vmlinuz`. A boot loader loads this file into memory and sets it in motion when the system boots.

Once the boot loader runs and sets the kernel in motion, the main kernel file is no longer used by the running system. However, you'll find many modules that the kernel can load and unload on demand during the course of normal system operation. Called loadable kernel modules, they are located under `/lib/modules`.

## Recommended Reading

[Filesystem Hierarchy Standard](http://www.pathname.com/fhs/)