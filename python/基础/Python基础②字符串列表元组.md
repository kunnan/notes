Python基础②字符串列表元组
====

## Python字符串

在Python中，万物皆对象，显然字符串是对象类型，用str表示。字符串类型通常用单引号或者双引号包裹起来。

```python
>>> "Hello,world"
'Hello,world'
>>> 'Hello,world'
'Hello,world'
>>> type("Hello,world")
<type 'str'>
>>> 
```

### 变量和字符串

在Python中变量无类型，对象有类型

```python
>>> b = "hello,world"
>>> b
'hello,world'

>>> print b
hello,world

>>> type(b)
<type 'str'>

```

### 连接字符串

```python
>>> "py" + "thon"
'python'
>>> 

>>> a = 1989
>>> b = "free"
>>> print a + b

Traceback (most recent call last):
  File "<pyshell#195>", line 1, in <module>
    print a + b
TypeError: unsupported operand type(s) for +: 'int' and 'str'

>>> print b + `a` #反引号将整数a转化为字符串
free1989
>>> 
```
用“+”拼接起来的字符串的两个对象必须是同一类型的。如果是数字则是求和，如果是字符串则得到一个新的字符串。

还有其他两种方法可以将整数转化为字符串：

-	str():将整数对象转换为字符串对象

```
>>> print b + str(a)
free1989
```

-	repr():相当于反引号``

```
>>> print b + repr(a)
free1989
```

-	int():将字符串对象转换为整数对象

```
>>> a = "250"
>>> b = int(a)
>>> b
250
>>> type(b)
<type 'int'>
>>> 
```

### 转义字符

在字符串中，总会有一些特殊的符号，就需要用转义符。所谓转义，就是不采用符号本来的含义，而采用另外一含义。下面表格中列出常用的转义符：

|转义字符  |  描述 |
|----------|-------|
| \ | (在行尾时) 续行符 |
| \\ | 反斜杠符号 |
| \' | 单引号 |
| \" | 双引号 |
| \a | 响铃 |
| \b | 退格(Backspace) |
| \e | 转义 |
| \000 |  空 |
| \n | 换行 |
| \v | 纵向制表符 |
| \t | 横向制表符 |
| \r | 回车 |
| \f | 换页 |
| \oyy | 八进制数，yy代表的字符，例如：\o12代表换行|
| \xyy  | 十六进制数，yy代表的字符，例如：\x0a代表换行|
| \other | 其它的字符以普通格式输出 |

### 原始字符串

用转义符能够让字符串中的某些符号表示原来的含义，而不是被解析成某种具有特别能力的符号。下面看看这个代码：

```python
>>> dos = "c:\news"
>>> dos
'c:\news'
>>> print dos #当用print打印时就出现问题了
c:
ews
```
如何避免上述代码的问题？有两种方法，其一是前面介绍的转义符解决。

```python
>>> dos = "c:\\news"
>>> print dos
c:\news
>>> 
```

其二是声明为原始字符串

```python
>>> dos = r"c:\news"
>>> print dos
c:\news
>>> dos = r"c:\news\python"
>>> print dos
c:\news\python
```

如`r"c:\news"`，由r开头引起的字符串就是声明了后面引号里的东西是原始字符串，在里面放任何字符都表示该字符的原始含义，不需要再用转义符了。

### raw_input和print

下面实现接收和打印用户通过键盘输入的内容：

不过在写这个功能前，要了解函数：

- Python 2：`raw_input()`
- Python 3: `input()`

这是Python的内建函数（built-in function）。关于内建函数，可以分别通过下面的链接查看：

- [Python 2的内建函数](https://docs.python.org/2/library/functions.html)
- [Python 3的内建函数](https://docs.python.org/3.5/library/functions.html)

关于Python的内建函数，下面列出来，供参考：

```
abs()	divmod()	input()	open()	staticmethod()
all()	enumerate()	int()	ord()	str()
any()	eval()	isinstance()	pow()	sum()
basestring()	execfile()	issubclass()	print()	super()
bin()	file()	iter()	property()	tuple()
bool()	filter()	len()	range()	type()
bytearray()	float()	list()	raw_input()	unichr()
callable()	format()	locals()	reduce()	unicode()
chr()	frozenset()	long()	reload()	vars()
classmethod()	getattr()	map()	repr()	xrange()
cmp()	globals()	max()	reversed()	zip()
compile()	hasattr()	memoryview()	round()	__import__()
complex()	hash()	min()	set()	
delattr()	help()	next()	setattr()	
dict()	hex()	object()	slice()	
dir()	id()	oct()	sorted()
```

怎么才能知道哪个函数怎么用，并且用来干什么的呢？

-	help(abs)命令
-	[Python的官网https://docs.python.org/2/library/functions.html](https://docs.python.org/2/library/functions.html)

-	raw_input

```python
>>> raw_input("input your name: ")
input your name: michael
'michael'
>>> name = raw_input("input your name: ")
input your name: michael
>>> name
'michael'
>>> type name
SyntaxError: invalid syntax
>>> type(name)
<type 'str'>
>>> age = raw_input("How old are you? ");
How old are you? 25  #输入数字
>>> age
'25'
>>> type(age)
<type 'str'>  #返回仍是str类型
```
-	print

```python
>>> print "Hello, world"
Hello, world
>>> a = "python"
>>> print a
python
>>> b = "good"
>>> print a,b
python good
>>> 
```

特别提醒的是，print的返回值默认是以`\n`结尾的，所以每个输出语句之后自动换行。

### 索引和切片

#### 索引

在Python中，把像字符串这样的对象类型统称为**序列**，顾名思义，序列就是有序排列。在这个序列中，每个人都有编号，编号和每个字符一一对应，在Python中这些编号称为**索引**。

```python
>>> lang = "study python"
>>> lang[0]
's'
>>> lang[1]
't'
```

也可以这样做：

```python
>>> "study python"[0]
's'
>>> "study python"[1]
't'
>>> 
```

通过字符找到其在字符串中的索引值：

```python
>>> lang = "study python"
>>> lang.index("p")
6
>>> lang.in
SyntaxError: invalid syntax
>>> lang.index("y")
4
```

#### 切片

不管是得到一个字符还是多个字符，通过索引得到字符的过程称为**切片**。

```python
>>> lang = "study python"

>>> lang[2:9] #得到从2号到9号之间的字符串（包括2号但不包括9号）
'udy pyt'
>>> b = lang[1:] #得到从1号到最末尾的字符
>>> b
'tudy python'
>>> c = lang[:] #得到所有的字符
>>> c
'study python'
>>> d = lang[:10] #得到从0到10号之前的字符
>>> d
'study pyth'
>>> lang[1:12] 
'tudy python'
>>> lang[0:13] #如果第二个数字的长度大于字符串的长度，得到的返回结果就自动到最大长度终止，但是这种获得切片的方法是值得提倡的。特别是如果在循环中，这样做很可能会遇到麻烦。
'study python'
>>> lang[0:14]
'study python
```

如果在切片的时候，冒号左右都不写数字，就是前面所操作的`c = lang[:]`,其结果是变量的值c与源字符串lang一样，即复制了一份，那是不是真的复制呢？下面的方式检验一下：

```python
>>> id(c)
44841944
>>> id(lang)
44841944
```
从上面可以看出,两个内存地址一样，说明c和lang两个变量指向的是同一个对象。用`c = lang[:]`的方式并没有生成一个新的字符串，而是将变量c这个标签页贴在原来那个字符串上了。

```python
>>> lang = "study python"
>>> c = lang
>>> id(c)
50456160
>>> id(lang)
50456160
```
上述这种情况，变量c和lang也指向同一个对象。

### 基本操作

所有序列都有如下操作，字符串是序列的子集。

-	len():返回序列长度
-	+:连接两个序列
-	*:重复序列元素
-	in: 判断元素是否存在于序列中
-	max():返回最大值
-	min():返回最小值
-	cmp(str1,str2): 比较两个序列值是否相同

#### "+"连接字符串

```python
>>> str1 = "study"
>>> str2 = "python"
>>> str1 + str2
'studypython'
>>> str1 + "-->" + str2
'study-->python'
```

#### in

in用来判断某个字符串是不是在另外一个字符串内，或者判断某个字符串内是否包含某个字符串，包含则返回True，否则返回false。

```python
>>> str = "study python"
>>> "s" in str
True
>>> "i" in str
False
>>> "su" in str
False
>>> "stu" in str
True
>>> 
```
#### max()

```python

```
>>> str = "python"
>>> max(str)
'y'
```python

```
#### min()

```python
>>> str = "python"
'y'
>>> min(str)
'h'
```
#### cmp()

将两个字符串进行比较，首先将字符串中的符号转化为对应的数字，然后再比较

-	如果返回的数值小于0，说明前者小于后者
-	等于0，表示两者相等
-	大于0，表示前者大于后者

```python
>>> cmp("aaa","abc")  #
-1
>>> cmp("abc","aaa");
1
>>> cmp("abc","abc");
0
```


```python
>>> ord("a") #返回字符的ASCII值
97
>>> chr(97)  #返回ASCII值所对应的字符
'a'
```

#### "*"

以指定的次数重复打印字符串

```python
>>> str = "python"
>>> str * 4
'pythonpythonpythonpython'
>>> print "-" * 20
--------------------
```

#### len()

len()返回字符串的长度，返回值类型是int型

```python
>>> a = "hello"
>>> m = len(a)
>>> m
5
>>> type(m)
<type 'int'>
>>> 
```

### 常用的字符串方法

字符串的方法有很多，可以通过dir来查看：

```
>>> dir(str)
['__add__', '__class__', '__contains__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__getslice__', '__gt__', '__hash__', '__init__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '_formatter_field_name_split', '_formatter_parser', 'capitalize', 'center', 'count', 'decode', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'index', 'isalnum', 'isalpha', 'isdigit', 'islower', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
```

#### 判断字符串是否全是字母

```
>>> "python".isalpha()  #字符串全是字母，返回TRUE
True
>>> "2python".isalpha() #字符串含有非字母，返回FALSE
False
>>> 
```
#### 分割字符串

split()作用是将字符串根据某个分隔符进行分割。

```
>>> str = "I love Beijing"
>>> str.split(" ")
['I', 'love', 'Beijing']  #返回值为名叫列表（list）的类型
>>> str = "www.baidu.com"
>>> str.split(",")
['www', 'baidu', 'com']
>>> 
```
#### 去掉字符串两头的空格

```
>>> str = "  python  "
>>> str.strip()  #去掉字符串左右空格
'python'
>>> str.lstrip() #去掉字符串左边空格
'python  '
>>> str.rstrip() #去掉字符串右边空格
'  python'
>>> 
```
#### 字符大小写转换

```
>>> str = "pyThon" 
>>> str.upper() #将小写字母完全变为大写字母
'PYTHON'
>>> str.lower() #将大写字母完全变成小写字母
'python'

>>> str.isupper() #判断是否全是大写字母
False
>>> str.islower() #判断是否全是小写字母
False
>>> str = "this is book"
>>> str.capitalize() #把所有单词的第一个字母转化为小写字母
'This is book'
>>> str.istitle() #判断每个单词的第一个字母是否为大写字母
False
>>> 
```
#### join连接字符串

```
>>> b = "www.baidu.com"
>>> c = b.split(".")
>>> c
['www', 'baidu', 'com']
>>> ".".join(c)
'www.baidu.com'
>>> "**".join(c)
'www**baidu**com'
```

### 字符串格式化输出

#### 使用占位符

```
>>> "I like %s" % "Python" #%s表示字符串
'I like Python'
>>> "%d years old" % 25 #%d表示整数
'25 years old'
"I like 'python'"
>>> "%s is %d years old" % ("deason",25)
'deason is 25 years old'
```
- %s 字符串（采用str()的显示）
- %r 字符串（采用repr（）的显示）
- %c 单个字符
- %b 二进制整数
- %d 十进制整数
- %e 指数（底数为e）
- %f 浮点数

#### 使用format格式化

{}作为占位符

```
>>> str = "I like {}".format("python");
>>> str
'I like python'
>>> str = "{0} is {1} years old".format("michael",25)
>>> str
'michael is 25 years old'
```

#### 字典式格式化

```
>>> lang = "python"
>>> print "I love %(program)s" %{"program":lang}
I love python
>>> print "I love %(program)s" %{"program":"python"}
I love python
```

综上，推荐使用:string.format()

## Python列表

此前已经知道了Python的三种对象类型：int、float和str。下面开始学习一种新的Python对象类型：list。list在Python中具有非常强大的功能。

### 定义

在Python中，用方括号表示一个list：[]，方括号里面的元素类型，可以是int型数据，也可以是str类型的数据，甚至也可以是bool类型数据。而像Java语言，数组里面只能存在一种类型的数据。

```python
>>> a=[]  #定义了一个空的列表，变量a相当于一个贴在其上的标签
>>> type(a)  #用内置函数type()查看变量a引用对象的类型，为list
<type 'list'>
>>> bool(a) #用内置函数bool()查看a的布尔值，因为是空，所以为False
False
>>> print a
[]
```
下面看看list具体的使用

```python
>>> a=['a',3,'hello',True]  #list中可以有多中数据类型
>>> a
['a', 3, 'hello', True]
>>> type(a)
<type 'list'>
>>> bool(a)
True
>>> print a
['a', 3, 'hello', True]
>>> a[3]
True
>>> type(a[3])
<type 'bool'>
```


### 索引和切片

#### 索引

和字符串中的索引类似，只不过list是以元素为单位，而不是以字符为单位进行索引。

```python
>>> a=[2,3,True,"Hello,world"]
>>> a
[2, 3, True, 'Hello,world']
>>> a[0]
2
>>> a[1]
3
>>> a[2]
True
>>> a[3]
'Hello,world'
>>> type(a[0])
<type 'int'>
>>> type(a[2])
<type 'bool'>
>>> type(a[3])
<type 'str'>

>>> a.index(2)
0
>>> a.index(3)
1
>>> a.index(True)
2
>>> a.index("Hello,world")
3
>>> 

```

#### 切片

```python
>>> lst = ['python','java','c++']
>>> lst[-1]
'c++'
>>> lst[-3:-1]
['python', 'java']
>>> lst[-1:-3]
[]
>>> lst[0:3]
['python', 'java', 'c++']
>>> 
```
序列的切片，一定要左边的数字小于右边的数字，lst[-1:-3]就没有遵守这个规则，返回的是一个空。

### 反转

```python
>>> mList = [1,2,3,4,5,6]
>>> mList[::-1]
[6, 5, 4, 3, 2, 1]
>>> m
>>> mList
[1, 2, 3, 4, 5, 6]
>>> 

```
当然，也可以对字符串进行反转:

```python
>>> lang="python"
>>> lang[::-1]
'nohtyp'
>>> lang
'python'
```
可以看出，不管是str还是list，反转之后原来的值没有改变。这说明，这里的反转不是把原来的值倒过来，而是新生成了一个值，生成的值跟原来的值相比是倒过来的。

Python还有一种方法使list反转，且比较容易阅读和理解，特别推荐：

```python
mList = [1,2,3,4,5,5,6,8,7]
>>> list(reversed(mList))
[7, 8, 6, 5, 5, 4, 3, 2, 1]

>>> s='abcd'
>>> list(reversed(s))
['d', 'c', 'b', 'a']
'abcd'
```
### 对list的操作

#### len()

```python
>>> lst=['python','java','c++']
>>> len(lst)
3
```

#### +,连接连个序列

```python
>>> lst=['python','java','c++']
>>> lst
['python', 'java', 'c++']
>>> alst =[1,2,3,4,5]
>>> lst + alst
['python', 'java', 'c++', 1, 2, 3, 4, 5]
```

#### *,重复元素

```python
>>> lst * 3
['python', 'java', 'c++', 'python', 'java', 'c++', 'python', 'java', 'c++']
```
#### in

```python
>>> 'python' in lst
True
```
#### max()和min()

```python
>>> max(alst)
5
>>> min(alst)
1
>>> max(lst)
'python'
>>> min(lst)
'c++'
```
#### cmp()

```python
>>> a = [2,3]
>>> b = [2,4]
>>> cmp(a,b)
-1
>>> cmp(b,a)
1
>>> c=[2]
>>> cmp(a,c)
1
>>> d=['2']
>>> cmp(a,d)
-1
```
#### 追加元素

```python
>>> lst
['python', 'java', 'c++']
>>> lst.append("html5") #将元素追加到list的尾部
>>> lst
['python', 'java', 'c++', 'html5']
>>> lst.append(100)
>>> lst
['python', 'java', 'c++', 'html5', 100]
```
等价于：
```python
>>> lst
['python', 'java', 'c++', 'html5', 100]
>>> lst[len(lst):] = [3]
>>> lst
['python', 'java', 'c++', 'html5', 100, 3]
>>> len(lst)
6
>>> lst[6:]=['javascript']
>>> lst
['python', 'java', 'c++', 'html5', 100, 3, 'javascript']
```

### 列表的函数

list是Python中的苦力，那么它都有哪些函数呢？

```python
>>> dir(list)
['__add__', '__class__', '__contains__', '__delattr__', '__delitem__', '__delslice__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getslice__', '__gt__', '__hash__', '__iadd__', '__imul__', '__init__', '__iter__', '__le__', '__len__', '__lt__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__reversed__', '__rmul__', '__setattr__', '__setitem__', '__setslice__', '__sizeof__', '__str__', '__subclasshook__', 'append', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort']

```

先不管以双下划线开始和结尾的函数，就剩下以下几个

```
'append', 'count', 'extend', 'index', 'insert', 'pop', 'remove', 'reverse', 'sort'
```

#### append和extend

list.append(x),是将某个元素x添加到已知的一个列表的尾部。

list.extend(L),则是将两个列表合并，或者说将列表L追加到列表list中。

```python
>>> a=[1,2,3]
>>> b=["lau",3,4,5]
>>> a.extend(b)
>>> a
[1, 2, 3, 'lau', 3, 4, 5]
>>> b
['lau', 3, 4, 5]
```

```python
>>> a=[1,2,3]
>>> b="abc"
>>> a.extend(b)
>>> a
[1, 2, 3, 'a', 'b', 'c']
>>> c = 5
>>> a.extend(c) 

Traceback (most recent call last):
  File "<pyshell#20>", line 1, in <module>
    a.extend(c)
TypeError: 'int' object is not iterable
```
如果extend的参数对象是数值型则报错。
所以，extend的参数对象是一个list，如果是str，则Python会先把它按照字符为单位转化为list再添加到目标list中。官方文档指出extend的元素对象必须是iterable（可迭代的）。

**判断一个对象是不是可迭代的？**

```python
>>> s="python"
>>> hasattr(s,'__iter__')
False
>>> a=[1,2,3]
>>> hasattr(a,'__iter__')
True
```
通过内建函数hasattr()判断一个对象是不是可迭代的，它的判断本质就是看那个类型中是否有`__iter__`函数。

**append和extend函数的相同点**

```python
>>> a=[1,2,3]
>>> b=[4,5,6]
>>> id(a)
49822856
>>> id(b)
50066536
>>> a.extend(b)
>>> a
[1, 2, 3, 4, 5, 6]
>>> id(a)
49822856
```
```python
>>> a=[1,2,3]
>>> id(a)
50068376
>>> a.append(12)
>>> a
[1, 2, 3, 12]
>>> id(a)
50068376
```
由上面两段代码可知，append和extend函数的共同点：

-	都是原地址修改列表，不会创建一个新的列表
-	没有返回值

**append和extend函数的不同点**

```python
>>> lst=[1,2,3]
>>> lst.append(["python","java"])
>>> lst
[1, 2, 3, ['python', 'java']]
>>> len(lst)
4

>>> 
>>> lst2=[1,2,3]
>>> lst2.extend(["python","java"])
>>> lst2
[1, 2, 3, 'python', 'java']

>>> len(lst2)
5
```

可知，append是整体的追加，extend是单个的追加

#### count

count的作用是计算某个元素在该list中出现的次数

```python
>>> a=[1,2,1,1,1]
>>> a.count(1)
4
>>> a.append('a')
>>> a.append('a')
>>> a
[1, 2, 1, 1, 1, 'a', 'a']
>>> a.count('a')
2
>>> a.count(4) #不存在，则返回为0
0
```

#### index

index计算元素在该列表首次出现的位置

```python
>>> a=[1,2,3,4,4,5]
>>> a.index(4)
3
>>> a.index(6) #如果不存，就报错

Traceback (most recent call last):
  File "<pyshell#85>", line 1, in <module>
    a.index(6)
ValueError: 6 is not in list
```
#### insert

list.insert(i,x),想列表中指定的位置插入元素x。

```python
>>> a=[1,2,3,4,5]
>>> a.insert(0,0)
>>> a
[0, 1, 2, 3, 4, 5]
>>> a.insert(3,7)
>>> a
[0, 1, 2, 7, 3, 4, 5]
>>> a.insert(8,4)  #若超出列表长度，则将元素插入到列表末尾
>>> a
[0, 1, 2, 7, 3, 4, 5, 4]
```

#### pop和remove

删除列表中元素的方法有两个，分别是：

-	list.remove(x)

删除列表中首次出现的元素x，如果不存在元素x，则报错

```python
>>> a
[0, 1, 2, 7, 3, 4, 5, 4]
>>> a.remove(4)
>>> a
[0, 1, 2, 7, 3, 5, 4]
>>> a.remove(6)

Traceback (most recent call last):
  File "<pyshell#103>", line 1, in <module>
    a.remove(6)
ValueError: list.remove(x): x not in list
```

-	list.pop([i])

删除列表中位置i的元素并且返回该元素。如果没有指定位置i，则默认删除并返回列表的最后一个元素。

```python
>>> a.pop(0) #删除指定位置0
0
>>> a
[1, 2, 7, 3, 5, 4]
>>> a.pop() #删除默认位置
4  
>>> a
[1, 2, 7, 3, 5]

>>> a.pop(5) #超过列表长度则会报错

Traceback (most recent call last):
  File "<pyshell#116>", line 1, in <module>
    a.pop(5)
IndexError: pop index out of range
```
#### reverse

将列表的元素顺序反过来,不会创建新的列表，因此没有返回值

```python
>>> a=[1,2,3,4,5,6,7]
>>> a.reverse()
>>> a
[7, 6, 5, 4, 3, 2, 1]
>>> a=['ss','aaa','dd']

>>> a.reverse()
>>> a
['dd', 'aaa', 'ss']
```
#### sort

sort是对列表进行排序，不会创建新的列表，因此没有返回值

```python
>>> a=[1,3,9,7,5] 
>>> a.sort() #默认从小到大的排序
>>> a
[1, 3, 5, 7, 9]
>>> a.sort(reverse = True) #从大到小的排序
>>> a
[9, 7, 5, 3, 1]
```

```python
>>> lst=["python","java","c","basic","ruby"]
>>> lst.sort(key=len) #以字符串长度为关键词进行排序
>>> lst
['c', 'java', 'ruby', 'basic', 'python']
```

## 比较列表和字符串

### 相同点

-	都属于序列，因此那些属于序列的操作对两者都适用

### 区别

-	最大区别是，列表自身可以改变，字符串自身不可以改变
-	字符串只能通过创建一个新的str来改变，新的str不是原来的字符串对象

### 多维列表

-	在字符串中，每个元素只能是字符
-	在列表中，每个元素可以是任何类型

```python
>>> matrix=[[1,2,3],[4,5,6],[7,8,9]]
>>> type(matrix)
<type 'list'>
>>> matrix[0][1]
2
>>> matrix[0]
[1, 2, 3]
```
### 列表和字符串的相互转化

#### str.split()

```python
>>> line="hello I am good"
>>> line.split(" ")
['hello', 'I', 'am', 'good']
>>> a = line.split(" ")
>>> a
['hello', 'I', 'am', 'good']
>>> type(a)
<type 'list'>
```

#### "split".join(list)

```python
>>> a
['hello', 'I', 'am', 'good']
>>> type(a)
<type 'list'>
>>> " ".join(a)
'hello I am good'

```

## Python元组

### 定义

元组（tuple）是用圆括号括起来的，元素之间用逗号隔开。

```
>>> t = 123,'abc',["come","here"]
>>> t
(123, 'abc', ['come', 'here'])
>>> type(t)
<type 'tuple'>
```

元组也是一种序列，这一点与列表、字符串类似。有以下特点：

-	元组其中的元素不能更改，和字符串类似
-	元组中的元素可以是任何类型的数据，和列表类似

### 索引和切片

元组的索引和切片的基本操作和字符串、列表是一样的。

```python
>>> t
(123, 'abc', ['come', 'here'])
>>> t[2]
['come', 'here']
>>> t[1:]
('abc', ['come', 'here'])
>>> t[2][0]
'come'
```

特别注意：如果一个元组中只有一个元素，应该在该元素后面加上一个半角的英文逗号：

```python
>>> a=(3)
>>> type(a)
<type 'int'>
>>> a=(3,) #加了逗号就是元组
>>> type(a)
<type 'tuple'>
```

列表和元组之间可以实现转换，分别使用list()和tuple()

```python
>>> t
(123, 'abc', ['come', 'here'])
>>> tlst = list(t)   # tuple--->list
>>> tlst
[123, 'abc', ['come', 'here']]
>>> type(tlst)
<type 'list'>
>>> 
>>> t_tuple = tuple(tlst)  #list--->tuple
>>> t_tuple
(123, 'abc', ['come', 'here'])
>>> type(t_tuple)
<type 'tuple'>
```

### 用途

-	元组比列表操作速度快。如果定义了一个值的常量集，并且唯一要用它做的是不断地遍历它，请使用元组代替列表
-	如果对不需要修改的数据进行“写保护”，可以使代码更安全，这时使用元组而不是列表。如果必须要改变这些值，则需要执行元组到列表的转换。
-	元组可以在字典中被用作key，但是列表不行。因为字典的key必须是不可变的，元组本身是不可变的。
-	元组可以用在字符串格式化中。

