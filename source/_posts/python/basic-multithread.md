---
title: Python3 多线程基础
tags:
  - Python
date: 2018-08-16
---

## Linux/Unix下的多进程

```python
import os
Python多进程
print('1\t%d' % os.getpid())

pid = os.fork()
if pid == 0:
    print('2\ti\'m son %d, my father is %d' % (os.getpid(), os.getppid()))
else:
    print('3\ti %d just create a son %d.' % (os.getpid(), pid))

print('4\t%d' % os.getppid())
```

> 我们通过上述代码的输出来讨论这个问题：

```sh
1   23714
3   i 23714 just create a son 23721.
4   5408
2   i'm son 23721, my father is 23714
4   23714
```

> 通过上述代码和对应输出，我们做一下总结：

首先我们了解一下fork()，fork是unix/linux系统提供的一种系统调用，但是windows没有，也就是说**上述代码在windows下没有效果。**
接下来讨论一下fork做的工作，fork把父进程复制了一次，生成一个子进程，然后在父进程和子进程中分别返回一次，也就是返回两次。

根据上面的输出结果，我们可以看到，进程23714调用了一次os.fork，这个时候父进程收到返回，继续向下走，走到输出3的位置，并打印出刚刚创建的子进程的pid，在走到4，输出父进程是5408

同时，子进程也做了一次返回，这时候按照子进程的流程往下走，会输出2的语句，打印出父进程，可以看到，是23714。
到这里，以上的情形也就大致清楚了。

其中父进程会记录子进程的pid

## 跨平台的多进程

我们已经知道，通过os.fork是没法在windows下进行多进程的，所幸，Python提供了可跨平台的多进程启动方式，通过multiprocess中的Process我们可以轻松启动一个子进程。

```python
from multiprocessing import Process


def task(name):
    print('task %s' % name)


def mission(*args):
    for i in args:
        print('task %s' % i)


def no_task():
    print('just for fun')


if __name__ == '__main__':
    p1 = Process(target=task, args=('A',))
    p2 = Process(target=mission, args=('B', 'C'))
    p3 = Process(target=no_task)
    p1.start()
    p2.start()
    p3.start()
    p1.join()
    p2.join()
    p3.join()
    print('Over')
```

> 上述代码，我们可以看到创建一个进程的方式。

```python
p1 = Process(target=task, args('A'))
```

> 通过提供要启动的方法，以及传递的参数，再调用start()方法，就可以轻松启动一个进程。

## 进程池

有时候，我们需要对进程的启用有所限制，不然不断地增加进程，会逐渐消耗完系统的资源。这个时候，我们需要使用线程池，来对线程的启用做一些限制。

```python
from multiprocessing import Pool
import os
import time
import random


def long_time_task(name):
    print('Run task %s (%s)...' % (name, os.getpid()))
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print('Task %s runs %0.2f seconds.' % (name, (end - start)))


if __name__ == '__main__':
    print('Parent process %s.' % os.getpid())
    p = Pool(4)
    for i in range(5):
        p.apply_async(long_time_task, args=(i,))
    print('Waiting for all subprocesses done...')
    p.close()
    p.join()
    print('All subprocesses done.')
```

> 代码输出如下：

```sh
Parent process 7240.
Waiting for all subprocesses done...
Run task 0 (7247)...
Run task 1 (7248)...
Run task 2 (7249)...
Run task 3 (7250)...
Task 3 runs 0.11 seconds.
Run task 4 (7250)...
Task 4 runs 0.74 seconds.
Task 0 runs 1.11 seconds.
Task 1 runs 2.14 seconds.
Task 2 runs 2.25 seconds.
All subprocesses done.
```

> 我们可以看到在执行任务0、1、2、3的时候没有提示任务4的执行，因为我们的进程池最多只能容纳4个进程执行。

```python
p = Pool(4)
```

> 所以只有当有进程执行完成返回后，才有机会载入执行。

其中，**Pool()的默认值是计算机的核数**

## 进程中通信

进程之间是需要通信的，python的multiprocessing封装了一些系统底层的东西，并且提供了Pipe/Queue多种方式来帮助交换数据。

***Waiting...***