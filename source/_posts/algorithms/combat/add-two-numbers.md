---
title: Add Two Numbers
tags:
  - Algorithms
date: 2019-01-24
---

## Question

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Example:

Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807

## Code

> Now we can use our data structures to solve some algorithms problems, view [LinkedList](https://sherlockblaze.com/2019/01/21/computer_science/data_structures/LinkedList/). This one is the first. Here's code(View All Code About Notes [Here](https://github.com/sherlockblaze/all_knowledge_review)):

```c
#include "../../data_structures/lists/linkedlist.h"

#ifndef _ADD_TWO_NUMBERS_H_
#define _ADD_TWO_NUMBERS_H_

#define ElementType int

List addTwoNumbers(List l1, List l2);
void TraverseList(List L);
void test();

#endif /*_ADD_TWO_NUMBERS_H_*/


List
addTwoNumbers(List l1, List l2)
{
	int overflow = 0;
	List result = NewList();
	Position l1_node, l2_node;

	if (l1 != NULL)
		l1_node = l1->Next;
	if (l2 != NULL)
		l2_node = l2->Next;

	while (1)
	{
		if (l1_node != NULL)
		{
			overflow += l1_node->Value;
			l1_node = l1_node->Next;
		}
		if (l2_node != NULL)
		{
			overflow += l2_node->Value;
			l2_node = l2_node->Next;
		}
		InsertAt(result, 0, overflow % 10);
		overflow /= 10;

		if (l1_node == NULL && l2_node == NULL && overflow == 0)
			break;
	}
	return result;
}

void
TraverseList(List L)
{
	if (L == NULL)
		FatalError("No such linkedlist");
	printf("There're %d elements:\n", L->Value);
	Position TmpPointer = L->Next;
	while(TmpPointer != NULL)
	{
		printf("%d\t", TmpPointer->Value);
		TmpPointer = TmpPointer->Next;
	}
	printf("\n");
}

void
test()
{
	List l1 = NewList();
	List l2 = NewList();
	int num1[] = {5};
	int num2[] = {5};
	InsertArray(l1, num1, 1);
	InsertArray(l2, num2, 1);
	TraverseList(l1);
	TraverseList(l2);
	List result = addTwoNumbers(l1, l2);
	TraverseList(result);
	DeleteList(l1);
	DeleteList(l2);
	DeleteList(result);
}
```