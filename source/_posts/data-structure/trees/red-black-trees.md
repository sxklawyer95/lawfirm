---
title: Red-Black Trees
tags:
  - Data Structures
date: 2019-05-05
catalog: true
---

## What's it for?

We got a binary search tree, and we know that a binary search tree of height ***h*** can support any of the basic dynamic-set operations - such as **Search**, **Predecessor** -- in ***O(h)*** time. ***The set operations are fast if the height of the search tree is small.*** So! **If its height is large, the set operations may run no faster than a linked list.**

## What's it?

### Definition

Red-Black trees are one of many search-tree schemes that are "balanced" in order to guarantee that the basic dynamic-set operations take **O(lgN)** time in the worst case.

So, it's a better **binary search tree**.

### Basic Concept

> ***Q: What's kind of binary search tree is a Red-Black Tree?***

1. **Every node is either red or black**
2. **The root is black**
3. **Every leaf is black**
4. **If a node is red, then both its children are black**
5. **For each node, all simple paths from the node to descendant leaves contains the same number of black nodes.**

**Black-Height**: The number of black nodes on any simple path from, but not including, a node x down to a leaf.

### Operations

#### Rotations

##### What's it for?

**The operations Insert and Delete modify the tree, the result may violate the red-black properties. We need to do something to restore these properties.**

##### What's it?

**It's a local operation in a search tree that preserves the binary-search-tree property.**

##### Operations

We show two kinds of rotations here: **Left-Rotate** and **Right-Rotate**.

![](https://sherlockblaze.com/resources/img/cs/trees/basic_rotations.png)

> The above two node states are converted by **Left-Rotation** and **Right-Rotation**.

##### Precedes

+ Left-Rotate

```c
y = x.right
x.right == y.left
if y.left != NULL
    y.left.p = x
y.p = x.p
if x.p == NULL
    T.root = y
elseif x == x.p.left
    x.p.left =y
else x.p.right = y
y.left = x
x.p = y
```

#### Insert

##### Process

In order to insert a new node into a Red-Black Tree in O(lgN) time, we do like this.

1. **Insert node Z into the tree T as if it were an ordinary binary search tree**
2. **Color Z red**
3. **Fix**

##### Precedes

```c
RB-INSERT(T, z)
	y = T.nil
	x = T.root
	while x != T.nil
		y = x
		if z.key < x.key
			x = x.left
		else x = x.right
	z.p = y
	if y == T.nil
		T.root = z
	elseif z.key < y.key
		y.left = z
	else y.right = z
	z.left = T.nil
	z.right = T.nil
	z.color = RED
	RB-INSERT-FIXUP(T, z)
```

This is what the simple Insertion do above.

```c
RB-INSERT-FIXUP(T, z)
	while z.p.color == RED
		if z.p == z.p.p.left
			y = z.p.p.right
			if y.color == RED
				z.p.color = BLACK
				y.color = BLACK
				z.p.p.color = RED
				z = z.p.p
			else if z == z.p.right
				z = z.p
				Left-Rotate(T, z)
			z.p.color = BLACK
			z.p.p.color = RED
			Right-Rotate(T, z.p.p)
		else(same as then clause with "right" and "left" exchanged)
	T.root.color = BLACK
```

Now let's take a look at the complete code:

```c
#ifndef _AVL_TREES_H_
#define _AVL_TREES_H_

#endif /*_AVL_TREES_H_*/
// TODO
```
