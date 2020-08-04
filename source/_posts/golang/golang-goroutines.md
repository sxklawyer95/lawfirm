---
title: Goroutines
tags:
  - Golang
date: 2019-12-04
---

## Goroutines

Many concepts that were previously considered academic and inefficient are beginning to find a place among modern software solutions. Two such concepts are coroutines(goroutines in Go) and channels.

Conceptually, they have evolved over time and they have been implemented differently in each programming language.

In many programming languages such as Ruby or Clojure, they are implemented as libraries, but in Go, **they are implemented within the language as a native feature.**

- **Concurrency and parallelism**
- **Go's runtime scheduler**
- **Gotchas when using goroutines**

### Concurrency and Parallelism

Computer and software programs are useful because they do a lot of laborious work very fast and can also do multiple things at once.

We want our programs to be able to do multiple things simultaneously, that is, multitask, and the success of a programming language can depend on how easy it's to write and understand multitasking programs.

> [concurrency is not paralleism](https://blog.golang.org/concurrency-is-not-parallelism)

- **Concurrency**: ***Concurrency is about dealing with lots of things at once.*** This means that we manage to get multiple things done at once in a give period of time. However, we will only be doing a single thing at a time. This tends to happen in programs where one task is waiting and the program decides to run another task in the idle time. 
- **Parallelism**: ***Parallelism is about doing lots of things at once.*** This means that even if we have two tasks, they are continuously working without any breaks in between them.

![concurrency-and-parallelism](https://sherlockblaze.com/resources/img/daily/2019-12-04/concurrency-and-parallelism.png)