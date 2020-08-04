---
title: Python3 多线程深入一些
tags:
  - Python
date: 2018-08-16
---

python提供了两个模块，用于完成多线程，_thread以及threading。关于两位的关系，官方文档中是如下的描述的：

> The threading module provides an easier to use and high-level threading API built on top of the _thread module

[_thread](https://docs.python.org/3/library/_thread.html) 是 threading 的底层建筑，threading 是 _thread的高级封装。这里我们主要讲一下 _thread

## [threading](https://docs.python.org/3/library/threading.html#module-threading)

```python
import time
import threading


def task_a():
    print('thread %s is running...' % threading.current_thread().name)
    print('solve task a')
    n = 0
    while n < 5:
        n = n + 1
        print('thread %s >>> %s' % (threading.current_thread().name, n))
        time.sleep(1)
    print('thread %s ended.' % threading.current_thread().name)


def task_b():
    print('thread %s is running...' % threading.current_thread().name)
    print('solve task b')
    n = 0
    while n < 5:
        n = n + 1
        print('thread %s >>> %s' % (threading.current_thread().name, n))
        time.sleep(2)
    print('thread %s ended.' % threading.current_thread().name)


print('thread %s is running...' % threading.current_thread().name)
t1 = threading.Thread(target=task_a, name='taskAThread')
t2 = threading.Thread(target=task_b, name='taskBThread')

t1.start()
t2.start()
t1.join()
print('thread %s ended.' % threading.current_thread().name)
```

> 我们可以看到python中启动一个线程的方法

```python
threading.Thread(target=task_a, name='thread_name')
```

> target是在该线程中执行的代码块，name也就是线程的名字。我们可以通过以下代码获取当前线程的名字。

```python
treading.current_thread().name
```

> 在创建出一个线程后，需要调用start方法来启动它。

以上就是启动一个线程的过程。
我没有对t2调用join方法，是为了想表现出来，即使主线程结束了，子线程还是会继续执行。大家需要分清楚，进程与线程以及线程与线程这两种关系。

## 同步

> 涉及到多线程的问题，就一定会讨论到同步加锁的问题。因为线程和进程不一样，进程中所有的资源都是复制了独一份，自己独享，线程生存在进程之下，进程中的资源在多线程中是被共享的。所以需要被同步，使用加锁的方式可以解决。

```python
import threading
import time

count = 0
lock = threading.Lock()


def add(n):
    global count
    count += n
    time.sleep(2)
    count -= n
    print(count)


threading.Thread(target=add, name='ThreadA', args=(1,)).start()
threading.Thread(target=add, name='ThreadB', args=(2,)).start()
在这里，当执行次数足够多的时候，打印出来的count值可能不是0，因为现代计算机代码执行速度较快，以上代码是很难看出效果的。但是上面的代码看出问题的时间可能会比较长，我们可以使用个小技巧，来加快暴露问题的速度。
import threading
import time

count = 0


def add(n):
    global count
    count += n
    time.sleep(2)
    count -= n
    print(count)
        
threading.Thread(target=add, name='ThreadA', args=(1,)).start()
threading.Thread(target=add, name='ThreadB', args=(2,)),start()
```

因为add方法是未同步的，所以导致问题的抛出，为了避免这个问题，我们需要对此这段代码加锁。加锁的方式很简单。

```python
import threading
import time

count = 0
lock = thrading.Lock()

def add(n):
    global count
    lock.acquire()
    try:
        count += n
        time.sleep(2)
        count -= n
        print(count)
    finally:
        lock.release()

threading.Thread(target=add, name='ThreadA', args=(1,)).start()
threading.Thread(target=add, name='ThreadB', args=(2,)),start()
```
通过上面的代码，我们可以很清楚的看到加锁的方式。

## GIL(global interpreter lock)

> 在Python中，有一个称为GIL的东西，在每个线程执行前，必须要获取GIL锁，然后执行100个字节后，再自动释放，然后让别的线程得到执行。
这样做就涉及到一个问题，现在的计算机大多为多核，这种情况下，Python多线程领域，多核的优势当然无存，因为GIL锁的存在，使得多核无法被充分利用。就算是10核的计算机，有10个线程在跑，它实际上也只相当于在一个核心上运行。因为线程必须要获得GIL锁才能够运行。
当然，Python中如果想利用到多核心，可以通过多进程的方式来控制，每个进程都一个单独的GIL锁。

## ThreadLocal的使用

使用多线程时，我们一定要考虑线程安全性问题。使用全局变量肯定是没有使用线程独有的局部变量来的好，使用全局变量意味着需要加锁、同步，这样必然会导致性能的下降。但是如果使用局部变量，也会有一定的问题，比如有一个线程，按照这样的顺序执行：A->B->C->D。意思是：步骤A到步骤B然后类推。这个时候，如果有个变量s，需要被每个步骤使用，我们就需要不断的做传递，这样一层层的传，一是，增加我们的调用栈内容，二是代码上也不够美观。

我们可以通过一个全局的dict，然后用线程的唯一标识作为dict的key，绑定对应的变量，这样会更加方便一些。

```python
import threading

dict = {}

def init():
    man = Man()
    dict[threading.current_thread()] = man
    doA()
    doB()
    
def doA():
    man = dict[threading.current_thread()]
    ...

def doB():
    ...
```

> 通过上述代码，我们声明了一个全局字典，然后将与线程对应的数据绑定起来，存放到dict中，后续再使用时，根据相应的条件直接获取即可。

这样的设计是绝对可行的，是个很好的方法，就是每次都要从dict中获取相应数据，在代码上不够美观，**于是一个利用了这种方式的更高级的表示方式出现了：ThreadLocal**

我们来看上面的例子用ThreadLocal是怎么来写的：

```python
import threading

global_var = threading.local()

def init(man, woman):
    global_val.man = man
    global_val.woman = woman
    doA()
    
def doA():
    man = global_val.man
    woman = global_val.woman
    man.run()
    woman.run()
    
...
```

> 通过上述代码中的 global_var = threading.local() 我们成功声明了一个ThreadLocal变量。后续的使用就很清晰了。

那么与使用dict，ThreadLocal有什么先进之处呢？

> 由于dict中key值是唯一的，所以一个线程仅仅只能存放一个线程独享的变量，但是ThreadLocal不一样，它可以存放很多个，如上述代码所示，存放了man,woman。