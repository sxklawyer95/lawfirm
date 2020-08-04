---
title: Queue
tags:
  - Data Structures
date: 2019-01-21
---

Extracted from the book. **Like stacks, queue are lists. With a queue, however, insertion is done at one end, whereas deletion is performed at the other end.** Just like people are waiting in line.

Still from books. **The basic operations on a queue are _Enqueue_, which inserts and element at the end of the list(called the rear),and _Dequeue_, which deletes (and returns) the element at the start of the list(known as the front).**

It's look like this:

![Queue](https://sherlockblaze.com/resources/img/cs/queue/queue.png)

## Operations

We already know about queue and it's basic operations above. That's pretty easy. So we are not going to talk more here.

We know stack run like **LIFO**(last in first out), queue is different, it's first in first out -- **FIFO**, just like people are waiting in line, remember?

And In my version, because the **Dequeue** operation is performed at the **front** of the queue , if we use arraylist, it cost O(n), we don't want it happen. So, we choose linkedlist to implement it, but the **Enqueue** operation is performed at the **rear** of the queue. If we just use a simple linkedlist, it's will cost O(n) to find the node at the end, so we modify the linkedlist, create a pointer points to the end of the linkedlist. we do it at [here](../../lists/linkedlist_with_tail_pointer.h).

## Conclusion

Now let's take a look at the complete code:

```c
#ifndef _QUEUE_H_
#define _QUEUE_H_

#include "./linkedlist_with_tail_pointer.h"

typedef List Queue;
List NewQueue();
void Enqueue(Queue Queue, ElementType Value);
ElementType Dequeue(Queue Queue);

#endif /*_QUEUE_H_*/

#include <stdio.h>

Queue
NewQueue()
{
    return NewList();
}

void
Enqueue(Queue Queue, ElementType Value)
{
    Insert(Queue, Value);
}

ElementType
Dequeue(Queue Queue)
{
    ElementType Value;
    Value = Queue->Head->Value;
    DeleteAt(Queue, 0);
    return Value;
}
```