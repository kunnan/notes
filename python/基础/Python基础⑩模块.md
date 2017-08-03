Python基础之(十)模块
===

# 编写模块

## 模块是程序

模块就是一个扩展名为`.py`的Python程序。

### 编写模块

```python
#!/usr/bin/env python
# coding=utf-8

lang = "python"
```

### 引入模块

```python
>>> import sys
>>> sys.path.append("~/Documents/VBS/StartLearningPython/2code/pm.py")
>>> import pm
>>> pm.lang
'python'
```

当Python解释器读取了`.py`文件，先将它变成由字节码组成的`.pyc`文件，然后这个`.pyc`文件交给一个叫做Python虚拟机的东西去运行（那些号称编译型的语言也是这个流程，不同的是它们先有一个明显的编译过程，编译好了之后再运行）。如果`.py`文件修改了，Python解释器会重新编译，只是这个编译过程不是完全显示给你看的。

我这里说的比较笼统，要深入了解Python程序的执行过程，可以阅读这篇文章：[说说Python程序的执行过程](http://www.cnblogs.com/kym/archive/2012/05/14/2498728.html)

有了`.pyc`文件后，每次运行就不需要重新让解释器来编译`.py`文件了，除非`.py`文件修改了。这样，Python运行的就是那个编译好了的`.pyc`文件。

### if __name__ == "__main__"

如果要作为程序执行，则`__name__ == "__main__"`；如果作为模块引入，则`pm.__name__ == "pm"`，即属性`__name__`的值是模块名称。

## 模块的位置

```python
>>> import sys
>>> import pprint
>>> pprint.pprint(sys.path)  #查看所有模块的位置
```

## `__all__`在模块中的作用

```python
    # /usr/bin/env python
    # coding:utf-8

    __all__ = ['_private_variable', 'public_teacher']

    public_variable = "Hello, I am a public variable."
    _private_variable = "Hi, I am a private variable."

    def public_teacher():
        print "I am a public teacher, I am from JP."    #Python 3: print("I am a public teacher, I am from JP.")

    def _private_teacher():
        print "I am a private teacher, I am from CN."    #Python 3:  print("I am a private teacher, I am from CN.")
```
`__all__`属性以及相应的值，在`__all__`属性列表中包含了一个私有变量的名字和一个函数的名字。这是在告诉引用本模块的解释器，这两个东西是有权限被访问的，而且只有这两个东西。

## 包或者库

包或者库，应该是比“模块”大的。也的确如此，一般来讲，一个“包”里面会有多个模块，当然，“库”是一个更大的概念了，比如Python标准库中的每个库都有好多个包，每个包都有若干个模块。

一个包是由多个模块组成，即多个`.py`的文件，那么这个所谓“包”也就是我们熟悉的一个目录罢了。现在就需要解决如何引用某个目录中的模块问题了。解决方法就是在该目录中放一个`__init__.py`文件。`__init__.py`是一个空文件，将它放在某个目录中，就可以将该目录中的其它`.py`文件作为模块被引用。

# 自带电池

在Python被安装的时候，就有不少模块也随着安装到本地的计算机上了。这些东西就如同“能源”、“电力”一样，让Python拥有了无限生机，能够非常轻而易举地免费使用很多模块。所以，称之为“自带电池”。

那些在安装Python时就默认已经安装好的模块被统称为“标准库”。

## 引用的方式

```python

import pprint #引入模块

from pprint import pprint #引入该模块下的方法

from pprint import * #引入该模块下的所有方法

import pprint as pr #重命名模块

from pprint import pprint as pt  #重命名方法

```

## 深入探究

-	`dir()`,查看对象的属性和方法
- `help()`查看对象的含义

```python

```

## 帮助、文档和源码

```python
print pprint.__doc__  #查看文档
print pprint.__file__ #查看模块的位置，根据这个位置查到源代码

```

# 标准库

## sys

### sys.argv

sys.argv是专门用来向python解释器传递参数，名曰“命令行参数”。

```python
    $ python --version # --veriosn就是命令行参数
    Python 2.7.6
```

### sys.exit()

退出当前程序.

在大多数函数中会用到return，其含义是终止当前的函数，并向调用函数的位置返回相应值（如果没有就是None）。但是`sys.exit()`的含义是退出当前程序——不仅仅是函数，并发起`SystemExit`异常。这就是两者的区别了。

如果使用`sys.exit(0)`表示正常退出。若需要在退出的时候有一个对人友好的提示，可以用`sys.exit("I wet out at here.")`，那么字符串信息就被打印出来。

### sys.stdout

与Python中的函数功能对照，`sys.stdin`获得输入（等价于Python 2中的raw_input()，Python 3中的input()），`sys.stdout`负责输出。

```python
    >>> f = open("stdout.md", "w")
    >>> sys.stdout = f  #重定向到文件
    >>> print "Learn Python: From Beginner to Master"        #Python 3: print("Learn Python: From Beginner to Master")
    >>> f.close()
```

## copy

```python
import copy
copy.copy() #浅拷贝
copy.deepcopy() #深拷贝
```
## os

### 操作文件

```python
import os
os.rename("22201.py", "newtemp.py") #重命名文件
os.remove("123.txt") #删除一个文件，不能是目录
```
### 操作目录

**os.listdir**：显示目录中的内容（包括文件和子目录）
**os.getcwd**：获取当前工作目录；
**os.pardir**:获得上一级目录
**os.chdir**：改变当前工作目录
**os.makedirs, os.removedirs**：创建和删除目录

### 文件和目录属性

`os.stat(p)`显示文件或目录的属性
`os.chmod()`改变权限

### 操作命令

`os`模块中提供了这样的方法，许可程序员在Python程序中使用操作系统的命令。

```python
    >>> p
    '/home/qw/Documents/VBS/StarterLearningPython'
    >>> command = "ls " + p #命令复制给Command变量
    >>> command
    >>> os.system(command) #执行命令
```

需要注意的是，`os.system()`是在当前进程中执行命令，直到它执行结束。如果需要一个新的进程，可以使用`os.exec`或者`os.execvp`。对此有兴趣详细了解的读者，可以查看帮助文档了解。另外，`os.system()`是通过shell执行命令，执行结束后将控制权返回到原来的进程，但是`os.exec()`及相关的函数，则在执行后不将控制权返回到原继承，从而使Python失去控制。

```python
#!/usr/bin/env python
# coding=utf-8

import webbrowser
webbrowser.open("http://www.baidu.com") #跨平台打开浏览器
```

```python

```

## heapq:堆

### headpq模块

```python
    >>> import heapq
    >>> heapq.__all__
    ['heappush', 'heappop', 'heapify', 'heapreplace', 'merge', 'nlargest', 'nsmallest', 'heappushpop']
```
**heappush(heap, x)**：将x压入堆heap

```python
    >>> import heapq
    >>> heap = []    
    >>> heapq.heappush(heap, 3)
    >>> heapq.heappush(heap, 9)
    >>> heapq.heappush(heap, 2)
    >>> heapq.heappush(heap, 4)
    >>> heapq.heappush(heap, 0)
    >>> heapq.heappush(heap, 8)
    >>> heap
    [0, 2, 3, 9, 4, 8]
```

**heappop(heap)**：删除最小元素

```python
    >>> heapq.heappop(heap)
    0
    >>> heap
    [2, 4, 3, 9, 8]
```

**heapify()**：将列表转换为堆

```python
    >>> hl = [2, 4, 6, 8, 9, 0, 1, 5, 3]
    >>> heapq.heapify(hl)
    >>> hl
    [0, 3, 1, 4, 9, 6, 2, 5, 8]
```

**heapreplace()**是`heappop()`和`heappush()`的联合，也就是删除一个，同时加入一个

```python
    >>> heap
    [2, 4, 3, 9, 8]
    >>> heapq.heapreplace(heap, 3.14)
    2
    >>> heap
    [3, 4, 3.14, 9, 8]
```

## deque:双端队列

```python
>>> qlst.append(5)        #从右边增加
>>> qlst
deque([1, 2, 3, 4, 5])
>>> qlst.appendleft(7)    #从左边增加
>>> qlst
deque([7, 1, 2, 3, 4, 5])
    
>>> qlst.pop() #右边删除一个元素
5
>>> qlst
deque([7, 1, 2, 3, 4])
>>> qlst.popleft() # 左边删除一个元素
7
>>> qlst
deque([1, 2, 3, 4])
    
>>> qlst.rotate(3) #循环移动n个位置
>>> qlst
deque([2, 3, 4, 1])

```
## calendar:日历

```python
import calendar
cal = calendar.month(2016,8)
print cal

    August 2016
Mo Tu We Th Fr Sa Su
 1  2  3  4  5  6  7
 8  9 10 11 12 13 14
15 16 17 18 19 20 21
22 23 24 25 26 27 28
29 30 31
```

**calendar(year,w=2,l=1,c=6)**
返回year年的年历，3个月一行，间隔距离为c。 每日宽度间隔为w字符。每行长度为`21* w+18+2* c`。l是每星期行数。

**isleap(year)**判断是否为闰年，是则返回true，否则false.

**leapdays(y1, y2)**返回在y1，y2两年之间的闰年总数，包括y1，但不包括y2.

**month(year, month, w=2, l=1)**返回year年month月日历，两行标题，一周一行。每日宽度间隔为w字符。每行的长度为7* w+6，l是每星期的行数。

**monthcalendar(year,month)**返回一个列表，列表内的元素还是列表。每个子列表代表一个星期，都是从星期一到星期日，如果没有本月的日期，则为0。

**monthrange(year, month)**返回一个元组，里面有两个整数。第一个整数代表着该月的第一天从星期几是（从0开始，依次为星期一、星期二，直到6代表星期日）。第二个整数是该月一共多少天。

**weekday(year,month,day)**输入年月日，知道该日是星期几（注意，返回值依然按照从0到6依次对应星期一到星期六）。

## time

### 常用方法

**time()**获得的是当前时间（严格说是时间戳），只不过这个时间对人不友好，它是以1970年1月1日0时0分0秒为计时起点，到当前的时间长度（不考虑闰秒）。

**localtime()**得到的结果可以称之为时间元组（也有括号），其各项的含义是：

|索引|	属性	|含义|
|----|----------|----|
|0	|tm_year|	年|
|1	|tm_mon|	月|
|2|	tm_mday	|日|
|3|	tm_hour|	时|
|4|	tm_min|	分|
|5|	tm_sec|	秒|
|6|	tm_wday|	一周中的第几天|
|7|	tm_yday|	一年中的第几天|
|8|	tm_isdst|	夏令时|

**gmtime()**localtime()得到的是本地时间，如果要国际化，就最好使用格林威治时间。

**asctime()**

```python
    >>> time.asctime()
    'Mon May  4 21:46:13 2015'
    
    time.asctime(h) #参数必须是时间元组，即localtime返回的值
```

**ctime()**

```python
>>> time.ctime()
'Mon May  4 21:52:22 2015'
    
 >>> time.ctime(1000000)  #参数是时间戳
'Mon Jan 12 21:46:40 1970'
```

**mktime()**mktime()也是以时间元组为参数，但是它返回的是时间戳

**strftime()**将时间元组按照指定格式要求转化为字符串。如果不指定时间元组，就默认为`localtime()`值。

|	格式|	含义|	取值范围（格式）|
|------|-------|-------------------|
|%y	|去掉世纪的年份	|00-99，如"15"|
|%Y	|完整的年份| 如"2015" |	
|%j	|指定日期是一年中的第几天|	001-366|
|	%m|	返回月份|	01-12|
|%b|	本地简化月份的名称|	简写英文月份|
|%B|	本地完整月份的名称|	完整英文月份|
|%d|	该月的第几日|	如5月1日返回"01"|
|%H|	该日的第几时（24小时制）|	00-23|
|%l|	该日的第几时（12小时制）|	01-12|
|%M|	分钟|	00-59|
|%S|	秒	|00-59|
|%U|	在该年中的第多少星期（以周日为一周起点）|	00-53|
|%W|	同上，只不过是以周一为起点|00-53|	
|%w|	一星期中的第几天	|0-6|
|%Z|	时区|在中国大陆测试，返回CST，即China Standard Time|
|%x|	日期|	日/月/年|
|%X|	时间|	时:分:秒|
|%c|	详细日期时间|	日/月/年 时:分:秒|
|%%|	‘%’字符	|‘%’字符|
|%p|	上下午|	AM    or    PM|

**strptime()**作用是将字符串转化为时间元组,其参数要指定两个，一个是时间字符串，另外一个是时间字符串所对应的格式，格式符号用上表中的。

```python
    >>> today = time.strftime("%y/%m/%d")
    >>> today
    '15/05/05'
    >>> time.strptime(today, "%y/%m/%d")
    time.struct_time(tm_year=2015, tm_mon=5, tm_mday=5, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=1, tm_yday=125, tm_isdst=-1)
```
## datetime

`datetime`模块中有几个类：

- datetime.date：日期类，常用的属性有year/month/day
- datetime.time：时间类，常用的有hour/minute/second/microsecond
- datetime.datetime：日期时间类
- datetime.timedelta：时间间隔，即两个时间点之间的时间长度
- datetime.tzinfo：时区类

### date类

```python

# 生成日期对象
>>> import datetime
>>> today = datetime.date.today()
>>> today
datetime.date(2015, 5, 5)

# 操作日期对象

>>> print today        #Python 3: print(today)
2015-05-05
>>> print today.ctime()        #Python 3: print(today.ctime())
Tue May  5 00:00:00 2015
>>> print today.timetuple()        #Python 3: print(today.timetuple())
time.struct_time(tm_year=2015, tm_mon=5, tm_mday=5, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=1, tm_yday=125, tm_isdst=-1)
>>> print today.toordinal()        #Python 3: print(today.toordinal())
735723
    
>>> print today.year
2015
>>> print today.month
5
>>> print today.day
5
    
# 时间戳与格式化时间格式的转换

>>> to = today.toordinal()
>>> to
735723
>>> print datetime.date.fromordinal(to)
2015-05-05

>>> import time
>>> t = time.time()
>>> t
1430787994.80093
>>> print datetime.date.fromtimestamp(t)
2015-05-05

# 修改日期。

>>> d1 = datetime.date(2015,5,1)
>>> print d1
2015-05-01
>>> d2 = d1.replace(year=2005, day=5)
>>> print d2
2005-05-05
```
### time类

```python

# 生成time对象

>>> t = datetime.time(1,2,3)
>>> print t
01:02:03

# 常用属性：

>>> print t.hour
1
>>> print t.minute
2
>>> print t.second
3
>>> t.microsecond
0
>>> print t.tzinfo
None
```

### timedelta类

主要用来做时间的运算。

```python
    >>> now = datetime.datetime.now()
    >>> print now        #Python 3: print(now)
    2015-05-05 09:22:43.142520

# 对`now`增加5个小时；

    >>> b = now + datetime.timedelta(hours=5)
    >>> print b        #Python 3: print(b)
    2015-05-05 14:22:43.142520

# 增加两周；

    >>> c = now + datetime.timedelta(weeks=2)
    >>> print c        #Python 3: print(c)
    2015-05-19 09:22:43.142520

# 计算时间差：

    >>> d = c - b
    >>> print d        #Python 3: print(d)
    13 days, 19:00:00
```

## urllib

`urllib`模块用于读取来自网上（服务器上）的数据，比如不少人用Python做爬虫程序，就可以使用这个模块。

```python
# 在Python 2中，这样操作：
    >>> import urllib
    >>> itdiffer =  urllib.urlopen("http://www.itdiffer.com")

# 但是如果读者使用的是Python 3，必须换个姿势：

    >>> import urllib.request
    >>> itdiffer = urllib.request.urlopen("http://www.itdiffer.com")
    
    >>> print itdiffer.read() #得到网页的内容
```

### urlopen()

`urlopen()`主要用于打开url文件，然后就获得指定url的数据，然后就如同在操作文件那样来操作,得到的对象叫做类文件对象。

参数说明一下：
- url：远程数据的路径，常常是网址
- data：如果使用post方式，这里就是所提交的数据
- proxies：设置代理

### url编码、解码

url对其中的字符有严格的编码要求，要对url进行编码和解码。

- quote(string[, safe])：对字符串进行编码。参数safe指定了不需要编码的字符
- urllib.unquote(string) ：对字符串进行解码
- quote_plus(string [ , safe ] ) ：与urllib.quote类似，但这个方法用'+'来替换空格`' '`，而quote用'%20'来代替空格
- unquote_plus(string ) ：对字符串进行解码；
- urllib.urlencode(query[, doseq])：将dict或者包含两个元素的元组列表转换成url参数。例如{'name': 'laoqi', 'age': 40}将被转换为"name=laoqi&age=40"
- pathname2url(path)：将本地路径转换成url路径
- url2pathname(path)：将url路径转换成本地路径

### urlretrieve()

将远程文件保存在本地存储器中.

`urllib.urlretrieve(url[, filename[, reporthook[, data]]])`

- url：文件所在的网址
- filename：可选。将文件保存到本地的文件名，如果不指定，urllib会生成一个临时文件来保存
- reporthook：可选。是回调函数，当链接服务器和相应数据传输完毕时触发本函数
- data：可选。如果用post方式所发出的数据

函数执行完毕，返回的结果是一个元组(filename, headers)，filename是保存到本地的文件名，headers是服务器响应头信息。

## urllib2

仅仅是针对Python 2的，在Python 3中，已经没有`urllib2`这个模块了，取代它的是`urllib.request`。

**Request类**

```python
>>>req = urllib2.Request("http://www.itdiffer.com")

# Python2
	>>> response = urllib2.urlopen(req)
	>>> page = response.read()
	>>> print page

Python 3:

    >>> response = urllib.request.urlopen(req)
    >>> page = response.read()
    >>> print(page)
```

`urllib2`或者`urllib.request`的东西还很多，比如还可以:

- 设置HTTP Proxy
- 设置Timeout值
- 自动redirect
- 处理cookie

## XML

Python提供了多种模块来处理XML。

- xml.dom.* 模块：Document Object Model。适合用于处理 DOM API。它能够将XML数据在内存中解析成一个树，然后通过对树的操作来操作XML。但是，这种方式由于将XML数据映射到内存中的树，导致比较慢，且消耗更多内存。
- xml.sax.* 模块：simple API for XML。由于SAX以流式读取XML文件，从而速度较快，切少占用内存，但是操作上稍复杂，需要用户实现回调函数。
- xml.parser.expat：是一个直接的，低级一点的基于 C 的 expat 的语法分析器。 expat接口基于事件反馈，有点像 SAX 但又不太像，因为它的接口并不是完全规范于 expat 库的。
- xml.etree.ElementTree (以下简称 ET)：元素树。它提供了轻量级的Python式的API，相对于DOM，ET快了很多
，而且有很多令人愉悦的API可以使用；相对于SAX，ET也有ET.iterparse提供了 “在空中” 的处理方式，没有必要加载整个文档到内存，节省内存。ET的性能的平均值和SAX差不多，但是API的效率更高一点而且使用起来很方便。

`ElementTree`在标准库中有两种实现。一种是纯Python实现：`xml.etree.ElementTree` ，另外一种是速度快一点：`xml.etree.cElementTree` 。

如果使用的是Python 2，可以像这样引入模块：

```python
    try:
        import xml.etree.cElementTree as ET
    except ImportError:
        import xml.etree.ElementTree as ET
```

如果是Python 3以上，就没有这个必要了，只需要一句话`import xml.etree.ElementTree as ET`即可，然后由模块自动来寻找适合的方式。显然Python 3相对Python 2有了很大进步。

### 常用属性和方法总结

ET里面的属性和方法不少，这里列出常用的，供使用中备查。

#### Element对象 

常用属性：

- tag：string，元素数据种类
- text：string，元素的内容
- attrib：dictionary，元素的属性字典
- tail：string，元素的尾形

针对属性的操作

- clear()：清空元素的后代、属性、text和tail也设置为None
- get(key, default=None)：获取key对应的属性值，如该属性不存在则返回default值
- items()：根据属性字典返回一个列表，列表元素为(key, value）
- keys()：返回包含所有元素属性键的列表
- set(key, value)：设置新的属性键与值

针对后代的操作

- append(subelement)：添加直系子元素
- extend(subelements)：增加一串元素对象作为子元素 
- find(match)：寻找第一个匹配子元素，匹配对象可以为tag或path
- findall(match)：寻找所有匹配子元素，匹配对象可以为tag或path
- findtext(match)：寻找第一个匹配子元素，返回其text值。匹配对象可以为tag或path
- insert(index, element)：在指定位置插入子元素
- iter(tag=None)：生成遍历当前元素所有后代或者给定tag的后代的迭代器
- iterfind(match)：根据tag或path查找所有的后代
- itertext()：遍历所有后代并返回text值
- remove(subelement)：删除子元素

#### ElementTree对象

- find(match)
- findall(match)
- findtext(match, default=None)
- getroot()：获取根节点.
- iter(tag=None)
- iterfind(match)
- parse(source, parser=None)：装载xml对象，source可以为文件名或文件类型对象.
- write(file, encoding="us-ascii", xml_declaration=None, default_namespace=None,method="xml")　

### 实例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore liu="a">
	<book category="COOKING">
		<title lang="en">Everyday Italian</title>
		<author>Giada De Laurentiis</author>
		<year>2005</year>
		<price>30.00</price>
	</book>
	<book category="CHILDREN">
		<title lang="en">Harry Potter</title>
		<author>J K. Rowling</author>
		<year>2005</year>
		<price>29.99</price>
	</book>
	<book category="WEB">
		<title lang="en">Learning XML</title>
		<author>Erik T. Ray</author>
		<year>2003</year>
		<price>39.95</price>
	</book>
</bookstore>
```

```python
#!/usr/bin/env python
# coding=utf-8

import xml.etree.ElementTree as ET

fd = open("xml.xml")

data = fd.read()

tree = ET.ElementTree(file="xml.xml")
print tree

#获得根元素
root = tree.getroot()
print root.tag
print root.attrib

#获得根元素下面的元素
for child in root:
    print child.tag,child.attrib
    for gen in child:
        print gen.tag,gen.text
```

## JSON

JSON建构于两种结构：

- “名称/值”对的集合（A collection of name/value pairs）。不同的语言中，它被理解为对象（object），纪录（record），结构（struct），字典（dictionary），哈希表（hash table），有键列表（keyed list），或者关联数组 （associative array）。
- 值的有序列表（An ordered list of values）。在大部分语言中，它被理解为数组（array）。

python标准库中有JSON模块，主要是执行序列化和反序列化功能：

- 序列化：encoding，把一个Python对象编码转化成JSON字符串
- 反序列化：decoding，把JSON格式字符串解码转换为Python数据对象

### encoding: dumps()

```python
data_json = json.dumps(data)
json.dumps(data, sort_keys=True, indent=2) #格式化输出json数据
```

### decoding: loads()


### 大json字符串

如果数据不是很大，上面的操作足够了。但现在是所谓“大数据”时代了，随便一个什么业务都在说自己是大数据，显然不能总让JSON很小，事实上真正的大数据，再“大”的JSON也不行了。前面的操作方法是将数据都读入内存，如果数据太大了就会内存溢出。怎么办？JSON提供了`load()`和`dump()`函数解决这个问题，注意，跟上面已经用过的函数相比，是不同的，请仔细观察。

```python
    >>> import tempfile    #临时文件模块
    >>> data
    [{'lang': ('python', 'english'), 'age': 40, 'name': 'qiwsir'}]
    >>> f = tempfile.NamedTemporaryFile(mode='w+')
    >>> json.dump(data, f)
    >>> f.flush()
    >>> print open(f.name, "r").read()        #Python 3: print(open(f.name, "r").read())
    [{"lang": ["python", "english"], "age": 40, "name": "qiwsir"}]
```

### 实例

```
{"code":20,"data":"liuguoquan","person":[{"name":"zhang","age":19,"sex":"male"},{"name":"zhang","age":20,"sex":"male"}]}
```

```python
#!/usr/bin/env python
# coding=utf-8

import json

class B(object):
    def __init__(self):
        self.age = 0
        self.name = ""
        self.sex = ""

class A(object):  
    def __init__(self):
        self.code = 2
        self.data = ""
        self.person = []
    

f = open("sample.json")
value = f.read();
print value

ret = json.loads(value)
print type(ret)

a = A()
#对象转为字典
a.__dict__ = ret
print a.code
print a.data
print a.person
print type(a.person)

for item in a.person:
    b = B()
    b.__dict__ = item;
    print b.age
    print b.name
    print b.sex
```

# 第三方库

## 安装第三方库

## 利用源码安装

在github.com网站可以下载第三方库的源码,通常会看见一个 setup.py 的文件。

```python
python setup.py install
```

## pip管理工具

>pip是一个以Python计算机程序语言写成的软件包管理系统，它可以安装和管理软件包，另外不少的软件包也可以在“Python软件包索引”（英语：Python Package Index，简称PyPI）中找到。

`pip install XXXXXX`（XXXXXX代表第三方库的名字）即可安装第三方库。

