---
title: Python3 基础语法
tags:
  - Python
date: 2018-08-16
---

## List/Tuple

> List和Tuple都是有序的列表，区别是List中的元素可以改变，而Tuple中的元素无法改变

```python
# 声明一个列表
list = [1, 2, 3]
# 列表中的元素可以是不同类型
list = [1, True, 'Sherlock Blaze']
# 可以通过类似访问数组的方式访问Python中的列表，比如
# list[0]、list[1]、list[2]分别得到 1、2、3
# 弹出最后一位的元素
list.pop()
# 弹出指定位置的元素
list.pop(1)
# 在结尾添加元素
list.append(4)
# 在指定位置插入元素
list.insert(0, 0)

# 声明一个元组
tuple = (1, 2, 3)
# 因为元组中的元素一旦生成就无法修改，所以没有其他操作
# 声明一个空元组
tuple = ()
# 声明只有一个元素的元组
tuple = (1,)
```

**Attention**

> 观察到上面声明一个元素元组的时候才用的语法为 tuple = (1,)，而不是 tuple = (1) 理由是，在第二种情况下，python解释器将()当做小括号处理，所以其实声明的tuple为数字 1。即 tuple = 1 

## 条件语句

```python
happy = True
if happy:
    print('xixixi')
    print('hahaha')
else:
    print('wuwuwu')
```

> 需要注意的是，这里的条件判断语句之后，有一个: ， 并且接在下面的语句都是通过缩进来控制代码域的。

## 循环 for/while

通过代码来看：

```python
sum = 0
for i in (1, 2, 3):
    sum += i
这里 (1, 2, 3) 是一个tuple，通过 in tuple 这个语法，可以让 i 遍历到tuple中的所有元素。
sum = 0
for i in range(5):
    sum += i
在这里，我们通过 range 函数，产生一个小于数字 5 的序列，也就是 [0, 1, 2, 3, 4]
num = 0
while num < 10:
    print(num)
    num += 1
    if num == 7:
        continue
    if num > 7:
        break
```

> 简单解释：在 num 小于 10时，执行循环下的语句，通过 break 语句，在 num 大于 7 时，退出循环，又通过 continue 语句，使得在 num 等于7时，直接重新从头开始执行循环体下的语句。所以以上代码输出结果为：

```sh
0
1
2
3
4
5
6
7
```

## dict/set

依然通过直接的代码来看
dict = {'J': 23, 343: 'T'}
以上是Python中声明一个字典的方式，所谓字典，类似于其他语言中的map，也就是键值对。可以观察到，python中声明字典的方式，跟直接写一个json没有区别。同时我们可以看到，dict中，所有key/value的数据类型不一定是要全部相同的。
用来做key的数据必须是不可变数据，在python中，字符串和整型数据都是不可变数据，所以都可以用来做key。
那么，何为不可变数据类型？
对于字符串类型的数据

```python
a = 'sherlock'
b = a.replace('s', 'S')
print(a)
print(b)
```

> 输出结果

```sh
sherlock
Sherlock
```

> 通过上述代码，我们可以看到虽然用replace代码把a中的小写s替换成了大写S，但是a的字符串值并没有发生改变，是另外生成了一个字符串来存储。

```python
setA = set([1, 2, 3])
setB = set([4, 5, 6, 4, 5])
print(setA)
print(setB)
setB.add(3)
setA.remove(2)
setC = setA & setB
setD = setA | setB
print(setC)
print(setD)
```

> 通过上述代码，知道了声明一个set的方式，& 是求两个set的交集， | 是求两个set的并集。

## 定义函数

```python
def sayhi():
    pass

def sayHello(x):
    if x > 10:
        pass
    return x * 5, x * 6
    
def showDefaultArg(x, y=1):
    print(x)
    print(y)

def showAlterableArg(*args):
    for i in args:
        print(i)
        
def showKeywordArg(**kw):
    if 'age' in kw:
        print(kw['age'])

def showAssignArg(*, name, age):
    print(name)
    print(age)
    
def main():
    sayhi()
    print(sayHello(3))
    a, b = sayHello(5)
    print(a)
    print(b)
    showDefaultArg(56)
    showAlterableArg(1, 2, 3)
    showAlterableArg(2, 5)
    showKeywordArg(name='sherlock', age=34)
    showAssignArg(name='blaze', age=45, face='handsome')

main()
```
    
程序输出

```sh
(15, 18)
25
30
```

> 通过上述的代码可以观察到

1. python定义函数的方式，
2. pass关键字，这个关键字可以看做是一个占位符，代表什么都不做
3. python 调用函数的方式
4. python 的函数可以返回多个值，但是通过print(sayHello(3))语句的输出 (15, 18)，我们可以观察到，返回的其实仍然是一个值，是一个tuple，只是在拿回值之后通过声明的顺序依次进行了赋值而已。
5. 可以设置默认参数值
6. 可以设置可变长度的参数值
7. 可以设置关键字参数值
8. 可以设置指定参数
在上述代码中，showAssignArg(name='blaze', age=45, face='handsome')语句执行时，程序会报错，因为face不是showAssignArg指定的参数值