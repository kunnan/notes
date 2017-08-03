Python基础之(六)文件
===

# 读文件

在某个文件夹下面建立了一个文件，名曰：130.txt，并且在里面输入了如下内容：

```
learn python
http://qiwsir.github.io
qiwsir@gmail.com
```

```python
f = open("123.txt") #打开已经存在的文件，此文件在当前目录，若在其他目录使用绝对路径
for line in f:
    print line, #Python 3: print(line, end='')

learn python
http://qiwsir.github.io
liuguoquan@gmail.com
```

紧接着做下面的操作：

```python
>>> for line2 in f:     #在前面通过for循环读取了文件内容之后，再次读取，
...     print line2         #然后打印，结果就什么也显示，这是什么问题？
...
>>>
```
这不是什么错误，是因为前一次已经读取了文件内容，并且到了文件的末尾了。再重复操作，就是从末尾开始继续读了。当然显示不了什么东西，但是Python并不认为这是错误。
> 在这里，如果要再次读取，那就从新`f = open('130.txt')`。

# 创建文件

```python
f = open("1234.txt","w") #创建文件
f.write("hello") #向文件写入内容
f.close() #关闭文件流
```
创建文件，我们同样是用`open()`这个函数，但是多了个`"w"`，这是在告诉Python用什么样的模式打开文件。也就是说，用`open()`操作文件，可以有不同的模式。看下表：

| 模式 | 描述 |
|------|------|
| r | 以读方式打开文件，可读取文件信息。|
| w | 以写方式打开文件，可向文件写入信息。如文件存在，则清空该文件，再写入新内容 |
| a | 以追加模式打开文件（即一打开文件，文件指针自动移到文件末尾），如果文件不存在则创建 |
| r+ | 以读写方式打开文件，可对文件进行读和写操作。 |
| w+ | 消除文件内容，然后以读写方式打开文件。 |
| a+ | 以读写方式打开文件，并把文件指针移到文件尾。 |
| b | 以二进制模式打开文件，而不是以文本模式。该模式只对Windows或Dos有效，类Unix的文件是用二进制模式进行操作的。 |

从表中不难看出，不同模式下打开文件，可以进行相关的读写。那么，如果什么模式都不写，像前面那样呢？那样就是默认为r模式，只读的方式打开文件。

# 使用with

使用with实现安全的关闭文件，不用close操作了

```python
>>> with open("130.txt","a") as f:
...     f.write("\nThis is about 'with...as...'")
...
>>> with open("130.txt","r") as f:
...     print f.read()
...
learn python
http://qiwsir.github.io
qiwsir@gmail.com
hello

This is about 'with...as...'
>>>
```

# 文件状态

```python
import os
file_stat = os.stat("123.txt")
print file_stat

posix.stat_result(st_mode=33188, st_ino=13575445, st_dev=16777220, st_nlink=1, st_uid=501, st_gid=20, st_size=57,
st_atime=1470988267,
st_mtime=1470988267,
st_ctime=1470988267) #文件创建时间

```

# read/readLine/readLines

- read：如果指定了参数size，就按照该指定长度从文件中读取内容，否则，就读取全文。被读出来的内容，全部塞到一个字符串里面。这样有好处，就是东西都到内存里面了，随时取用，比较快捷；“成也萧何败萧何”，也是因为这点，如果文件内容太多了，内存会吃不消的。文档中已经提醒注意在“non-blocking”模式下的问题，关于这个问题，不是本节的重点，暂时不讨论。
- readline：那个可选参数size的含义同上。它则是以行为单位返回字符串，也就是每次读一行，依次循环，如果不限定size，直到最后一个返回的是空字符串，意味着到文件末尾了(EOF)。
- readlines：size同上。它返回的是以行为单位的列表，即相当于先执行`readline()`，得到每一行，然后把这一行的字符串作为列表中的元素塞到一个列表中，最后将此列表返回。

有这样一个文档,you.md，其内容和基本格式如下：

```
    You Raise Me Up
    When I am down and, oh my soul, so weary;
    When troubles come and my heart burdened be;
    Then, I am still and wait here in the silence,
    Until you come and sit awhile with me.
    You raise me up, so I can stand on mountains;
    You raise me up, to walk on stormy seas;
    I am strong, when I am on your shoulders;
    You raise me up: To more than I can be.
```
分别用上述三种函数读取这个文件。

```python
>>> f = open("you.md")
>>> content = f.read()
>>> content #结果为字符串
'You Raise Me Up\nWhen I am down and, oh my soul, so weary;\nWhen troubles come and my heart burdened be;\nThen, I am still and wait here in the silence,\nUntil you come and sit awhile with me.\nYou raise me up, so I can stand on mountains;\nYou raise me up, to walk on stormy seas;\nI am strong, when I am on your shoulders;\nYou raise me up: To more than I can be.\n'
>>> f.close()
>>> print content
```

```python
>>> f = open("you.md")
>>> f.readline()
'You Raise Me Up\n'
>>> f.readline()
'When I am down and, oh my soul, so weary;\n'
>>> f.readline()
'When troubles come and my heart burdened be;\n'
>>> f.close()

f = open("you.md")
while True:
   line = f.readline()
   if not line:         #到EOF，返回空字符串，则终止循环
       break
   print line ,         #Python 3: print(line, end='')

f.close()                #别忘记关闭文件
```

```python
>>> f = open("you.md")
>>> content = f.readlines()
>>> content #结果为列表
['You Raise Me Up\n', 'When I am down and, oh my soul, so weary;\n', 'When troubles come and my heart burdened be;\n', 'Then, I am still and wait here in the silence,\n', 'Until you come and sit awhile with me.\n', 'You raise me up, so I can stand on mountains;\n', 'You raise me up, to walk on stormy seas;\n', 'I am strong, when I am on your shoulders;\n', 'You raise me up: To more than I can be.\n']

>>> for line in content:
...     print line ,         #Python 3: print(line, end='')
>>> f.close
```

# 读很大的文件fileinput

如果文件太大，就不能用`read()`或者`readlines()`一次性将全部内容读入内存，可以使用while循环和`readline()`来完成这个任务。

此外，还有一个方法：`fileinput`模块

```python
>>> import fileinput
>>> for line in fileinput.input("you.md"):
...     print line ,        #Python 3: print(line, end='')
...
You Raise Me Up
When I am down and, oh my soul, so weary;
When troubles come and my heart burdened be;
Then, I am still and wait here in the silence,
Until you come and sit awhile with me.
You raise me up, so I can stand on mountains;
You raise me up, to walk on stormy seas;
I am strong, when I am on your shoulders;
You raise me up: To more than I can be.
```

还有一种方法，更为常用：

```python
  >>> for line in f:
    ...     print line ,      #Python 3:  print(line, end='')
    ...
    You Raise Me Up
    When I am down and, oh my soul, so weary;
    When troubles come and my heart burdened be;
    Then, I am still and wait here in the silence,
    Until you come and sit awhile with me.
    You raise me up, so I can stand on mountains;
    You raise me up, to walk on stormy seas;
    I am strong, when I am on your shoulders;
    You raise me up: To more than I can be.
```

之所以能够如此，是因为文件是可迭代的对象，直接用for来迭代即可。

# seek

这个函数的功能是让指针移动。

```python
>>> f = open("you.md")
>>> f.readline()
    'You Raise Me Up\n'
    >>> f.readline()
    'When I am down and, oh my soul, so weary;\n'

>>> f.seek(0)  #回到文件的最开始位置
>>> f.readline()
    'You Raise Me Up\n'

此时指针所在的位置，还可以用`tell()`来显示，如

>>> f.tell()
    17L
```

`seek()`还有别的参数，具体如下：

>seek(...)
>    seek(offset[, whence]) -> None.  Move to new file position.

whence的值：

- 默认值是0，表示从文件开头开始计算指针偏移的量（简称偏移量）。这是offset必须是大于等于0的整数。
- 是1时，表示从当前位置开始计算偏移量。offset如果是负数，表示从当前位置向前移动，整数表示向后移动。
- 是2时，表示相对文件末尾移动。
