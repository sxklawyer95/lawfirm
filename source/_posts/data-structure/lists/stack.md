---
title: Stack
tags:
  - Data Structures
date: 2019-01-21
---

A stack is a list with the restriction that insertions and deletions can be performed in only one position. We call this behavior last in first out(**LIFO**), and the position, we call it **top**, top of a stack.

Those Pictures shows a stack can be.

It can be ...

![ArrayList Stack](https://sherlockblaze.com/resources/img/cs/stack/stack_1.png)

or It can be ...

![LinkedList Stack](https://sherlockblaze.com/resources/img/cs/stack/stack_2.png)

As we can see, The most recently inserted element can be examined prior to performing a Pop by use of Top routine.

## Operation

- [Push](#Push)
- [Pop](#Pop)
- [Implementation](#Implemention)

### Push

It easy to understand how to push a value into a stack.

![Push](https://sherlockblaze.com/resources/img/cs/stack/stack_push.png)

### Pop

It also easy to understand how to pop a value into a stack.

![Pop](https://sherlockblaze.com/resources/img/cs/stack/stack_pop.png)

## Implementation

Actually, stack still is a list, whatever a linkelist or arraylist, all of them can be the data structure of a stack.
In my version, I use the linkedlist as the data structure of it.

![My Stack](https://sherlockblaze.com/resources/img/cs/stack/my_stack.png)

For better push of element, I make the head node points to the **Top**, and the last node of this linkedlist called **Bottom**. If we always insert or delete node at the index 0, we can get a stack.

![More about My Stack](https://sherlockblaze.com/resources/img/cs/stack/my_stack_more.png)

