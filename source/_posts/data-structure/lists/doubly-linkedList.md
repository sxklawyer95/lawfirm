---
title: Doubly LinkedList
tags:
  - Data Structures
date: 2019-01-21
---

Sometimes it's convenient to traverse lists backwards. We just add an extra field to the data structure, containing a pointer to the previous node.

Here is what Doubly LinkedList looks like.

![Doubly LinkedList](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/doublylinkedlist.png)

In my version, there's still has a head node of the list, and it's value equals the total number of valid nodes.

![Doubly LinkedList With Head Node](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/doublylinkedlist_with_head.png)

Now we can see the basic operations of doubly linkedlist.

## Operations

- [Insert](#Insert)
- [Delete](#Delete)

### Insert

![Insert Step1](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/insert_step1.png)

First Step, we get a new node called NewNode, and get it ready for insertion. Then we let the Next Pointer of NewNode equals the Next pointer of previous node.

![Insert Step2](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/insert_step2.png)

Then we need to let the Previous pointer of the node that the Next pointer of previous node points to points to the NewNode.

![Insert Step3](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/insert_step3.png)

Let the Next pointer of Previous node points to NewNode

![Insert Step4](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/insert_step4.png)

Now we can let the Previous pointer of NewNode points to the Previous node.

**And then we finished the insertion. Insert at the end of the list is similar to this.**

![Insert Succeed](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/insert_successed.png)

### Delete

First, we call the node we want to delete TargetNode.

![Delete Step1](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/delete_step1.png)

As the picture shows, we should let the previous pointer of the next node of the TargetNode points to the previous node of the TargetNode.

![Delete Step2](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/delete_step2.png)

Then we let the Next pointer of previous node points to the next node of the TargetNode.

**Finished!! Just don't forget to free the space of TargetNode.**

![Delete Succeed](https://sherlockblaze.com/resources/img/cs/doublylinkedlist/delete_successed.png)

## Conclusion

The cost of insertion or deletion still O(1).
It's just as same as [linkedlist](https//sherlockblaze.com/2019/01/20/LinkedList) -- The standard implementation.