Python基础之(五)语句
====

# 运算符

## 算术运算符

前面已经讲过了四则运算，其中涉及到一些运算符：加减乘除，对应的符号分别是：+  -  *  /，此外，还有求余数的：%。这些都是算术运算符。其实，算术运算符不止这些。根据中学数学的知识，也应该想到，还应该有乘方、开方之类的。

下面列出一个表格，将所有的运算符表现出来。

|运算符|描述                |实例                |
|------|--------------------|--------------------|
|+     |加 - 两个对象相加   | 10+20 输出结果 30   |
|-     |减 - 得到负数或是一个数减去另一个数 | 10-20 输出结果 -10|
|*     |乘 - 两个数相乘或是返回一个被重复若干次的字符串 | 10 * 20 输出结果 200|
|/     |除 - x除以y         | 20/10 输出结果 2 |
|%     |取余 - 返回除法的余数 | 20%10 输出结果 0 |
|**    |幂 - 返回x的y次幂   | 10**2 输出结果 100 |
|//    |取整除 - 返回商的整数部分  | 9//2 输出结果 4 , 9.0//2.0 输出结果 4.0 |

## 比较运算符

以下假设`a=10`，`b=20`：

| 运算符 | 描述 | 实例 |
|--------|------|------|
|== | 等于 - 比较对象是否相等 | (a == b) 返回 False。|
|!= | 不等于 - 比较两个对象是否不相等 | (a != b) 返回 True. |
|>  | 大于 - 返回x是否大于y | (a > b) 返回 False。|
|<  | 小于 - 返回x是否小于y | (a < b) 返回 True。|
|>= | 大于等于 - 返回x是否大于等于y。| (a >= b) 返回 False。|
|<= | 小于等于 - 返回x是否小于等于y。| (a <= b) 返回 True。|

## 逻辑运算符

（假设`a=10`，`b=20`）

|运算符 | 描述  |  实例 |
|-------|-------|-------|
|and    | 布尔"与" - 如果x为False，x and y返回False，否则它返回y的计算值。| (a and b) 返回 20。|
|or     | 布尔"或" - 如果x是True，它返回True，否则它返回y的计算值。| (a or b) 返回 10。|
|not    | 布尔"非" - 如果x为True，返回False。如果x为False，它返回True。 | not(a and b) 返回 false。|

- and

and，翻译为“与”运算，但事实上，这种翻译容易引起望文生义的理解。

先说一下正确的理解。

`A and B`，含义是：

首先运算A，如果A的值是True，就计算B，并将B的结果返回做为最终结果，如果B是False，那么`A and B`的最终结果就是False，如果B的结果是True，那么A and B的结果就是True；

如果A的值是False ，就不计算B了，直接返回`A and B`的结果为False.

# 简单语句

## print

在Python 2中，print是一个语句，但是在Python 3中它是一个函数了。这点请注意。

以Python 2为例，说明print语句。如果说读者使用的是Python 3，请自行将print xxx修改为print(xxx)，其它不变。

print发起的语句，在程序中主要是将某些东西打印出来，还记得在讲解字符串的时候，专门讲述了字符串的格式化输出吗？那就是用来print的。

```python
>>> print "hello, world"
hello, world
>>> print "hello","world" //逗号表示打印在同一行
hello world
```
本来，在print语句中，字符串后面会接一个\n符号。即换行。但是，如果要在一个字符串后面跟着逗号，那么换行就取消了，意味着两个字符串"hello"，"world"打印在同一行。

但是，在Python 3中情况有变。默认的end='\n'，如果不打算换行，可以在使用print()函数的时候，修改end这个参数的值。

```python
>>> for i in [1,2,3,4]:
    print(i, end=',')

1,2,3,4,
```

## import

import引入模块的方法，是Python编程经常用到的。引用方法有如下几种：

```python
>>> import math
>>> math.pow(3,2)
9.0
```
这是一种可读性非常好的引用方式，并且不同模块的同名函数不会产生冲突。

```python
>>> from math import pow
>>> pow(3,2)
9.0
```
这种引用方法，比较适合于引入模块较少的时候。如果引入模块多了，可读性就下降了，会不知道那个函数来自那个模块。

```python
>>> from math import pow as pingfang
>>> pingfang(3,2)
9.0
```
将从某个模块引入的函数重命名，比如讲pow充命名为pingfang，然后使用pingfang()就相当于在使用pow()了

```python
>>> from math import pow, e, pi
>>> pow(e,pi)
23.140692632779263
```
引入了math模块里面的pow,e,pi，pow()是一个乘方函数，e是那个欧拉数；pi就是π.

```python
>>> from math import *
>>> pow(3,2)
9.0
>>> sqrt(9)
3.0
```
一下将math中的所有函数都引过来了。不过，这种方式的结果是让可读性更降低了。仅适用于模块中的函数比较少的时候，并且在程序中应用比较频繁。

事实上，不仅函数可以引入，模块中还可以包括常数等，都可以引入。在编程中，模块中可以包括各样的对象，都可以引入。

## 赋值

```python
a = 3
```

```python
>>> x, y, z = 1, "python", ["hello", "world"]
>>> x
1
>>> y
'python'
>>> z
['hello', 'world']
```

```Python
>>> a = "itdiffer.com", "python"
>>> a
('itdiffer.com', 'python')
```
原来是将右边的两个值装入了一个元组，然后将元组赋给了变量a。

两个变量的值对调,python只要一行就完成了:

```python
>>> a = 2
>>> b = 9
>>> a, b = b, a
>>> a
9
>>> b
2
```
因为我前面已经数次提到的Python中变量和对象的关系。变量相当于贴在对象上的标签。这个操作只不过是将标签换个位置，就分别指向了不同的数据对象。

还有一种赋值方式，被称为“链式赋值”

```python
>>> m = n = "I use python"
>>> print m, n            #Python 3：print(m, n)
I use python I use python
```

用这种方式，实现了一次性对两个变量赋值，并且值相同。

```python
>>> id(m)
3072659528L
>>> id(n)
3072659528L
```

用`id()`来检查一下，发现两个变量所指向的是同一个对象。

另外，还有一种判断方法，来检查两个变量所指向的值是否是同一个（注意，同一个和相等是有差别的。在编程中，同一个就是`id()`的结果一样。

```python
>>> m is n
True
```

这是在检查m和n分别指向的对象是否是同一个，True说明是同一个。

```python
>>> a = "I use python"
>>> b = a
>>> a is b
True
```

这是跟上面链式赋值等效的。

但是：

```python
>>> b = "I use python"
>>> a is b
False
>>> id(a)
3072659608L
>>> id(b)
3072659568L

>>> a == b
True
```

看出其中的端倪了吗？这次a、b两个变量虽然相等，但不是指向同一个对象。

还有一种赋值形式，如果从数学的角度看，是不可思议的，如：`x = x + 1`，在数学中，这个等式是不成立的。因为数学中的“=”是等于的含义，但是在编程语言中，它成立，因为"="是赋值的含义，即将变量x增加1之后，再把得到的结果赋值变量x.

这种变量自己变化之后将结果再赋值给自己的形式，称之为“增量赋值”。``+、-、*、/、``%都可以实现类似这种操作。
为了让这个操作写起来省点事（要写两遍同样一个变量），可以写成：`x += 1`

```python
>>> x = 9
>>> x += 1
>>> x
10
```

除了数字，字符串进行增量赋值，在实际中也很有价值。

```python
>>> m = "py"
>>> m += "th"
>>> m
'pyth'
>>> m += "on"
>>> m
'python'
```

# 条件语句

## if语句

```python
>>> a = 8
>>> if a == 8: //冒号是必须的
...     print a //四个空格的缩进是必须的
...
8
>>>
```

*	必须要通过缩进方式来表示语句块的开始和结束
* 缩进用四个空格也是必须的


## if...elif...else

```python
#coding:utf-8
print "请输入一个整数数字:"
number = int(raw_input())

if number == 10:
    print "您输入的数字是： %d" % number
    print "You are smart"
elif number > 10:
    print "您输入的数字是： %d" % number
    print "This number is more than 10."
elif number < 10:
    print "您输入的数字是： %d" % number
    print "This number is less than 10."
else:
    print "Are you a human?"
    
结果打印：

请输入一个整数数字:
12
您输入的数字是： 12
This number is more than 10.

请输入一个整数数字:
10
您输入的数字是： 10
You are smart

请输入一个整数数字:
9
您输入的数字是： 9
This number is less than 10.
```

## 三元操作符

```python
>>> name = "liuguoquan" if "deason" else "xiaoniu"
>>> name
'liuguoquan'
>>> name = "liuguoquan" if "" else "xiaoniu"
>>> name
'xiaoniu'
>>> name = "liuguoquan" if "xiaoniu" else ""
>>> name
'liuguoquan'
```
总结一下：A = Y if X else Z
根据上述例子可以看出：
*	如果X为真，那么就执行A = Y
* 如果X为假，那么就执行A = Z

```python
>>> a = "python" if x > y else "java"
>>> a
'java'
>>> a = "python" if x < y else "java"
>>> a
'python'
>>>
```

# for循环

For循环的基本结构是：

for 循环规则： //需要冒号
	 操作语句		//语句要缩进

## 简单的for循环

```python
>>> hello = "liuguoquan"
>>> for i in hello:
...     print i
...
l
i
u
g
u
o
q
u
a
n

or:

>>> for i in range(len(hello)):
...     print i
...
0
1
2
3
4
5
6
7
8
9

```

## range（start,stop,[,step])

range是个内建函数，一般形式是range（start,stop,[,step])

关于range()函数注意一下几点：

*	该函数可以创建一个数字元素组成的列表
* 该函数最常用于for循环
* 函数的参数必须是整数，默认从0开始
* step默认值是1，不写就是默认值
* step是正数，返回list最后的值不包含stop的值，即start + step的值小于stop；step是负数，start + step的值大于stop；
* step不能等于0

参数解释：

*	start：开始数值，默认为0
* stop：结束数值，必须要写
* step：变化的步长，默认是1，不能为0

正数

```python
>>> range(9)
[0, 1, 2, 3, 4, 5, 6, 7, 8]
>>> range(0,9)
[0, 1, 2, 3, 4, 5, 6, 7, 8]
>>> range(0,9,1)
[0, 1, 2, 3, 4, 5, 6, 7, 8]
>>> range(1,9)
[1, 2, 3, 4, 5, 6, 7, 8]
>>> range(1,9,2)
[1, 3, 5, 7]
```
负数

```python
>>> rang(-9)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'rang' is not defined
>>> range(0,-9,-1)
[0, -1, -2, -3, -4, -5, -6, -7, -8]
>>> range(0,-9,-2)
[0, -2, -4, -6, -8]
>>>
```

### 获取列表的索引值组成的列表

```python
>>> lgq = ["I","am","a","Android"]
>>> index = range(len(lgq))
>>> index
[0, 1, 2, 3]
>>>
```

### 找出100以内的能够被3整除的正整数

```python
#coding:utf-8
result = []
for n in range(1,100):
    if n % 3 == 0:
        result.append(n)
        
print result

or:

print range(3,100,3)

结果打印：

[3, 6, 9, 12, 15, 18, 21, 24, 27, 30, 33, 36, 39, 42, 45, 48, 51, 54, 57, 60, 63, 66, 69, 72, 75, 78, 81, 84, 87, 90, 93, 96, 99]
```

## for对象

```python
 #coding:utf-8

name = "liuguoquan"

print "str"
for i in name:
    print i,

print "list"
name_list = list(name)
print name_list
for i in name_list:
    print i,
    
print "set"
name_set = set(name)
print name_set
for i in name_set:
    print i,
    
print "tuple"
name_tuple = set(name)
print name_tuple
for i in name_tuple:
    print i,
    
print "dict"
name_dict = {"name":"liuguoquan","sex":"male","age":"18"}
print name_dict
for i in name_dict:
    print i,"-->",name_dict[i]
    
 结果打印:
 
str:
l i u g u o q u a n 

list:
['l', 'i', 'u', 'g', 'u', 'o', 'q', 'u', 'a', 'n']
l i u g u o q u a n 

set:
set(['a', 'g', 'i', 'l', 'o', 'n', 'q', 'u'])
a g i l o n q u 

tuple
('l', 'i', 'u', 'g', 'u', 'o', 'q', 'u', 'a', 'n')
l i u g u o q u a n 

dict:
{'age': '18', 'name': 'liuguoquan', 'sex': 'male'}
age --> 18
name --> liuguoquan
sex --> male
```
### 字典遍历的多种写法

```python
name_dict = {"name":"liuguoquan","sex":"male","age":"18"}
print name_dict
for i in name_dict:
    print i,"-->",name_dict[i]
```

注意到，上面的循环，其实是读取了字典的key。在字典中，有一个方法，dict.keys()，得到的是字典key列表。

```python
#获取字典key列表
for k in name_dict.keys():
    print k    #Python 3: print(k)
```

这种循环方法和上面的循环方法，结果是一样的，但是，这种方法并不提倡，以为它在执行速度上表现欠佳。

如果要获得字典的value怎么办？不要忘记dict.values()方法

```python
for k,v in name_dict.items():
    print k,"-->",v
```
用上面的方法，要把所有的内容都读入内存，内存东西多了，可能会出麻烦。为此，Python中提供了另外的方法。

```python
for k,v in name_dict.iteritems(): #注意：仅在Python2中可用，Python 3中已经做了优化，d.items()即有同等功能。
    print k,"-->",v
```

这里是循环一个迭代器，迭代器在循环中有很多优势。除了刚才的dict.iteritems()之外，还有dict.itervalues()，dict.iterkeys()供你选用（以上三个dict.iter*都只用在Python 2中，Python 3中已经不需要了）

### 如何判断一个对象是不可迭代的？

```python
>>> import collections
```

引入collections这个标准库。要判断数字321是不是可迭代的，可以这么做：

```python
>>> isinstance(321, collections.Iterable)
    False
 ```

返回了False，说明321这个整数类型的对象，是不可迭代的。再判断一个列表对象。

```python
 >>> isinstance([1,2,3], collections.Iterable)
    True
```

从返回结果，我们知道，列表`[1,2,3]`是可迭代的。

当然，并不是要你在使用for循环之前，非要判断某个对象是否可迭代。因为至此，你已经知道了字符串str、列表list、字典dict、元组tuple、集合set都是可迭代的。

## zip()

`zip()`——一个内建函数,Python 2中，参数是`seq1, seq2, ...`，意思是序列数据；在Python 3中，参数需要时可迭代对象。这点差别，通常是没有什么影响的，因为序列也是可迭代的。值得关注的是返回值，在Python 2中，返回值是一个列表对象，里面以元组为元素；而Python 3中返回的是一个zip对象。
通过实验来理解上面的文档：

```python
Python 2：
    >>> a = "qiwsir"
    >>> b = "github"
    >>> zip(a, b)
    [('q', 'g'), ('i', 'i'), ('w', 't'), ('s', 'h'), ('i', 'u'), ('r', 'b')]
    
Python 3：
    >>> zip(a, b)
    <zip object at 0x0000000003521D08>
    >>> list(zip(a, b))
    [('q', 'g'), ('i', 'i'), ('w', 't'), ('s', 'h'), ('i', 'u'), ('r', 'b')]
```

如果序列长度不同，那么就以"the length of the shortest argument sequence"为准。

```python
>>> c = [1, 2, 3]
>>> d = [9, 8, 7, 6]
>>> zip(c, d)        #这是Python 2的结果，如果是Python 3，请仿照前面的方式显示查看
[(1, 9), (2, 8), (3, 7)]
```

```python
>>> s = {"name":"qiwsir"}
>>> t = {"lang":"python"}
>>> zip(s,t)
[('name', 'lang')]
```

zip是一个内置函数，它的参数必须是序列，如果是字典，那么键视为序列。然后将序列对应的元素依次组成元组，做为一个列表的元素。

### **问题：**有两个列表，分别是：a = [1, 2, 3, 4, 5], b = [9, 8, 7, 6, 5]，要计算这两个列表中对应元素的和。

*	for循环实现

```python
>>> a = [1, 2, 3, 4, 5]
>>> b = [9, 8, 7, 6, 5]
>>> c = []
>>> for i in range(len(a)):
...     c.append(a[i]+b[i])
... 
>>> c
[10, 10, 10, 10, 10]
```

*	zip()实现

```python
>>> d = []
>>> for x,y in zip(a,b):
...     d.append(x+y)
... 
>>> d
[10, 10, 10, 10, 10]
```

### **问题**：有一个字典，myinfor = {"name":"qiwsir", "site":"qiwsir.github.io", "lang":"python"}，将这个字典变换成：infor = {"qiwsir":"name", "qiwsir.github.io":"site", "python":"lang"}

*	for循环

```python
>>> myinfor = {"name":"qiwsir", "site":"qiwsir.github.io", "lang":"python"}
>>> infor = {}
>>> for k,v in myinfor.items():
...     infor[v]=k
... 
>>> infor
{'python': 'lang', 'qiwsir.github.io': 'site', 'qiwsir': 'name'}
```

*	zip()

```python
>>> dict(zip(myinfor.values(), myinfor.keys()))
{'python': 'lang', 'qiwsir.github.io': 'site', 'qiwsir': 'name'}
```

## enumerate()

`enumerate()`也是内建函数。

```python
>>> seasons = ['Spring', 'Summer', 'Fall', 'Winter']
>>> list(enumerate(seasons))
[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
    
>>> list(enumerate(seasons, start=1))
[(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]
```

### 如果要同时得到元素索引和元素怎么办？

*	for循环

```python
week = ["monday","sunday","friday"]
for i in range(len(week)):
    print week[i] + ' is ' + str(i) #注意，i是int类型，如果和前面的用+连接，必须是str类型
```
* enumerate

```python
for (i,day) in enumerate(week):
    print day + " is " + str(i)
```

### **问题：**将字符串中的某些字符替换为其它的字符串。原始字符串"Do you love Canglaoshi? Canglaoshi is a good teacher."，请将"Canglaoshi"替换为"PHP".

```python
raw = "Do you love Canglaoshi? Canglaoshi is a good teacher."
print raw

#1. 先将字符串转换为列表
raw_list = raw.split(" ")
print type(raw_list)
print raw_list

#2. 对列表中的字符串进行替换
for i,string in enumerate(raw_list):
    if "Canglaoshi" in string:
        raw_list[i] = "PHP"
        
print raw_list

#3. 将列表转换为字符串
print " ".join(raw_list)
```

## 列表解析

Python有一个非常强大的功能，就是列表解析，它这样使用：

```python
>>> squares = [x**2 for x in range(1, 10)]
>>> squares
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

```python
>>> mybag = [' glass',' apple','green leaf ']   #有的前面有空格，有的后面有空格
>>> [one.strip() for one in mybag]              #去掉元素前后的空格
['glass', 'apple', 'green leaf']
```

在很多情况下，列表解析的执行效率高，代码简洁明了。是实际写程序中经常被用到的。
现在Python的两个版本，对列表解释上，还是有一点点差别的，请认真看下面的比较操作。

```python
Python 2:

    >>> i = 1
    >>> [ i for i in range(9)]
    [0, 1, 2, 3, 4, 5, 6, 7, 8]
    >>> i
    8

Python 3:

    >>> i = 1
    >>> [i for i in range(9)]
    [0, 1, 2, 3, 4, 5, 6, 7, 8]
    >>> i
    1
```
先`i = 1`，然后是一个列表解析式，非常巧合的是，列表解析式中也用了变量`i`。这种情况，在编程中是常常遇到的，我们通常把`i=1`中的变量`i`称为处于全局命名空间里面（命名空间，是一个新词汇，暂且用起来，后面会讲述），而列表解析式中的变量`i`是在列表解析内，称为处在局部命名空间。在Python 3中，for循环里的变量不再与全局命名空间的变量有关联了。

## while语句

### while

```python
while i< 5:
    print i
    i += 1
print "quit"

0
1
2
3
4
quit
```

### break

```python
i= 10
while i:
    if i % 2 == 0:
        break
    else:
        print "%d is odd number" %i
        i = 0
print "%d is even number" %i

10 is even number
```

### continue

```python
i= 10
while i:
    if i % 2 == 0:
        i -= 1
        continue; # 继续下一次循环
    else:
        print "%d is odd number" %i
        i -= 1
print "%d is even number" %i
```

### while...else

`while...else`有点类似`if ... else`，只需要一个例子就可以理解。 当然，一遇到`else`了，就意味着已经不在while循环内了。

```python
count = 0
while count < 5:
   print count, " is  less than 5"
   count = count + 1
else:
   print count, " is not less than 5"
   
执行结果：

    0 is less than 5
    1 is less than 5
    2 is less than 5
    3 is less than 5
    4 is less than 5
    5 is not less than 5
```

### for...else

除了有`while...else`外，还可以有`for...else`。这个循环也通常用在当跳出循环之后要做的事情。

```python
from math import sqrt

    for n in range(99, 1, -1):
        root = sqrt(n)
        if root == int(root):
            print n # 满足此条件后,不执行else
            break

    else:
        print "Nothing." # break不执行就会执行else
```

