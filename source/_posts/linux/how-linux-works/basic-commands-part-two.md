---
title: How Linux Works(Chapter Two)--Basic Commands(Part Two)
tags:
  - Linux
date: 2019-03-02
---

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

## Shell Input and Output

To send the output of `command` to a file instead of the terminal, use the `>` redirection character:

```sh
command > file
```

The shell creates `file` if it does not already exist. If `file` exists, the shell erases the original file first.

You can **append** the output to file instead of overwriting it with the `>>` redirection syntax:

```sh
command >> file
```

This is handy standard output of a command to the standard input of another command, use the pipe character(`|`). To see how this works, try these two commands:

```sh
head /proc/cpuinfo
head /proc/cpuinfo | tr a-z A-Z
```

You can send output through as any piped commands as you wish; just add another pipe before each additional command.

### Standard Error

Occasionally, you may redirect standard output but find that the program still prints something to the terminal. This is called ***standard error***; it's an additional output stream for diagnostics and debugging. For example, this command produces an error:

```sh
ls /fffff > f
```

After completion, `f` should be empty, but you still see the following error message on the terminal as standard error:

```sh
ls: cannot access /fffff: No such file or directory
```

You can redirect the standard error if you like. For example, to send standard output to `f` and standard error to `e`, use the `2>` syntax, like this:

```sh
ls /fffff > f 2> e
```

The number 2 specifies the ***stream ID*** that the shell modifies. **Stream ID 1 is standard output(the default), and 2 is standard error.**

You can also send the standard error to the same place as stdout with the `>&` notation. For example, to send both standard output and standard error to the file named `f`, try this command:

```sh
ls /fffff > f 2>&1
```

### Standard Input Redirection

To channel a file to a program's standard input, use the `<` operator:

```sh
head < /proc/cpuinfo
```

## Understanding Error Messages

When you encounter a problem on a Unix-like system such as Linux, you must read the error message. Unlike messages from other operating systems, Unix errors usually tell you exactly what went wrong.

### Anatomy of a UNIX Error Message

```sh
ls /dsafsd
ls: cannot access  /dsafsd: No such file or directory
```

***There are three components to this message:***

- The program name `ls`. Some programs omit this identifying information, which can be annoying when writing shell scripts, but it's not really a big deal.
- The filename, /dsafsd, which is a more specific piece of information. There's a problem with this path.
- The error `No such file or directory indicates the problem with the filename.`

## Listing and Manipulating Processes

**A process is a running program.** Each process on the system has a numeric ***process ID***(PID). For a quick listing of running processes,just run `ps` on the command line. You should get a list like this one:

```sh
ps
PID TTY STAT TIME COMMAND  
520  p0 S    0:00 -bash  
545   ? S    3:59 /usr/X11R6/bin/ctwm -W  
548   ? S    0:10 xclock -geometry -0-0 
2159  pd SW   0:00 /usr/bin/vi lib/addresses
31956  p3 R    0:00 ps
```

The fields are as follows:

- **PID.** The process ID.
- **TTY.** The terminal device where the process is running.
- **STAT.** The process status, that is, what the process is doing and where its memory resides. For example, `S` means sleeping and `R` means running.
- **TIME.** The amount of CPU time in minutes and seconds that the process has used so far. In other words, the total amount of time that the process has spent running instructions on the processor.
- **COMMAND.** This one might seem obvious, but be aware that a process can change this field from its original value.

### Command Options

The `ps` command has many options.

| command | Function |
| ------- | -------- |
| **ps x** | Show all of your running processes |
| **ps ax** | Show all processes on the system, not just the ones you own. |
| **ps u** | Include more detailed information on processes. |
| **ps w** | Show full command names, not just what fits on one line. |

As with other programs, you can combine options, as in `ps aux` and `ps auxw`. To check on a specific process, add its PID to the argument list of the `ps` command. For example, to inspect the current shell process, you could use `ps u $$`, because `$$` is a shell variable that evaluates to the current shell’s PID.

### Killing Processes

To terminate a process, sent it a `signal` with the `kill` command. A signal is a message to a process from the kernel. When you run `kill`, you're asking the kernel to send a signal to another process.

```sh
kill pid
```

There are many types of signals. The default is `TERM`, or terminate. You send different signals by adding an extra option to `kill`. For example, to freeze a process instead of terminating it, use the `STOP` signal:

```sh
kill -STOP pid
```

A stopped process is stil in memory, ready to pick up where it left off. Use the `CONT` continue running the process again:

```sh
kill -CONT pid
```

**NOTE**: Using `ctrl-c` to terminal a process that is running in the current terminal is the same as using `kill` to end the process with the `INT`(interrupt) signal.

### Job Control

## Recommended Reading

- The Linux Command Line (No Starch Press, 2012)
- UNIX for the Impatient (Addison-Wesley Professional, 1995)
- Learning the UNIX Operating System, 5th edition (O’Reilly, 2001).