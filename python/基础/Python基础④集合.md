[TOC]

集合Set类似字典的特点，可以用{}花括号来定义；其中的元素是没有序列，也就是非序列类型的数据；而且集合中的元素不可重复，这就类似于dict键。

# 创建集合

```python
>>> s1 = set("qiswri") #有两个i
>>> s1
set(['q', 'i', 's', 'r', 'w']) #只有一个i
>>> 
>>> s2 = set([123,"google","facebook","book","facebook"]);
>>> s2
set(['facebook', 123, 'google', 'book']) #只有一个facebook
```
说明集合中的元素是不能重复的，在创建集合的时候，如果发现了重复的元素，就会自定过滤重复的元素。并且集合中的元素也是随机排序的。

除了用set()来创建集合，还可以使用{}的方式，但是这种方式不提倡使用，因为在某些情况下，Python搞不清楚是字典还是集合。

unhashtable（不可哈希的）表示可变数据，如列表和字典都能原地修改，就是unhastable的；
hastable（可哈希）表示不可变数据，如字符串不能修改。

```python
>>> s1
set(['q', 'i', 's', 'r', 'w'])
>>> s1[1] = "w"   #集合不是序列类型，不能用索引方式对其进行修改

Traceback (most recent call last):
  File "<pyshell#9>", line 1, in <module>
    s1[1] = "w"
TypeError: 'set' object does not support item assignment
```
list()和set()实现集合和列表两种对象之间的转化。

```python
>>> s2
set(['facebook', 123, 'google', 'book'])
>>> type(s2)
<type 'set'>
>>> lst = list(s2)
>>> lst
['facebook', 123, 'google', 'book']
>>> type(lst)
<type 'list'>
>>> s3 = set(lst)
>>> s3
set(['google', 123, 'facebook', 'book'])
```

# 集合的函数

dir(set)：列出集合的函数

## add和update

help(set.add)

```python
>>> s1 = set(["a","b","c"])
>>> s1
set(['a', 'c', 'b'])
>>> s1.add("abc") #向集合中添加元素
>>> s1
set(['a', 'c', 'b', 'abc'])

>>> s1.add([1,2,3]) #不能添加列表，因为列表的数据是可变的

Traceback (most recent call last):
  File "<pyshell#31>", line 1, in <module>
    s1.add([1,2,3])
TypeError: unhashable type: 'list'
>>> 
>>> s1
set(['a', 'c', 'b', 'abc'])
>>> s2 = set(["e","f","g"])
>>> s1.update(s2) #集合s2添加到s1中
>>> s1
set(['a', 'c', 'b', 'e', 'abc', 'g', 'f'])
>>> s2
set(['e', 'g', 'f'])
```
## pop、remove、discard、clear

```python
>>> s1
set(['a', 'c', 'b', 'e', 'abc', 'g', 'f'])
>>> s1.pop() #随机删除一个元素，并返回这个元素
'a'
>>> s1.pop()
'c'
>>> s1.pop("b")  #不能指定参数

Traceback (most recent call last):
  File "<pyshell#43>", line 1, in <module>
    s1.pop("b")
TypeError: pop() takes no arguments (1 given)
>>> 
>>> s1.remove("b") #删除集合中指定的元素
>>> s1
set(['e', 'abc', 'g', 'f'])
>>> s1.remove()  #remove必须指定参数

Traceback (most recent call last):
  File "<pyshell#47>", line 1, in <module>
    s1.remove()
TypeError: remove() takes exactly one argument (0 given)
>>> s1.discard("g") #存在则删除集合中的元素
>>> s1
set(['e', 'abc', 'f'])
>>> s1.discard("i") #不存在则不做处理，不抛出异常
>>>
>>> s1.clear()  #清空集合
>>> s1
set([])
```

# 不变的集合

```python
>>> f1 = frozenset("qwert")
>>> f1
frozenset(['q', 'r', 'e', 't', 'w'])
>>> f1.add("q") #不能修改集合

Traceback (most recent call last):
  File "<pyshell#62>", line 1, in <module>
    f1.add("q")
AttributeError: 'frozenset' object has no attribute 'add'
>>> f1.remove("r")

Traceback (most recent call last):
  File "<pyshell#63>", line 1, in <module>
    f1.remove("r")
AttributeError: 'frozenset' object has no attribute 'remove'
>>> f1.pop()

Traceback (most recent call last):
  File "<pyshell#64>", line 1, in <module>
    f1.pop()
AttributeError: 'frozenset' object has no attribute 'pop'
```

# 集合元素

## 元素与集合的关系

```python
>>> aset=set(["q","w","e","r","t"])
>>> aset
set(['q', 'r', 'e', 't', 'w'])
>>> "q" in aset
True
>>> "a" in aset
False
```

## 集合与集合的关系

### A是否等于B

```python
>>> a
set(['q', 'r', 'e', 't', 'w'])
>>> b = set(["a","b","c"])
>>> a == b
False
>>> a != b
True
```

### A是否是B的子集

```python
>>> c = set(["q","r"])
>>> a
set(['q', 'r', 'e', 't', 'w'])
>>> 
>>> c.issubset(a) #判断c是否是a的子集
True
>>> a.issuperset(c) #判断a是否是c的超集
True
```
### AB的并集

```python
>>> a
set(['q', 'r', 'e', 't', 'w'])
>>> b
set(['a', 'c', 'b'])
>>> a | b
set(['a', 'c', 'b', 'e', 'q', 'r', 't', 'w']) #ab的并集
>>> a.union(b)
set(['a', 'c', 'b', 'e', 'q', 'r', 't', 'w']) #ab的并集
```

### AB的交集

```python
>>> a
set(['q', 'r', 'e', 't', 'w'])
>>> b
set(['a', 'q', 'c', 'b'])
>>> a & b   #ab的交集
set(['q'])
>>> a.intersection(b)
set(['q'])
```

### A相对B的差集

A相对B不同的部分元素集合

```python
>>> a
set(['q', 'r', 'e', 't', 'w'])
>>> b
set(['a', 'q', 'c', 'b'])
>>> a - b
set(['r', 'e', 't', 'w'])
>>> a.difference(b)
set(['r', 'e', 't', 'w'])
```

### AB的差集

AB中不同元素的集合

```python
>>> a
set(['q', 'r', 'e', 't', 'w'])
>>> b
set(['a', 'q', 'c', 'b'])
>>> a.symmetric_difference(b)
set(['a', 'c', 'b', 'e', 'r', 't', 'w'])
```

