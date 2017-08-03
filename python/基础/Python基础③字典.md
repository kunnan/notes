[TOC]

## 定义

Python中有一个叫作dictionary的对象类型，翻译过来就是“字典”，用dict表示。

## 创建字典

### 创建空的字典

```python
>>> mydict = {}
>>> mydict
{}
>>> type(mydict)
<type 'dict'>
>>> person={"name":"liu","sex":"male","age":10}
>>> person
{'age': 10, 'name': 'liu', 'sex': 'male'}
```
字典dict是以键值对的形式存储数据。比如`"name":"liu"`前面的name叫作键（key），后面的liu是前面的键所对应的值（value）。在一个字典中，键是唯一的，不能重复。值则对应于键，值可以重复。

键值之间用冒号隔开，每一对键值之间用逗号隔开。

### 添加字典

```python
>>> person
{'age': 10, 'name': 'liu', 'sex': 'male'}
>>> person["hobby"]="reading" #增加键值对的方法
>>> person
{'hobby': 'reading', 'age': 10, 'name': 'liu', 'sex': 'male'}
```
### 修改字典

```python
>>> person["age"]=20
>>> person
{'hobby': 'reading', 'age': 20, 'name': 'liu', 'sex': 'male'}
>>> id(person)
49368784
>>> person["age"]=30
>>> person
{'hobby': 'reading', 'age': 30, 'name': 'liu', 'sex': 'male'}
>>> id(person)
49368784
```
结果可以看出，字典可以原地地修改，即它是可变的，并且不会创建新的对象。

### 利用元组创建字典

```python
>>> name = (["first","Google"],["second","Yahoo"])
>>> website=dict(name)
>>> website
{'second': 'Yahoo', 'first': 'Google'}
>>> type(website)
<type 'dict'>

或者这样：

>>> ad = dict(name="liu",age= 26)
>>> ad
{'age': 26, 'name': 'liu'}
>>> type(ad)
<type 'dict'>
```

### 使用fromkeys创建字典

```python
>>> website={}.fromkeys(("third","forth"),("facebook","amazon"))
>>> website
{'forth': ('facebook', 'amazon'), 'third': ('facebook', 'amazon')}
>>> website={}.fromkeys(("third","forth"),"facebook")
>>> website
{'forth': 'facebook', 'third': 'facebook'}
```

特别注意的是，字典中的键必须是不可变对象，值可以是任意类的对象。

```python
>>> dd = {(1,2):1}
>>> dd
{(1, 2): 1}
>>> type(dd)
<type 'dict'>

>>> dd={[1,2]:1}

Traceback (most recent call last):
  File "<pyshell#52>", line 1, in <module>
    dd={[1,2]:1}
TypeError: unhashable type: 'list'
```

## 访问字典的值

字典类型的对象是以键值对的形式存储数据的，所以只要知道键就能得到值，这在本质上就是一种映射关系。

```python
>>> person
{'hobby': 'reading', 'age': 30, 'name': 'liu', 'sex': 'male'}
>>> person["age"]
30
>>> person["hobby"]
'reading'
```

## 基本操作

### len(d)

返回字典d中键值对的数量

```python
>>> city_code = {"beijing":"010","shanghai":"021","guangzhou":"020"}
>>> len(city_code)
3
```

### d[key]

返回字典d中的键key的值

```python
>>> city_code
{'beijing': '010', 'shanghai': '021', 'guangzhou': '020'}
>>> city_code["beijing"]
'010'
```

### d[key] = value

将值赋给字典d中的键key

```python
>>> city_code
{'beijing': '010', 'shanghai': '021', 'guangzhou': '020'}
>>> city_code["beijing"]="01110"
>>> city_code
{'beijing': '01110', 'shanghai': '021', 'guangzhou': '020'}
```

### del d[key]

删除字典d中的键key值对

```python
>>> city_code
{'beijing': '01110', 'shanghai': '021', 'guangzhou': '020'}
>>> del city_code["beijing"]
>>> city_code
{'shanghai': '021', 'guangzhou': '020'}
```

### key in d

检查字典d中是否含有键为key的项

```python
>>> city_code
{'shanghai': '021', 'guangzhou': '020'}
>>> "shanghai" in city_code
True
>>> "beijing" in city_code
False
```

## 字符串格式化输出

```python
>>> city_code
{'shanghai': '021', 'guangzhou': '020'}
>>> "Shanghai is a wonderful city,its area code is %(shanghai)s" % city_code
'Shanghai is a wonderful city,its area code is 021'
```

## 相关概念

### 关联数组

在计算机科学中，关联数组又称为映射（Map）、字典（Dictionary），是一个抽象的数据结构，它包含着类似于键值的有序对。这种数据结构包含以下几种常见的操作：

-	向关联数组添加键值对
-	从关联数组内删除键值对
-	修改关联数组内的键值对
-	根据已知的键寻找值

字典问题是设计一种能够具备关联数组特性的数据结构。解决字典问题的常用方法是散列表，但有些情况也可以直接使用有地址的数组、二叉树，或者其他结构。

### 散列表

散列表（hash table，也叫哈希表），是根据关键字而直接访问在内存存储位置的数据结构。即把键值通过一个函数的计算，映射到表中的一个位置来访问记录，加快了查找速度。这个映射函数称为散列函数，存放记录的数组称为散列表。


## 字典的函数

### 拷贝

浅拷贝：Python在所执行的复制动作中，如果是基本类型的数据，就在内存中新建一个地址存储，如果不是基本类型，就不会新建一个地址，而是用标签引用原来的对象。copy()实现的是浅拷贝

深拷贝：无论是基本数据类型还是其他类型都会新建一个地址来存储。deepcopy()实现的是深拷贝

#### copy()

```python
>>> ad={"name":"liu","age":20}
>>> id(ad)
49368640
>>> cd= ad.copy()
>>> cd
{'age': 20, 'name': 'liu'}
>>> id(cd)  #与ad是不同的对象
49225776
>>> cd["age"]=40  #修改cd没有对ad造成影响
>>> cd
{'age': 40, 'name': 'liu'}
>>> ad
{'age': 20, 'name': 'liu'}

>>> x={"name":"liu","lang":["python","java","c"]}
>>> y = x.copy()
>>> y
{'lang': ['python', 'java', 'c'], 'name': 'liu'}
>>> id(x)
49226496
>>> id(y)
49228800

##y是从x拷贝过来的，两个在内存中是不同的对象

>>> y["lang"].remove("c")
>>> y
{'lang': ['python', 'java'], 'name': 'liu'}
>>> x
{'lang': ['python', 'java'], 'name': 'liu'}
>>> id(x)
49226496
>>> id(y)
49228800

y中删除元素“c“后，x的键lang的值也发生了变化

>>> id(x["lang"])
49607424
>>> id(y["lang"])
49607424

x与y中的列表是同一个对象，但是作为字符串的那个键值对属于不同的对象。
```

#### deepcopy()

```python
>>> x
{'lang': ['python', 'java'], 'name': 'liu'}
>>> import copy #导入一个模块
>>> z = copy.deepcopy(x) #深拷贝
>>> z
{'lang': ['python', 'java'], 'name': 'liu'}
>>> id(x["lang"])
49607424
>>> id(z["lang"])
49607184

#此时x和z中的列表不属于同个对象

>>>
>>> x["lang"].remove("java")
>>> x
{'lang': ['python'], 'name': 'liu'}
>>> z
{'lang': ['python', 'java'], 'name': 'liu'}

#此时，修改一个列表的值，就不会影响另外一个列表的值了。
```

### clear

clear的作用是将字典清空，得到的是空字典。del是将字典删除，内存中就没有它了。

```python
>>> x
{'lang': ['python'], 'name': 'liu'}
>>> x.clear()
>>> x
{}
>>> del a

>>> del x
>>> x  #x已删除

Traceback (most recent call last):
  File "<pyshell#147>", line 1, in <module>
    x
NameError: name 'x' is not defined
>>>
```
如果要清空一个字典，还能够使用x = {}这种方法，但这种方法的本质是将变量a的引用转向了{}这个对象，那么原来的对象呢？原来的对象称为了断了线的风筝，这样的东西称为在Python中称为垃圾，而且Python能够自动将这样的垃圾回收。

### get和setdefault

得到字典中的某个值

#### get

```python
>>> z
{'lang': ['python', 'java'], 'name': 'liu'}
>>> z["lang"].remove("java")
>>> z
{'lang': ['python'], 'name': 'liu'}
>>> 
>>> z.get("name")
'liu'

>>> print z.get("age") #返回None
None
>>> d["age"]  #抛出异常
Traceback (most recent call last):
  File "<pyshell#160>", line 1, in <module>
    d["age"]
NameError: name 'd' is not defined

>>> z.get("age",30) #如果age不存在，则返回默认值30
30

```
dict.get()和dict[key]的区别在于：如果键不在字典中，前者返回None，后者抛出异常

#### setdefault

```python
>>> z
{'lang': ['python'], 'name': 'liu'}
>>> z.setdefault("age",30)  #如果age不存在，则返回默认值30，同时将这对键值对添加到字典中
30
>>> z
{'lang': ['python'], 'age': 30, 'name': 'liu'}

>>> z.setdefault("web")
>>> z
{'lang': ['python'], 'web': None, 'age': 30, 'name': 'liu'}
```

### items/iteritems,keys/iterkeys,values/itervalues

```python
>>> dd = {"name":"liu","age":40,"websit":"www.liuguoquan.com"}
>>> dd_kv = dd.items()
>>> dd_kv
[('websit', 'www.liuguoquan.com'), ('age', 40), ('name', 'liu')]
>>> type(dd_kv)
<type 'list'>
>>> list(dd_kv)
[('websit', 'www.liuguoquan.com'), ('age', 40), ('name', 'liu')]

>>> dd_iter=dd.iteritems()
>>> type(dd_iter)
<type 'dictionary-itemiterator'>
>>> dd_iter
<dictionary-itemiterator object at 0x02F5F420>
>>> list(dd_iter)  #必须用list()迭代遍历
[('websit', 'www.liuguoquan.com'), ('age', 40), ('name', 'liu')]

>>> dd
{'websit': 'www.liuguoquan.com', 'age': 40, 'name': 'liu'}
>>> dd.keys() #列出键的列表
['websit', 'age', 'name']

>>> dd.values() #列出值的列表
['www.liuguoquan.com', 40, 'liu']
```

### pop和popitem

删除字典键值对

```python
>>> dd
{'websit': 'www.liuguoquan.com', 'age': 40, 'name': 'liu'}
>>> dd.pop("name") #删除指定键
'liu'

>>> dd
{'websit': 'www.liuguoquan.com', 'age': 40}
>>> dd.pop() #参数不能省略

Traceback (most recent call last):
  File "<pyshell#198>", line 1, in <module>
    dd.pop()
TypeError: pop expected at least 1 arguments, got 0


>>> dd
{'websit': 'www.liuguoquan.com', 'age': 40}
>>> dd.popitem() #随机删除一个键值对
('websit', 'www.liuguoquan.com')

>>> dd
{'age': 40}
>>> dd.popitem()
('age', 40)

>>> dd
{}
```
### update

更新字典

```python
>>> d1={"name":"liu"}
>>> d2={"age":20}
>>> d1.update(d2) #将字典到d2添加到d1中，更新字典
>>> d1
{'age': 20, 'name': 'liu'}
>>> d2
{'age': 20}

>>> d2.update([("name","lee"),("lang","python")]) #更新字典
>>> d2
{'lang': 'python', 'age': 20, 'name': 'lee'}
```

### has_key

判断字典中是否存在某个键

```python
>>> d2
{'lang': 'python', 'age': 20, 'name': 'lee'}
>>> d2.has_key("name")
True
>>> d2.has_key("web")
False
```





