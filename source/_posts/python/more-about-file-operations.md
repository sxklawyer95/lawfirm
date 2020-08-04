---
title: Python3 文件操作（一年后新版）
tags:
  - Python
date: 2018-08-16
---

## 打开文件

```python
f = open(filepath, mode)
```

> 通过上述代码，可以打开一个文件

以下是mode的种类：

| mode | 写法 |
| ---- | --- |
| 读取模式 | r |
| 写入模式 | w |
| 读取二进制模式 | rb |
| 写入二进制模式 | wb |
| 追加模式 | a |
| 以读写模式打开文件 | r+/w+/a+ |
| 以二进制读写模式打开 | rb+/wb+/ab+ |

**注意** 

当在windows下通过绝对路径来打开文件的时候，需要注意以下内容：

```python
f = open(r"d:\pythonworkspace\Test.txt","r")
```

> 通过添加一个r，来表示这个字符串是一个行字符串，这样可以让python解释器将文件中的反斜线理解成字面意义上的反斜线。

或者不想加r，就需要这样来写

```python
f = open("d:\\pythonworkspace\\Test.txt","r")
```

## 文件相关的方法

```python
with open('/Users/sherlockblaze/Desktop/stu.txt', 'a+') as f:
    f.write('the most handsome guy')
    for line in f.readlines():
        print(line)

try:
    f = open('/Users/sherlockblaze/Desktop/stu.txt', 'r')
    print('---------')
    print(f.readline())
finally:
    f.close()

try:
    f = open('/Users/sherlockblaze/Desktop/stu.txt', 'r')
    print('---------')
    print(f.read())
finally:
    f.close()

try:
    f = open('/Users/sherlockblaze/Desktop/stu.txt', 'r')
    print('---------')
    print(f.read(4))
finally:
    f.close()


try:
    f = open('/Users/sherlockblaze/Desktop/stu.txt', 'r', encoding='UTF-8')
    print('---------')
    print(f.read())
finally:
    f.close()
```

> 通过上述代码，我们总结如下：

1. with 语句的操作后，无需再手动关闭文件
2. readlines方法是读取所有行，并返回一个列表
3. read方法是读取所有内容，可以在read方法中指定需要读取的字节数，防止内容过多，爆掉内存，readline是读取一行内容
4. 打开的文件操作完成后记得手动close掉(除with语句的操作方式外)

## StringIO/BytesIO

```python
StringIO
from io import StringIO

s = StringIO()
s.write('sherlock')
print(s.getvalue())

s.write(' blaze')
# print(s.read())
print(s.getvalue())

s = StringIO('sherlock blaze is the most handsome man in this world')
print(s.getvalue())
print(s.read())
# print(s.readline())
# print(s.readlines())

from io import BytesIO

b = BytesIO()
b.write('帅气的夏洛克'.encode('UTF-8'))
print(b.getvalue())

b = BytesIO(b.getvalue())
print(b.read())
```

> 有时候读取不一定要从文件中，还可以从内存中，StringIO可以让你生成一个字符串，然后获取，BytesIO主要就是用于字节。注意的是，**只有在声明时做了赋值，才可以通过类似于操作文件的read方法等进行读取，否则只能用getvalue。**

## 更多操作

***os模块，通过导入os模块可以进行更多操作***

> 导入os模块

```python
import os
```

> 获取环境变量以及获取指定的环境变量

```python
os.environ
os.environ.get('JAVA_HOME')
os.mkdir('/Users/sherlockblaze/Desktop/xixi')
os.rmdir('/Users/sherlockblaze/Desktop/xixi')
```

## 创建和删除目录

```python
os.path.split('/Users/sherlockblaze/Desktop/xixi.txt')
os.path.splitext('/Users/sherlockblaze/Desktop/xixi.txt')
```

> 第一个是把文件名和路径分隔开，第二个是拿到文件的扩展名

```python
os.rename('xixi.txt', 'hxihxi.txt')
os.remove('hxihxi.txt')
```

> 文件重命名及删除文件