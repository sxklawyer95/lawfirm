---
title: Python3 函数式编程(一)
tags:
  - Python
date: 2018-08-16
---

## 高阶函数

```python
f = abs
print(f(-10))

def showtime(x, f):
    print(f(x))
    
def main():
    showtime(-20, abs)
    
main()
```

> 通过以上的代码，我们发现函数名，就是一个指向函数的指针。如果熟悉C语言，我们其实可以知道以f()方式来调用函数仅仅是一个语法糖，实际上也是通过指针来访问的。

## map/reduce

```python
def f(x):
    return x * x

r = map(f, [1, 2, 3, 4, 5, 6])
rlist = list(r)
print(rlist)
```

> 上面的代码可以看到，map接收两个参数，一个是一个函数，一个是iterable。可以对元素进行同样的处理。

```python
def add(x, y):
    return x * 10 + y
    
from functools import reduce
result = reduce(add, [1, 2, 3, 4, 5])
print(result)
reduce 函数的参数同样是两个，一个是函数，一个是序列。可以把要做的操作迭代在每个元素上，上面的语句等同于:
f(f(f(f(1, 2), 3), 4), 5)
filter

def is_odd(x):
    return x % 2 == 0

list = list(filter(is_odd, [1, 2, 3, 4, 5, 6, 7]))
print(list)
```

> 通过上面的代码，我们可以观察到，filter同样传入一个函数，和一个序列。上述代码输出后，我们可以得到这样的结论：对于filter函数，它是把序列中的元素，根据传入函数返回值为True/False来进行筛选。

```python
def _odd_list():
    n = 1
    while True:
        n = n + 2
        yield n

def _no_division(n):
    return lambda x: x % n > 0
    
def primes():
    yield 2
    it = _odd_list()
    while True:
        n = next(it)
        yield n
        it = filter(_no_division(n), it)
        
for n in primes():
    if n < 1000:
        print(n)
    else:
        break
```

## 返回函数

python可以把函数作为返回值返回

```python
def do_later(*arg):
    def sum():
        sum = 0
        for element in arg:
            sum += element
        return sum
    return sum

f = do_later(1, 2, 3, 4, 5)
print(f())
```

> 上面的代码，把sum函数作为返回值返回给调用者，这个方式，导致调用do_later方法时不是立马进行计算，而是在实际执行f()方法的时候，才会进行计算。这样的方式叫做闭包。

这个方式用起来很酷，但是同时要注意可能存在的问题：

```python
def do_later():
    fs = []
    for i in range(3):
        def multiply():
            return i * i
        fs.append(multiply)
    return fs
    
f1, f2, f3 = do_later()
print(f1())
print(f2())
print(f3())
```

输出：

```sh
4
4
4
```

> 通过上面的例子，我们可以看到里面存在的问题。也很好理解，函数也是存在内存中的一段指令，里面所用到的参数值，也是通过同样的方式来保存的。在上面，f1/f2/f3都利用了i这个参数，但是随着迭代，i的值发生了变化，所以在真正调用执行函数的时候，输出的结果都是4。

## lambda

> 通过lambda关键字，我们可以声明匿名函数。

```python
f = lambda x: x * x
print(f)
print(f(4))
f = lambda x, y: x * y
print(f)
print(f(4, 3))
```

> 通过上面的代码，我们总结如下结论:

1. lambda后的标识符为函数的参数
2. 不需要提供return语句，后面的表达式即为返回的内容。

## 偏函数

```python
print(int('3421', base=16))
print(int('3421', 16))

import functools
int16 = functools.partial(int, base=16)
print(int16('3421'))

def multiply(x, y):
    return x * y
    
multiply10 = functools.partial(multiply, y = 10)
multiply2 = functools.partial(multiply, x = 2)
print(multiply10(10))
print(multiply2(y=4))

# 会报错
print(multiply2(4))
```

> 通过上面的代码，我们通过functools.partial，将某个函数的参数设置为一个固定值，这样可以满足对某些特定意义的函数要求，很方便的进行调用。会报错的语句是因为赋值是按照顺序来的，multiply2的时候，x已经有固定值为2，按照顺序，调用语句会再给x做一次赋值，这一次是4，导致***TypeError: multiply() got multiple values for argument 'x'***

## 装饰器

```python
def printname():
    print('Sherlock Blaze')
```

> 有上述函数定义，当我们想在输出Sherlock Blaze 之前，输出一些日志，比如先输出，who's the most handsome man。但是又不想修改printname函数的定义，这个时候，我们可以通过装饰器来包装一下这个函数，达到我们想要的效果。

```python
def log(func):
    def wrapper(*arg, **kw):
        print('who\'s the most handsome man.')
        return func(*arg, **kw)
    return wrapper

@log
def printname():
    print('Sherlock Blaze')
    
printname()
```

> 通过上面的代码，我们可以观察到装饰器大概的用法。可以有好几层，比如：

```python
def log(text):
    def decorator(func):
        def wrapper(*arg, **kw):
            print('who\'s the most handsome man.')
            return func(*arg, **kw)
        return wrapper
    return decorator
    
@log('xixixi')
def printname():
    print('Sherlock Blaze')
    
printname()
```

> 注意一下装饰器中func参数的位置