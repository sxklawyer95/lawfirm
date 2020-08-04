---
title: ArrayList
tags:
  - Data Structures
date: 2019-01-21
catalog: true
---

Now let's talk about arraylist, we know it costs O(n) when you want access a value at the index you give by using a linkedlist, if you want make it faster to finish, you should use arraylist, it cost O(1) when you wanna access the index you want. Because the underlying is an array. So you can access any value at the index you give in one step, just arr[index]. It's pretty cool. Now let me show you what the arraylist look like.

![ArrayList](https://sherlockblaze.com/resources/img/cs/arraylist/arraylist.png)

And in my version, there's still head node here which save the total number of values in the arraylist, the pointer points to the arraylist and the capacity of the arraylist.Let's meet it first.

![Arraylist With HeadNode](https://sherlockblaze.com/resources/img/cs/arraylist/arraylist_with_head_node.png)

***Let's be clear:***

> **Size**: The total number of values saved in the arraylist
> **Capacity**: The capacity of the arraylist, means how many values can be saved in this arraylist now.
> **ArrayList**: The pointer points to the first address of the Array we save values.

Now let's review the basic operations of arraylist.

## Operations

- [Insert](#Insert)
- [Delete](#Delete)

### Insert

If we wanna insert a value into a arraylist at the tail, that'll be easy, just ***arr[L->Size] = Value*** , but if we want to do more about insertion, we should do like this.

In this example, we want to insert value 9 at index 2 of the array.
First step, we should move all the values between index 2 and the end of array one step backward.

![Insert Step1](https://sherlockblaze.com/resources/img/cs/arraylist/insert_step1.png)

Second step, we just need to do like this. `array[2] = 9`

![Insert Step2](https://sherlockblaze.com/resources/img/cs/arraylist/insert_step2.png)

It's finished. Pretty Easy. And it costs O(n). Don't forget to change the value of **Size**, it's supposed to plus one after doing this.

![Insert Successed](https://sherlockblaze.com/resources/img/cs/arraylist/insert_successed.png)

**But what happens when you insert an element when you have insufficient capacity?**

Here is the answer.

First step, we get a new capacity called NewCapacity, and `NewCapacity = OldCapacity / 2 * 3 + OldCapacity` in my version, you can modify it to your perferred value. Then we get a new array with capacity **NewCapacity**.

![Insert Without Enough Room Step1](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_step1.png)

Second step we copy the elements of the old array to the new array in order.

![Insert Without Enough Room Step2](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_step2.png)

Then we can do like inserting with enough room.

![Insert Without Enough Room Step3](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_step3.png)

![Insert Without Enough Room Success](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_successed.png)

Let's take a look at the code.

+ Insert at the tail

```c
void
Insert(List L, ElementType value)
{
    Array array;
    if (L->Size >= L->Capacity)
        IncreaseCapacity(L);
    array = L->ArrayList;
    *(array + L->Size) = value;
    L->Size += 1;
}

+ Insert at given index

```c
void
InsertAt(List L, int index, ElementType value)
{
    Array array;
    if (index > L->Size)
        FatalError("Insert Failed. Illegal index.");
    if (L->Size + 1 > L->Capacity)
        IncreaseCapacity(L);

    array = L->ArrayList;
    if (index == L->Size)
        Insert(L, value);
    else
    {
        MoveTheValuesBackwards(L, index);
        *(array +  index) = value;
    }
    L->Size += 1;
}
```

+ Capacity expansion

```c
void 
IncreaseCapacity(List L)
{
    Array array;
    int Capacity = L->Capacity;
    int NewCapacity = Capacity / 2 * 3 + Capacity;
    array = (ElementType *)malloc(sizeof(ElementType) * NewCapacity);
    if (array == NULL)
        FatalError("No Enough Room!!");
    for (int i = 0; i < L->Size; ++i)
    {
        *(array + i) = *(L->ArrayList + i);
    }
    L->ArrayList = array;
    L->Capacity = NewCapacity;
}
```

### Delete

Then let's talk about deletion of arraylist. Same with insert at the tail, if you wanna delete the value at the tail, it's will be easy too, we just let the **Size** be smaller like this. `L->Size -= 1`. But if you wanna do more?? We should be like this.

First Step. Move all the values between index 2 and the end of array one step forward, like this.

![Delete Step1](https://sherlockblaze.com/resources/img/cs/arraylist/delete_step1.png)

Then we just do like this. `L->Size -= 1`. Finished. Easy too, but it's cost O(n) too;

![Delete Step2](https://sherlockblaze.com/resources/img/cs/arraylist/delete_step2.png)

**Delete Succeed!!!**

![Delete Succeed](https://sherlockblaze.com/resources/img/cs/arraylist/delete_successed.png)

We also take a look at the deletion code too.

```c
void
Delete(List L)
{
    L->Size -= 1;
}

void
DeleteAt(List L, int index)
{
    if (index > L->Size)
        FatalError("Delete Failed. Illegal index.");
    if (index + 1 == L->Size)
        Delete(L);
    else
    {
        MoveTheValuesForWard(L, index);
        L->Size -= 1;
    }
}
```

## Conclusion

And we're supposed to remember the cost of insertion and deletion by using linkedlist, it O(1), now we can see the differences between arraylist and linkedlist.

**Here, we just talk about the operations.**

| Implementation | Delete At Tail | Insert At Tail | Delete At | Insert At | Access At | Access Specified Value |
| --- | --- | --- | --- | --- | --- | --- |
| LinkedList | O(1) | O(1) | O(1) | O(1) | O(n) | O(n) |
| ArrayList | O(1) | O(1) | O(n) | O(n) | O(1) | O(n) |

So, just choose the right way to save your data. And if you wanna read all code about arraylist, try go to my [repo](https://github.com/sherlockblaze/all_knowledge_review/blob/master/Data_Structures/lists/arraylist.h)