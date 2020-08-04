---
title: Python3 面向对象
tags:
  - Python
date: 2018-08-16
---

## 基础知识

```python
class humanbeing(object):
    love = 'forever'

    def __init__(self, name, sex, age):
        self.__name = name
        self.__sex = sex
        self.age = age

    def printInfo(self):
        print('name is %s, sex is %s, I\'m %d years old' % (self.__name, self.__sex, self.age))


class male(humanbeing):
    def printInfo(self):
        print('i\'m a male, i\'m %d years old' % self.age)


class female(humanbeing):
    def printInfo(self):
        # print('my name is %s, I\'m %d years old, and I\'m a female' % (self.__name, self.age))
        print('i\'m a femal, i\'m %d years old' % self.age)


class animal(object):
    def printInfo(self):
        print('I\'m a monster!')


def printInfo(humanbeing):
    humanbeing.printInfo()


def main():
    human = humanbeing('sherlock blaze', 'male', 19)
    # human = humanbeing('sherlock blaze')
    man = male('sherlock blaze', 'male', 19)
    woman = female('yifei liu', 'female', 28)
    printInfo(human)
    printInfo(man)
    printInfo(woman)
    printInfo(animal())
    print(isinstance(man, humanbeing))
    print(type(woman))
    print(type(human) == humanbeing)

    import types
    print(type(humanbeing) == types.FunctionType)
    print(type(printInfo) == types.FunctionType)
    print(dir(man))
    print(hasattr(man, '__name'))
    if hasattr(man, '__name'):
        print(getattr(man, '__name'))
        print(setattr(man, '__name', 'blaze'))
        print(getattr(man, '__name'))
    print(getattr(man, 'age'))
    setattr(man, 'age', 23)
    print(getattr(man, 'age'))

    print(humanbeing.love)

    man.length = 18
    print(man.length)

    del man.length
    # print(man.length)


main()
```

> 根据上述代码，我们做以下总结：

1. __ init __ 中定义了实例所必须具有的属性
2. class A(object)括号中为该类的父类
3. 通过a = A()的方式声明一个实例，同时必须要传入 __ init __ 声明的属性
4. 你可以通过a.xixi = 'x' 的方式给该对象添加属性，然后通过del a.xixi的方式删除掉它
5. __ name前面用 __ 修饰的属性名为私有的，无法从外部进行访问。
6. love 是类变量，跟 __ name不一样， __ name是实例变量，专属于一个实例
7. 父类和子类都拥有同一个方法，调用时会调用子类的方法。
8. animal类并不是humanbeing的子类，但是拥有跟humanbeing类一样的方法，所以同样可以传入printInfo方法中，并输出结果。这是由于python的鸭子类型，意思就是你不必的确是鸭子，只要你的行为看起来是鸭子，你就可以是一只鸭子。
9. isinstance(a, A)可以判断a是否是类A的实例
10. type(a)用于输出a的类型
11. getattr/setattr/hasattr 用于直接对a的属性进行操作
12. dir(a) 可以输出一个实例所有的属性及相关的操作，用在类上也是一样的，dir(A)，会输出类A所有的属性及具有的操作。

## 更多内容

### 动态地添加属性/方法

```python
class A(object):
    pass
    
a = A()
a.name = 'Blaze'
print(a.name)

def set_age(self, age):
    self.age = age

from types import MethodType
a.set_age = MethodType(set_age, a)
a.set_age(12)
print(a.age)

A.set_age = set_age
a1 = A()
a1.set_age(18)
print(a1.age)

def set_favourite(self, sport):
    self.sport = sport

A.set_favourite = MethodType(set_favourite, A)
a2 = A()
a2.set_favourite('basketball')
print(a2.sport)
```

### __ slots __

上面看到，对于对象a，我们可以通过a.xixixi来添加一个属性，但是我们如果不想让这样的事情发生呢？不允许增加新的属性。
针对上面的代码，我们需要这样写：

```python
class humanbeing(object):
    __slots__ = ('name', 'sex', 'age')
    ...
```

> 通过上面的方式，除了等式右边切片内的属性，将无法给实例添加任何属性。

### property

```python
class A(object):
    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        self._name = value
        
    @property
    def age(self):
        return self._age


def main():
    a = A('blaze', 23)
    a.name = 'sherlock blaze'
    print(a.name)


main()
```

> 通过@property 和 @name.setter让属性name成为了可读且可修改的属性，而age属性仅为可读属性。通过这个方式，我们可以像调用属性一样调用方法。

### 多重继承

```python
class electronics(object):
    pass
    
class smartphone(object):
    pass
    
class iphoneX(electornics, smarphone):
    pass
```

> 按照上面的操作即可实现多重继承，这种设计通常称之为MixIn

### __ str __

类似于Java的toString