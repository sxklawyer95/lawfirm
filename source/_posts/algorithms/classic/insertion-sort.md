---
title: Insertion-Sort
tags:
  - Algorithms
date: 2019-01-24
---

> Know about it by writing CODE.

> ***Because it's really easy!!!!***

## Pseudocode

```
for j = 2 to A.length
	key = A[j]
	// Insert A[j] into the sorted sequence A[1...j-1]
	i = j - 1
	while i > 0 and A[i] > key
		A[i+1] = A[i]
		i = i - 1
	A[i + 1] = key
```

> Know more by reading the complete code:

```c
#ifndef _INSERT_SORT_H_
#define _INSERT_SORT_H_

void insert_sort(int *p, int length);
void insert_sort_test();

#endif /*_INSERT_SORT_H_*/

#include <stdio.h>

void
insert_sort(int *p, int length)
{
	int key = 0;
	// start from the second element
	for (int i = 1; i < length; ++i)
	{
		key = p[i];
		int j = i - 1;
		while (j >= 0 && p[j] > key)
		{
			p[j + 1] = p[j];
			j = j - 1;
		}
		p[j + 1] = key;
	}
}

void
insert_sort_test()
{
	int a[] = {15, 1, 51, 6, 13};
	printf("before sorted!\n");
	for (int i = 0; i < 5; ++i)
		printf("%d\t", a[i]);
	insert_sort(a, 5);
	printf("\nafter sorted\n");
	for (int i = 0; i < 5; ++i)
		printf("%d\t", a[i]);
	printf("\n");
}

```

## Process

In order to help people who don't know about insertion-sort before. I made this. Let's check.

![step1](https://sherlockblaze.com/resources/img/cs/insertion-sort/step1.png)

As we know, we start from the second element. We call it i, the value of i is between 0 and the size of this array or some structure else, the total number of elements. Then we get a key, the value of key is A[i].

![step2](https://sherlockblaze.com/resources/img/cs/insertion-sort/step2.png)

Then we put the second pointer points to the element just before i, we call it j, j start from i - 1, end with 0.

![step3](https://sherlockblaze.com/resources/img/cs/insertion-sort/step3.png)

If the element is pointed by j is bigger than the pointer i points, they exchange their values.

![step4](https://sherlockblaze.com/resources/img/cs/insertion-sort/step4.png)

when j got the end, i plus one, new loop start.

![step5](https://sherlockblaze.com/resources/img/cs/insertion-sort/step5.png)

***And so on....***

![step6](https://sherlockblaze.com/resources/img/cs/insertion-sort/step6.png)

![step7](https://sherlockblaze.com/resources/img/cs/insertion-sort/step7.png)

![step8](https://sherlockblaze.com/resources/img/cs/insertion-sort/step8.png)

![step9](https://sherlockblaze.com/resources/img/cs/insertion-sort/step9.png)

![step10](https://sherlockblaze.com/resources/img/cs/insertion-sort/step10.png)

