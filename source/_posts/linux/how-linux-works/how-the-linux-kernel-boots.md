---
title: How Linux Works(Chapter Five)--How the Linux Kernel Boots
tags:
  - Linux
date: 2019-03-12
---

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

Learn about how the kernel moves into memory up to the point where the first user process starts.

A simplified view of the boot process looks like this:

1. The machine's BIOS or boot firmware loads and runs a boot loader.
2. The boot loader finds the kernel image on desk, loads it into memory, and starts it.
3. The kernel initializes the devices and its drivers
4. The kernel mounts the root filesystem.
5. The kernel starts a program called `init` with a process ID of 1. This point is the ***user space start***.
6. `init` sets the rest of system processes in motion.
7. At some point, `init` starts a process allowing you to log in, usually at the end or near the end of the boot.

## Kernel Initialization and Boot Options

Linux kernel initializes in this general order:

1. CPU inspection
2. Memory inspection
3. Device bus discovery
4. Device discovery
5. Auxiliary kernel subsystem setup(networking, and so on)
6. Root filesystem mount
7. User space start

## Kernel Parameters

When running the Linux kernel, the boot loader passes in a set of text-based kernel parameters that tell the kernel how it should start.

The parameters specify many different types of behavior, such as the amount of diagnostic output the kernel should produce and device driver-specific options.

You can review the kernel parameters from your system's boot by looking at the `/proc/cmdline` file:

```sh
$ cat /proc/cmdline
```

```
BOOT_IMAGE=/boot/vmlinuz-4.18.0-16-generic root=UUID=91ffe6ed-09ff-4355-880d-fc1a4aaac6ba ro quiet splash vt.handoff=1
```

The parameters are either simple one-word flags, such as `ro` and `quiet`, or ***key=value*** pairs, such as ***vt.handoff=7***. Many of the parameters are unimportant, such as the `splash` flag **for displaying a splash screen,** but one that is critical is the `root` parameter. This is the location of the root filesystem; without it, the kernel cannot find `init` and therefore cannot perform the user space start.

The root filesystem can be specified as a device file, such as in this example:

```
root=/dev/sda1
```

However, on most modern desktop systems, a UUID is more common:

```
root=UUID=91ffe6ed-09ff-4355-880d-fc1a4aaac6ba
```

The `ro` parameter is normal; **It instructs the kernel to mount the root filesystem in read-only mode upon user space start.** (Read-only mode ensures that `fsck` can check the root filesystem safely; after the check, the bootup process remounts the root filesystem in read-write mode.)

Upon encountering a parameter that it does not understand, the Linux kernel saves the parameter. The kernel later passes the parameter to `init` when performing the user space start. For example, if you add `-s` to the kernel parameters, the kernel passes the `-s` to the `init` program to indicate that it should start in single-user mode.

Now let's look at the mechanics of how **boot loaders** start the kernel.

## Boot Loaders

**At the start of the boot process, before the kernel and `init` start, a boot loader starts the kernel.**

The task of a boot loader sounds simple: **It loads the kernel into memory, and then starts the kernel with a set of kernel parameters.** But consider the questions that the boot loader must answer:

- Where is the kernel?
- What kernel parameters should be passed to the kernel when it starts?

The answers are (typically) that the kernel and its parameters are usually somewhere on the **root filesystem**. It sounds like the kernel parameters should be easy to find, except that the kernel is not yet running, so it can't traverse a filesystem to find necessary files. Worse, the kernel device drivers normally used to access the disk are also unavailable. Think of this as a kind of "chicken or egg" problem.

Let's start with the driver concern. On PCs, **boot loaders use the Basic Input/Output System(BIOS) or Unified Extensible Firmware Interface(UEFI) to access disks.** ***Nearly all disk hardware has firmware that allows the BIOS to access attached storage hardware with Linear Block Addressing(LBA).*** Although it exhibits poor performance, this mode of access does allow universal access to disks. **Boot loaders are often the only programs to use the BIOS for disk access;** the kernel uses its own high-performance drivers.

Most modern boot loaders can read partition tables and have built-in support for read-only access to filesystems. Thus, they can find and read files. This capability makes it far easier to dynamically configure and enhance the boot loader. Linux boot loaders have not always had this capability; without it, configuring the boot loader was more difficult.

### Boot Loader Tasks

A Linux boot loader's core functionality includes the ability to do the following:

- Select among multiple kernels.
- Switch between sets of kernel parameters.
- Allow the user to manually override and edit kernel image names and parameters.(for example, to enter single-user mode).
- Provide support for booting other operating systems.