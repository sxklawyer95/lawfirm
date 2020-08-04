---
title: Golang GC
tags:
  - Golang
date: 2019-12-13
---

## Basic

**Garbage Collection is the process of freeing memory space that is not being used.**

> The garbage collector sees which objects are out of scope and can no longer be referenced, and it frees the memory space they consume.

**This process happens in a concurrent manner while a Go program is running.**

The documentation of the Go garbage collector implementation states the following:

- **The GC runs concurrently with mutator threads, is type accurate(aka precise), allows multiple GC threads to run in parallel.**
- **It is a concurrent mark and sweep that uses a write barrier.**
- **It is non-generational and non-compacting.**
- **Allocation is done using size segregated per P allocation areas to minimize fragmentation while eliminating locks in the common case.**

```golang
package main

import (
    "fmt"
    "runtime"
    "time"
)

func printStats(mem runtime.MemStats) {
    runtime.ReadMemStats(&mem)
    fmt.Println("mem.Alloc:", mem.Alloc)
    fmt.Println("mem.TotalAlloc:", mem.TotalAlloc)
    fmt.Println("mem.HeapAlloc:", mem.HeapAlloc)
    fmt.Println("mem.NumGC:", mem.NumGC)
    fmt.Println("-------")
}

func main() {
    var mem runtime.MemStats
    printStats(mem)

    for i := 0; i < 10; i++ {
        s := make([]byte, 50000000)
        if s == nil {
            fmt.Println("Operation failed!")
        }
    }
    printStats(mem)

    for i := 0; i < 10; i++ {
        s := make([]byte, 100000000)
        if s == nil {
            fmt.Println("Operation failed!")
        }
        time.Sleep(5 * time.Second)
    }
    printStats(mem)
}
```

> There is a trick that allows you to get even more detailed output about the way the Go garbage collector operates, which is illustrated by the next command:

```sh
$ GODEBUG=gctrace=1 go run main.go
```

## The Tricolor Algorithms

The operation of the Go garbage collector is based on the **tricolor algorithm.**

> The tricolor algorithms is not unique to Go, and it can be used in other programming languages as well.

The official name for the algorithms used in Go is the **tricolor mark-and-sweep algorithm**.

It can work concurrently with the program and uses a **write barrier**.

> This means that when a Go program runs, the Go scheduler is responsible for the scheduling of the application and the garbage collector as if the Go scheduler had to deal with a regular application with multiple **goroutines!**

The Primary principle behind the tricolor mark-and-sweep algorithm is that **it divides the objects of the heap into three different sets according to their color, which is assigned by algorithm.**

- The objects of **black set** are guaranteed to have no pointers to any object of the white set.
- An object in the **white set** can have a pointer to an object of the black, because this has no effect on the operation of the garbage collector
- The objects of the **grey set** might have pointers to some objects of the white set.

**The objects of the white set are candidates for garbage collection.**

- **No object can go directly from the black set to the white set**, which allows the algorithm to operate and be able to clear the objects in the white set.
- **No object of the black set can directly point to an object of the white set.**

Let's go:

1. When the garbage collection begins, **all objects are white** and the **garbage collector visitor all of the root objects and colors them grey.** The roots are the objects that can be directly accessed by the application, **which includes global variables and other things on the stack.**
2. After this, the garbage collector **picks a grey object**, makes it black, and starts searching to determine if that object has pointers to other objects of the white set.
3. When a grey object is being scanned for pointers to other objects, it is colored black. **If that scan discovers that this particular object has one or more pointers to a white object**, it puts that white object in the grey set.

This process keeps going for as long as objects exist in the grey set. **After that, the objects in the white set are unreachable and their memory space can be reused.**

Therefore, at this point, the elements of the white set are said to be garbage collected.

> If an object of the grey set becomes unreachable at some point in a garbage collection cycle, it will not be collected in that garbage collection cycle but rather in the next one!

During this process, the running application is called the **mutator**.

The mutator runs a small function named **write barrier** that is executed each time a pointer in the heap is modified.

**If the pointer of an object in the heap is modified, which means that this object is now reachable, the write barrier colors it grey and puts it in the grey set.**

> The mutator is responsible for the invariant that no element of the black set has a pointer to an element of the white set. This is accomplished with the help of the write barrier function. Failing to accomplish this invariant will ruin the garbage collection process, and it will most likely crash your program in an ugly and undesired way.

![gc-cycle-mastering-go](https://sherlockblaze.com/resources/img/golang/gc-cycle-mastering-go.png)

The Go garbage collection can also be applied to variables such as **channel**. When the garbage collector finds out that a channel is unreachable and that the channel variable can no longer be accessed, it will free its resources even if the channel has not closed.

> Go allows you to initiate a garbage collection manually by putting a `runtime.GC()` statement in your Go code.
Keep in mind that `runtime.GC()` will block the caller, and it might block the entire program, especially if you are running a very busy Go program with many objects. This happens mainly because you cannot perform garbage collections while everything else is rapidly changing, as this will not give the garbage collector the opportunity to identify clearly the members of the white, black, and grey sets!
This garbage collection status is also called the **garbage collection safe-point.**

## More

The main concern of the Go garbage collector is **low latency**, which basically means short pauses in its operation in order to have real-time operation.

What a program does all the time is to create new objects and manipulate existing objects with pointers. This process can end up creating objects that cannot be accessed any longer because no pointers exist that point to these objects.

**Such objects are now garbage waiting for the garbage collector to clean them up and free their memory space.**

The way that the **mark-and-sweep algorithm** works is pretty simple:

- The algorithm stops the program execution(**stop-the-world garbage collector**) in order to visit all of the accessible objects of the heap of a program and marks them.
- It sweeps the inaccessible objects. During the mark phase of the algorithm, each object is marked as white, grey or black.
- The children of a grey object are colored grey, whereas the original grey object is now colored black.

The sweep phase begins when there are no more grey objects to examine. This technique works because **there are no pointers from the black set to the white set, which is a fundamental invariant of the algorithm.**

Go tries to lower that particular latency by running the garbage collector as a concurrent process and using the tricolor algorithm.

Other processes can move pointers or create new objects while the garbage collector runs concurrently. As a result, **the principal point that allows the tricolor algorithm to run concurrently is to be able to maintain the fundamental invariant of the mark-and-sweep algorithm-no object of the black set can point to an object of the white set.**

**But new objects must go to grey set,** because this way the fundamental invariant of the mark-and-sweep algorithm cannot be altered.

Additionally, when a pointer of the program is moved, color the object to which the pointer points as grey. Last, each time a pointer is moved, some Go code gets automatically executed, which is the **write barrier** that does some recoloring.

**The latency introduced by the execution of the write barrier code is the price we have to pay for being able to run the garbage collector concurrently.**

[On-the-fly Garbage Collection: An Exercise in Cooperation](https://sherlockblaze.com/resources/file/golang/on-the-fly-gc.pdf)