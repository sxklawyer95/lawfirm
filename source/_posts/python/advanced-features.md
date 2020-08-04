---
title: Python3 高级特性
tags:
  - Python
date: 2018-08-16
---

## 切片

```python
# 创建 0-99 的列表
L = list(range(100))
print(L[:])
print(L[1:10])
print(L[-10:])
print(L[::2])
```

> 通过上述代码，运行输出后，我们可以得到如下的结论：
1. 复制了一个列表
2. 截取了L列表从index为1到index为9的元素
3. 截取了L列表倒数第10个元素到最后一个元素
5. 所有的元素，每隔两个元素取一个

***不仅可以对列表做如上的操作，对tuple也可以。***

## 迭代

```python
for ch in 'ABC':
    print(ch)
    
for index, value in enumerate(['A', 'B', 'C']):
    print(index, value)

from collections import Iterable
isinstance('abc', Iterable)
isinstance(123, Iterable)
```

> 通过上述代码，运行输出后，我们可以得到如下的结论：
1. 字符串可以被迭代
2. 可以通过enumerate方法，将一个list变成一个 index - value 键值对，然后进行迭代
3. 可以通过isinstance方法判断一个对象是否可以被迭代，要先从collections中导入Iterable

**注**：enumerate对tuple也有效

## 列表生成式

```python
list = [x * x for x in range(1, 100) if x % 2 == 0]
list = [m + n for m in 'ABC' for n in 'XYZ']
```

## 生成式

```python
# 生成式的用法
list = (x * x for x in range(1, 100) if x % 2 == 0)
for n in list:
    print(next(list))
    
# yield的用法
def fib(max):
    n, a, b = 0, 1, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
    
def main():
    f = fib(5)
    for n in f:
        print(n)
    
main()
```

> 先看生成式的用法，通过上面的用法，可以知道通过(***生成式***)的格式，我们可以得到一个generator，generator保存的不是整个列表，而是保存了所需列表下一个元素的计算方法。当程序运行到yield语句时，程序会返回，等到调用next()的时候，程序再从yield下面的语句开始执行。

```python
def generateNum():
    yield 1
    print('a')
    yield 2
    print('b')
    yield 3
    print('c')

def main():
    g = generateNum()
    print(next(g))

main()
```

> 上面的代码体现了yield的执行方式

## 迭代器

```python
it = iter([1, 2, 3, 4, 5])
while True:
    try:
        x = next(it)
    except StopIteration:
        break
```

> 通过iter()方法可以获得一个iterator，然后通过next()方法获取下一个元素。
iterator的运行是惰性的，我们无法将很多的元素一次性存储在内存中，通过这个方式，我们可以得到更多的数据。