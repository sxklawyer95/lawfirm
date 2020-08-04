---
title: Golang Concurrent
tags:
  - Golang
date: 2019-12-19
---

## Concurrency and Parallelism

Computer and software programs are useful because they do a lot of laborious work very fast and can also do multiple things at once.

We want our programs to be able to do multiple things simultaneously, that is, multitask, and the success of a programming language can depend on how easy it's to write and understand multitasking programs.

> [concurrency is not paralleism](https://blog.golang.org/concurrency-is-not-parallelism)

- **Concurrency**: ***Concurrency is about dealing with lots of things at once.*** This means that we manage to get multiple things done at once in a give period of time. However, we will only be doing a single thing at a time. This tends to happen in programs where one task is waiting and the program decides to run another task in the idle time. 
- **Parallelism**: ***Parallelism is about doing lots of things at once.*** This means that even if we have two tasks, they are continuously working without any breaks in between them.

![concurrency-and-parallelism](https://sherlockblaze.com/resources/img/daily/2019-12-04/concurrency-and-parallelism.png)

**Let's first take a look at how concurrency became such an important topic.**

## Moore's Law

**The number of components on an integrated circuit would double every two years.** This prediction more on less held true until just recently -- around 2012.

Several companies foresaw this slowdown in the rate Moore's law predicted and began to investigate alternative ways to increase computing power.

**Necessity is the mother of innovation.**

So, multicore processors were born.

## Amdahl's Law

Amdahl's law describes a way in which to model the potential performance gains from implementing the solution to a problem in a parallel manner.

Simply put, **it states that the gains are bounded by how much of the program must be written in a sequential manner.**

> For example, imagine you were writing a program that was largely GUI based: a user is presented with an interface, clicks on some buttons, and stuff happens. This type of program is bounded by one very large sequential portion of the pipeline: human interaction. No matter how many cores you make available to this program, **it will always be bounded by how quickly the user can interact with the interface.**

> Now consider a different example, calculating digits of pi. Thanks to a class of algorithms called [spigot algorithms](https://en.wikipedia.org/wiki/Spigot_algorithm), this problem is called ***embarrassingly parallel***, which -- despite sounding made up -- is a technical term which means that it can easily be divided into parallel tasks. In this case, significant gains can be made by making more cores available to your program, and **your new problem becomes how to combine and store the results.**

**Amdahl's law helps us understand the difference between there two problems, and can help us decide whether parallelization is the right way to address performance concerns in our system.**

## Why is Concurrency Hard?

[The free lunch is over: A fundamental turn toward concurrency in software](http://www.gotw.ca/publications/concurrency-ddj.htm)

**"We desperately need a higher-level programming model for concurrency than languages offer today."**

Concurrent code is notoriously difficult to get right. **Fortunately everyone runs into the same issues when working with concurrent code.** Because of this, computer scientists have been able to label the common issues, which allows us to discuss how they arise, why, and how to solve them.

### Race Conditions

**A race condition occurs when two or more operations must execute in the correct order, but the program has not been written so that this order is guaranteed to be maintained.**

### Atomicity

**When something is considered atomic, or to have the property of atomicity, this means that within the context that it is operating, it's indivisible, or uninterruptible.**

**Something may be atomic in one context, but not another.**

> Operations that are atomic within the context of your process may not be atomic in the context of the operating system; operations that are atomic within the context of the operating system may not be atomic within the context of your machine; and operations that are atomic within the context of your machine may not be atomic within the context of your application.

**When thinking about atomicity, very often the first thing you need to do is to define the context, or scope, the operation will be considered to be atomic in.**

Let's look at an example:

```cpp
i++
```

It may took atomic, but a brief analysis reveals several operations:

- Retrieve the value of i
- Increment the value of i
- Store the value of i

### Memory Access Synchronization

Two concurrent processes are attempting to access the same area of memory, and the way they are accessing the memory is not atomic.

```golang
var data int
go func() {data++}()
if data == 0 {
    fmt.Println("the value is 0.")
} else {
    fmt.Println("the value is %v.\n", data)
}
```

### Deadlocks

**A deadlocked program is one in which all concurrent processes are waiting on one another.** In this state, the program will never recover without outside intervention.

```golang
type value struct {
    mu    sync.Mutex
    value int
}

var wg sync.WaitGroup
printSum := func(v1, v2 *value) {
    defer wg.Done()
    v1.mu.Lock()
    defer v1.mu.Unlock()

    time.Sleep(2 * time.Second)
    v2.mu.Lock()
    defer v2.mu.Unlock()

    fmt.Printf("sum=%v\n", v1.value + v2.value)
}

var a, b value
wg.Add(2)
go printSum(&a, &b)
go printSum(&b, &a)

wg.Wait()
```

It turns out there are a few conditions that must be present for deadlocks to arise, Edgar Coffman enumerated there conditions in this paper:[System Deadlock](https://sherlockblaze.com/resources/file/golang/system-deadlock.pdf). The conditions are now known as the Coffman Conditions and are the basis for techniques that help detect, prevent, and correct deadlocks.

The Coffman Conditions are as follows:

- **Mutual Exclusion**: A concurrent process holds exclusive rights to a resource at any one time.
- **Wait For Condition**: A concurrent process must simultaneously hold a resource and be waiting for an additional resource.
- **No Preemption**: A resource held by a concurrent process can only be released by that process, so it fulfills this condition.
- **Circular Wait**: A concurrent process(P1) must be waiting on a chain of other concurrent processes(P2), which are in turn waiting on it(P1), so it fulfills this final condition too.

### LiveLock

**Livelocks are programs that are actively performing concurrent operations, but these operations do nothing to move the state of the program forward.**

```golang
cadence := sync.NewCond(&sync.Mutex{})
go func() {
    for range time.Tick(1 * time.Millisecond) {
        cadence.Broadcase()
    }
}()

takeStep := func() {
    cadence.L.Lock()
    cadence.Wait()
    cadence.L.Unlock()
}

tryDir := func(dirName string, dir *int32, out *bytes.Buffer) bool {
    fmt.Fprintf(out, " %v", dirName)
    atomic.AddInt32(dir, 1)
    takeStep()
    if atomic.LoadInt32(dir) == 1 {
        fmt.Fprint(out, ". Success!")
        return true
    }
    takeStep()
    atomic.AddInt32(dir, -1)
    return false
}

var left, right int32
tryLeft := func(out *bytes.Buffer) bool {return tryDir("left", &left, out)}
tryRight := func(out *bytes.Buffer) bool {return tryDir("right", &right, out)}

walk := func(walking *sync.WaitGroup, name string) {
    var out bytes.Buffer
    defer func() {fmt.Println(out.String())}()
    defer walking.Done()
    fmt.Fprintf(&out, "%v is trying to scoot:", name)
    for i := 0; i < 5; i++ {
        if tryLeft(&out) || tryRight(&out) {
            return
        }
    }
    fmt.Fprintf(&out, "\n%v tosses her hands up in exasperation!", name)
}

var peopleInHallway sync.WaitGroup
peopleInHallway.Add(2)
go walk(&peopleInHallway, "Alice")
go walk(&peopleInHallway, "Barbara")
peopleInHallway.Wait()
```

This example demonstrates a very common reason livelocks are written: two or more concurrent processes attempting to prevent a deadlock without coordination.

### Starvation

**Starvation is any situation where a concurrent process cannot get all the resources it needs to perform work.**

In a livelock, all the concurrent processes are starved equally, and **no work** is accomplished. More broadly, starvation usually implies that there are one or more greedy concurrent process that are unfairly preventing one or more concurrent processes from accomplishing work as efficiently as possible, or maybe at all.

```golang
var wg sync.WaitGroup
var sharedLock sync.Mutex
const runtime = 1 * time.Second

greedyWorker := func() {
    defer wg.Done()

    var count int
    for begin := time.Now(); time.Since(begin) <= runtime {
        sharedLock.Lock()
        time.Sleep(3 * time.Nanosecond)
        sharedLock.Unlock()
        count++
    }

    fmt.Printf("Greedy worker was able to execute %v work loops\n", count)
}

politeWorker := func() {
    defer wg.Done()

    var count int
    for begin := time.Now(); time.Since(begin) <= runtime; {
        sharedLock.Lock()
        time.Sleep(1 * time.Nanosecond)
        sharedLock.Unlock()

        sharedLock.Lock()
        time.Sleep(1 * time.Nanosecond)
        sharedLock.Unlock()

        sharedLock.Lock()
        time.Sleep(1 * time.Nanosecond)
        sharedLock.Unlock()

        count++
    }

    fmt.Printf("Polite worker was able to execute %v work loops.\n", count)
}

wg.Add(2)
go greedyWorker()
go politeWorker()

wg.Wait()
```

Note our technique here for identifying the starvation: a metric. Starvation makes for good argument for recording and sampling metrics. One of the ways you can detect and solve starvation is by logging when work is accomplished, and then determining if your rate of work is as high as you expect it.

If you utilize memory access synchronization, you'll have to find a balance between preferring coarse-grained synchronization for performance, and fine-grained synchronization for fairness.

**When it comes time to performance tune your application, to start with, I highly recommend you constrain memory access synchronization only to critical sections; if the synchronization becomes a performance problem, you can always broaden the scope.**