---
title: How Linux Works(Chapter Two)--Basic Commands(Part One)
tags:
  - Linux
date: 2019-02-28
---

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

## The Bourne Shell

The shell is one of the most important parts of a Unix system. A `shell` is a program that runs commands, like the ones that users enter. The shell also serves as a small programming environment. **Unix programmers often break common tasks into little components and use the shell to manage tasks and piece things together.**

One of the best things about the shell is that if you make a mistake, you can easily see what you typed to find out what went wrong, and then try again.

> Linux uses an enhanced version of the Bourne shell called `bash` or the "Bourne-again" shell. The `bash` shell is the default shell on most Linux distributions, and `/bin/sh` is normally a link to `bash` on a Linux system. You should use `bash` shell when running the examples I copy from the [book](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1).

## Using the Shell

When you install Linux, you should create at least one regular user in addition to the root user; this will be your personal account.

### cat

The `cat` command is one of the easiest Unix commands to understand, it simply outputs the contents of one or more files. The general syntax of the `cat` command is as follows:

```sh
cat file1 file2
```

When you run this command, `cat` prints the contents of `file1`, `file2`, and any other files that you specify, and then exits. The command is called `cat` because it performs concatenation when it prints the contents of more than one file.

## Standard Input and Standard Output

We'll use `cat` to briefly explore Unix input and output(I/O). **Unix processes use I/O `stream` to read and write data. Processes read data from input streams and write data to output streams.** Streams are very flexible. For example, the source of an input stream can be a file, a device, a terminal, or even the output stream from another process.

***Standard output*** is similar. The kernel gives each process a standard output stream where it can write its output. The `cat` command always writes its output to the standard output. When you ran `cat` in the terminal, the standard output was connected to that terminal, so that's where you saw the output.

Standard input and output are often abbreviated as `stdin` and `stdout`. Many commands operate as `cat` does; if you don't specify an input file, the command reads from `stdin`. Output is a little different. Some commands (like `cat`) send output only to stdout, but others have the option to send output directly to files.

**And there is a third standard I/O stream called standard `error`.**

One of the best features of standard streams is that you can easily manipulate them read and write to places other than the terminal.

## Basic Commands

Now let's look at some more Unix commands.

#### `ls`

The ls command lists the contents of a directory. The default is the current directory. Use `ls -l` for a detailed (long) listing and `ls -F` to display file type information.

#### `cp`

In its simplest form `cp` copies files. For example, to copy ***file1*** to ***file2***, enter this:

```sh
cp file1 file2
```

To copy a number of files to a directory (folder) named ***dir***, try this instead:

```sh
cp file1 ... fileN dir
```

#### `mv`

The `mv` (move) command is like `cp`. In its simplest form, it renames a file. For example, to rename ***file1*** to ***file2***, enter this:

```sh
mv file1 file2
```

You can use `mv` to move a number of files to a different directory:

```sh
mv file1 ... fileN dir
```

#### `touch`

The `touch` command creates a file. If the file already exists, `touch` does not change it, but it does update the file's modification time stamp printed with the `ls -l` command. For example, to create empty file, enter this:

```sh
touch file
```

#### `rm`

To delete(remove) a file, use `rm`. After you remove a file, it's gone from your system and generally cannot be undeleted.

```sh
rm file
```

#### `echo`

The `echo` command prints its arguments to the standard output:

```sh
echo Hello again.
Hello again.
```

The `echo` command is very useful for finding expansions of shell globs and variables.

## Navigating Directories

Unix has a directory hierarchy that starts at `/`, sometimes called the `root` directory. The directory separator is the slash (`/`), not the backslash (`\`). There are several standard subdirectories in the `root` directory, such as `/usr`.

When you refer to a file or directory, you specify a `path` or `pathname`. When a path starts with `/` (such as `/usr/lib`), it's a full or absolute path.

A path component identified by two dot (..) specifies the parent of a directory. For example, if you're working in `/usr/lib`, the path `..` would refer to `/usr`. Similarly, `../bin` would refer to `/usr/bin`.

One dot(.) refers to the current directory; for example, if you're in `/usr/lib`, the path `.` is still `/usr/lib`, and `./X11` is `/usr/lib/X11`. You won't have to use `.` very often because most commands default to the current directory if a path doesn't start with `/` (you could just use `X11` instead of `./X11` in the preceding example).

A path not beginning with `/` is called a `relative path`. Most of the time, you'll work with relative pathnames, because you'll already be in the directory you need to be in or somewhere close by.

#### `cd`

The current working directory is the directory that a process(such as the shell) is currently in. The `cd` command changes the shell's current working directory:

```sh
cd dir
```

If you omit ***dir***, the shell returns to your home directory, the directory you started in when you first logged in.

#### `mkdir`

The `mkdir` command creates a new directory ***dir***.

```sh
mkdir dir
```

#### `rmdir`

The `rmdir` command removes the directory ***dir***.

```sh
rmdir dir
```

If ***dir*** isn't empty, this command fails. However, if you're impatient, you probably don't want to laboriously delete all the files and subdirectories inside `dir` first. You can use `rm -rf dir` to delete a directory and its contents, but be careful! Double-check your command before you run it.

## Recommended Reading

- The Linux Command Line (No Starch Press, 2012)
- UNIX for the Impatient (Addison-Wesley Professional, 1995)
- Learning the UNIX Operating System, 5th edition (O’Reilly, 2001).