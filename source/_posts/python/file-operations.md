---
title: Python3 简单文件操作
tags:
  - Python
date: 2017-06-17 11:40:49
---

> 首先介绍一下什么叫做相对路径和绝对路径，我们程序狗家族想必都是懂这个的，但是难免会有童鞋忘记。所以码出来供大家快速回忆一下。

相对路径

> 相对路径是相对于文件当前的工作路径而言的

绝对路径

> 绝对路径是由文件名和它的完整路径以及驱动器字母组成的，如果是Windows系统，那么某一个文件的绝对路径可能是:
c:\pythonworkspace\firstpy.py
在Unix平台上，文件的绝对路径可能是: /home/sherlockblaze/Documents/pythonworkspace/firstpy.py

## 文件类型

> 文件大概可以分为文本文件和二进制文件。在不同操作系统下，可以用文本编辑器编辑的文件，都称为文本文件，那么其他的文件就属于二进制文件。而二进制文件相比与文本文件的优势在于二进制文件的处理效率更高一些。

## 读取文件的开始

> 读取一个文件的思路永远都是相同的，第一步自然就是打开一个文件。在python中我们通过如下代码使用open函数来打开一个文件。

```python
input = open(filepath,mode)
```

我们的mode主要由以下几种方式。

| 模式 | 作用 |
| --- | --- |
| r  | 读取模式 |
| w  | 写入模式 |
| a  | 追加模式 |
| rb | 读取二进制数据模式打开文件 |
| wb | 写入二进制数据模式打开文件 |

同样我们有两种途径来打开文件。

+ 通过绝对路径

```python
input = open("/Users/sherlockblaze/Documents/pythonworkspace/Test.txt","r")
```

+ 通过相对路径（需要注意的是，我们通过相对路径是可以打开当前工作目录下的文件的，也就是说如果我的.py文件存在 ／User/sherlock/Documents 下的话，我们通过相对路径打开的文件也同样存在当前路径下）

```python
input = open("Test.txt","r")
```

**注意**

> 在Windows下我们通过绝对路径来打开文件的时候，我们需要在绝对文件名前加上一个 r 前缀，来表示这个字符串是一个行字符串，这样可以让python解释器将文件中的反斜线理解成字面意义上的反斜线。例如：

```python
input = open(r"d:\pythonworkspace\Test.txt","r")
```

> 如果我们不添加 r 作为前缀，则需要用转义字符把上面的语句修改成如下所示：

```python
input = open("d:\\pythonworkspace\\Test.txt","r")
```

## 向文件中写入数据

> 我们首先通过写入的方式打开文件，然后通过调用write方法，向文件中写入数据。

```python
def main():
    input = open("Test.txt","w")
    input.write("SherlockBlaze")
    input.write("\t is the most handsome guy!\n")
    input.close()

main()
```

> 通过这种方式，我们往当前目录下的 Test.txt 文件中写入了 SherlockBlaze is the most handsome guy! 这句话，并且需要注意的是，我们在写完文件后，调用close()方法关闭了文件流。

常见小特性

> 当使用w模式打开一个文件时，如果文件不存在，open函数就会创建一个新文件，如果该文件存在，那么这个文件里的内容会被心的内容覆盖。当我们用读／写模式打开文件的时候，文件内部会添加一个叫做文件指针的特殊标记，文件的读写操作都发生在指针当前位置上。

判断文件是否存在

> 为了避免误操作，我们可以通过os.path模块中的isFile函数来判断一个文件是否存在。即：

```python
import os.path
is os.paht.isfile("Test.txt"):
    print("Test.txt exists")
else:
    print("Test.txt doesn't exists")
```

## 简单小程序

+ 输入文件路径，并且从中计算各个字母出现的次数

```python
def main():
    filename = input("Enter a filename: ").strip()
    infile = open(filename,"r")

    counts = 26 * [0]
    for line in infile:
        countLetters(line.lower(),counts)

    for i in range(len(counts)):
        if counts[i] != 0:
            print(chr(ord('a') + i) + "appears " + str(counts[i])
            + (" time" if counts[i] == 1 else " times"))

    infile.close()

def countLetters(line,counts):
    for ch in line:
        if ch.isalpha():
            counts[ord(ch) - ord('a')] += 1

main()
```

> 思路简单叙述：首先创建数组，每当读取到一个字符，对对应位置的数字进行加一，最后在进行遍历得到输出。

+ 下载网站源代码，然后写入目的文件中

```python
import sys
import urllib
import urllib.request
import os.path

def download(url,num_retries = 2):
    print ('Downloading:',url)
    try:
        html = urllib.request.urlopen(url).read()
    except urllib.URLError as e:
        print ('Download error:',e.reason)
        html = None
        if num_retries > 0:
            if hasattr(e,'code') and 500 <= e.code <600:
                return download(url,num_retries-1)
    return html

def main():
    url = input("Enter a url:\n").strip()
    f2 = input("Enter a target file:\n").strip()
    if os.path.isfile(f2):
        print(f2 + " already exists")
        sys.exit()

    html = download(url)
    target = open(f2,"w")

    content = html.decode(encoding="utf-8")
    target.write(content)
    target.close()

main()
```

> 比如我输入网址 `http://www.game2.cn/`，在输入目的文件：game2.txt。即可进行下载并把对应html输入到当前工作目录的game2.txt文件中。