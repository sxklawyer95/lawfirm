---
title: How Linux Works(Chapter Three)--Devices
tags:
  - Linux
date: 2019-03-11
---

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

Throughout the history of Linux, there have been many changes to how the kernel presents  devices to the user. We'll begin by looking at the traditional system of device files to see how the kernel provides device configuration information through `sysfs`.

**Goal:** 

1. Be able to extract information about the devices on a system in order to understand a few rudimentary operations.
2. Understand how the kernel interacts with user space when presented with new devices. The `udev` system enables user-space programs to automatically configure and use new devices.

## Device Files

**It's easy to manipulate most devices on a Unix system because the kernel presents many of the device I/O interfaces to user processes as files.**

These device files are sometimes called ***device nodes.***

There's a limit to what you can do with a file interface, so not all devices or device capabilities are accessible with standard file I/O.

Using command `ls -l` and get result like this:

![ls -l](https://sherlockblaze.com/resources/img/profession/linux/how-linux-works/ls-l-command.png)

Note the first character of each line(the first character of the file's mode), if this character is `b`, `c`, `p` or `s`, the file is a device. These letters stand for `block`, `pipe`, and `socket`, respectively.

### Block device

- Program access data from a ***block device*** in fixed chunks, The `sda` in the preceding example is a ***disk device***, a type of block device. Disk can be easily split up into blocks of data. Because a block device's total size is fixed and easy to index, processes have random access to any block in the device with the help of the kernel.

### Character device

- Character devices work with data streams. You can only read characters from or write characters to character devices, as previously demonstrated with `/dev/null`. Character devices don't have a size; **when you read from or write to one, the kernel usually performs a read or write operation on the device.** Printers directly attached to your computer are represented by character devices. **It's important to note that during character device interaction, the kernel cannot back up and reexamine the data stream after it has passed data to a device or process.**

### Pipe device

- Named pipes are like character devices, with another process at the other end of the I/O stream instead of a kernel driver.

### Socket device

- **Sockets** are special-purpose interfaces that are **frequently used for interprocess communication.** They're often found outside of the `/dev` directory. Socket files represent Unix domain sockets.

> Note all devices have device files because the block and character device I/O interfaces are not appropriate in all cases. For example, network interfaces don't have device files. It is theoretically possible to interact with a network interface using a single character device, but because it would be exceptionally difficult, the kernel uses other I/O interfaces.

## The `sysfs` Device Path

The traditional Unix `/dev` directory is a convenient way for user processes to reference and interface with devices supported by the kernel, but it's also a very simplistic scheme. The name of device in `/dev` tells you a little about device, but not a lot. **Another problem is that the kernel assigns devices in the order in which they are found, so a device may have a different name between reboots.**

To provide a uniform view for attached devices based on their actual hardware attributes, the Linux kernel offers the `sysfs` interface through a system of files and directories. The base path for devices is `/sys/devices`.

For example, the SATA hard disk at `/dev/sda` might have the following path in `sysfs`:

![](https://sherlockblaze.com/resources/img/profession/linux/how-linux-works/device-in-sysfs.png)

The `/dev` file is there so that user processes can use the device, whereas the `/sys/devices` path is used to view information and manage the device.

There are a few shortcuts in the `/sys` directory. For example, `/sys/block` should contain all of the block devices available on a system. However, those are just symbolic links; run `ls -l /sys/block` to reveal the true sysfs paths.

It can be difficult to find the sysfs location of a device in `/dev`. Use the `udevadm` command to show the path and other attributes:

```sh
$ udevadm info --query=all --name=/dev/sda
```

## dd and Devices

The program `dd` is extremely useful when working with block and character devices.

**This program's sole function is to read from an input file or stream and write to an output file or stream, possibly doing some encoding conversion on the way.**

`dd` copies data in blocks of a **fixed** size. Here's how to use `dd` with a character device and some common options:

```sh
$ dd if=/dev/zero of=new_file bs=1024 count=1
```

As you can see, the `dd` option format differs from the option formants of most other Unix commands; it's based on an old IBM Job Control Language(JCL) style. Rather than use the dash(-) character to signal an option, you name an option and set its value to something with the equals(=) sign. The preceding example copies a single 1024-byte block from `/dev/zero` to ***new_file***.

**There are the important `dd` options:**

- **if=file** The input file. The default is the standard input.
- **of=file** The output file. The default is the standard output.
- **bs=size** The block size. `dd` reads and writes this many bytes of data at a time. To abbreviate large chunks of data, you can use `b` and `k` to signify 512 and 1024 bytes, respectively. Therefore, the example above could read `bs=1k` instead of `bs=1024`.
- **ibs=size, obs=size** The input and output block sizes. If you can use the same block size for both input and output, use the `bs` option; if not, use `ibs` and `obs` for input and output, respectively.
- **count=num** The total number of blocks to copy. When working with a huge file -- or with a device that supplies an endless stream of data, such as `/dev/zero` -- you want `dd` to stop at a fixed point or you could waste a lot of disk space, CPU time or both. Use `count` with the `skip` parameter to copy a small piece from a large file or device.
- **skip=num** Skip past the first num blocks in the input file or stream and do not copy them to the output.

**WARNING**

> dd is very powerful, so make sure you know what you're doing when you run it. It's very easy to corrupt files and data on devices by making a careless mistake. It often helps to write the output to a new file if you're not sure what it will do.

## Device Name Summary

It can sometimes be difficult to find the name of a device. Here are a few ways to find out what it is in this book:

- Query udevd using `udevadm`
- Look for the device in the `/sys` directory
- Guess the name from the output of the `dmesg` command(which prints the last few kernel messages) or the kernel system log file. This output might contain a description of the devices on your system.
- For a disk device that is already visible to the system, you can check the output of `mount` command.
- Run `cat /proc/devices` to see the block and character devices for which your system currently has drivers. It's the most useful way i think. Each line consists of a number and name. The number is the major number of the device as described in [Device Files](#Device-Files). If you can guess the device from the name, look in `/dev` for the character or block devices with the corresponding major number, and you've found the device files.

> Among these methods, only the first is reliable, but it does require udev. If you get into a situation where udev is not available, try the other methods but keep in mind that the kernel might not have a device file for your hardware.

The following sections list the most common Linux devices and their naming conventions.

### Hard Disks: `/dev/sd*`

Most hard disks attached to current Linux systems correspond in device names with an ***sd*** prefix, such as `/dev/sda`, `/dev/sdb`, and so on. 

These devices represent entire disks; the kernel makes separate device files, such as `/dev/sda1` and `/dev/sdb2` for the partitions on a disk.

The naming convention requires a little explanation. The **sd** portion of the name stands for SCSI disk. **Small Computer System Interface(SCSI)** was originally developed as a hardware and protocol standard for communication between devices such as disks and other peripherals.

Although traditional SCSI hardware isn't used in most modern machines, **the SCSI protocol is everywhere due to its adaptability.** The story on SATA disks is a little more complicated, but the Linux kernel still uses SCSI commands at a certain point when talking to them.

To list the SCSI devices on your system, use a utility that walks the device paths provided by sysfs. One of the most succinct is `lsscsi`, maybe you should install it on your machine first. For example, my system is Ubuntu 18.04, I installed by using command `sudo apt install lsscsi`. Here is what you can expect when you run it:

![](https://sherlockblaze.com/resources/img/profession/linux/how-linux-works/output-of-lsscsi.png)

The first column identifies the address of the device on the system, the second describes what kind of device it is, and the last indicates where to find the device file. Everything else is vendor information.

**Linux assigns devices to device files in the order in which its drivers encounter devices.**

**Notice:**

> Unfortunately, this device assignment scheme has traditionally caused problems when reconfiguring hardware. Say, for example, that you have a system with three disks: `/dev/sda`, `/dev/sdb`, and `/dev/sdc`. If `/dev/sdb` explodes and you mush remove the disk so that the machine can work again, the former `/dev/sdc` moves to `/dev/sdb`, and there is no longer a `/dev/sdc`. If you were referring to the device names directly in the `fstab` file, you'd have to make some changes to that file in order to get things back to normal. To solve this problem, **most modern Linux systems use the Universally Unique Identifier(UUID)** fro persistent disk device access.

### CD and DVD Drives: `/dev/sr*`

Linux recognizes most optical storage drives as the SCSI devices `/dev/sr0`, `/dev/sr1`, and so on. However, if the drive uses an older interface, it might show up as a PATH device , as discussed below. The `/dev/sr*` devices are read only, and they are used only for reading from discs. For the write and rewrite capabilities of optical devices, you’ll use the “generic” SCSI devices such as `/dev/sg0`.

