Python基础之(十一)数据存储
===

# pickle

pickle是标准库中的一个模块，在Python 2中还有一个cpickle，两者的区别就是后者更快。所以，下面操作中，不管是用`import pickle`，还是用`import cpickle as pickle`，在功能上都是一样的。

而在Python 3中，你只需要`import pickle`即可，因为它已经在Python 3中具备了Python 2中的cpickle同样的性能。

`pickle.dump(obj,file[,protocol])`

- obj：序列化对象，在上面的例子中是一个列表，它是基本类型，也可以序列化自己定义的对象。
- file：要写入的文件。可以更广泛地可以理解为为拥有`write()`方法的对象，并且能接受字符串为为参数，所以，它还可以是一个`StringIO`对象，或者其它自定义满足条件的对象。
- protocol：可选项。默认为False（或者说0），是以ASCII格式保存对象；如果设置为1或者True，则以压缩的二进制格式保存对象。

## 序列化对象 

```python
    >>> import pickle
    >>> d = {}
    >>> integers = range(9999)
    >>> d["i"] = integers        #下面将这个字典类型的对象存入文件
    
    >>> f = open("22902.dat", "wb")
    >>> pickle.dump(d, f)           #文件中以ascii格式保存数据
    >>> f.close()

    >>> f = open("22903.dat", "wb")
    >>> pickle.dump(d, f, True)     #文件中以二进制格式保存数据,文件较小，推荐方式
    >>> f.close()

    >>> import os
    >>> s1 = os.stat("22902.dat").st_size    #得到两个文件的大小
    >>> s2 = os.stat("22903.dat").st_size
    
    >>> print "%d, %d, %.2f%%" % (s1, s2, (s2+0.0)/s1*100)    #Python 3: print("{0:d}, {1:d}, {2:.2f}".format (s1, s2, (s2+0.0)/s1*100))
    68903, 29774, 43.21%
```

## 反序列化对象

```python
将数据保存入文件，还有另外一个目标，就是要读出来，也称之为反序列化。

    >>> integers = pickle.load(open("22901.dat", "rb"))
    >>> print integers    #Python 3: print(integers)
    [1, 2, 3, 4, 5]

再看看以二进制存入的那个文件：

    >>> f = open("22903.dat", "rb")
    >>> d = pickle.load(f)
    >>> print d
    {'i': [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, ....   #省略后面的数字}
    >>> f.close()
```

## 读取自定义对象

```python
    >>> import cPickle as pickle        #这是Python 2的引入方式，如果是Python 3，直接使用import pickle
    >>> import StringIO                 #标准库中的一个模块，跟file功能类似，只不过是在内存中操作“文件”
    
    >>> class Book(object):             #自定义一种类型
    ...     def __init__(self,name):
    ...         self.name = name
    ...     def my_book(self):
    ...         print "my book is: ", self.name        #Python 3: print("my book is: ", self.name)
    ... 
    
# 存数据

>>> file = StringIO.StringIO()
>>> pickle.dump(pybook, file, 1)
    
# 取数据

>>> file.seek(0)       #找到对应类型  
>>> pybook2 = pickle.load(file)
>>> pybook2.my_book()
my book is:  <from beginner to master>
>>> file.close()
```

# shelve

由于数据的复杂性，`pickle`只能完成一部分工作，在另外更复杂的情况下，它就稍显麻烦了。于是，又有了`shelve`。

```python

# 写操作

>>> import shelve
>>> s = shelve.open("22901.db")
>>> s["name"] = "www.itdiffer.com"
>>> s["lang"] = "python"
>>> s["pages"] = 1000
>>> s["contents"] = {"first":"base knowledge","second":"day day up"}
>>> s.close()
  
# 读操作
    
>>> s = shelve.open("22901.db")
>>> name = s["name"]
>>> print name        #Python 3: print(name)
www.itdiffer.com
>>> contents = s["contents"]
>>> print contents        #Python 3: print(contents)
{'second': 'day day up', 'first': 'base knowledge'}

# for循环读

>>> for k in s:
...     print k, s[k]
... 
contents {'second': 'day day up', 'first': 'base knowledge'}
lang python
pages 1000
name www.itdiffer.com
```

所建立的对象被变量`s`所引用，就如同字典一样，可称之为类字典对象。所以，可以如同操作字典那样来操作它。

但是，要小心坑：

```python
    >>> f = shelve.open("22901.db")
    >>> f["author"]
    ['qiwsir']
    >>> f["author"].append("Hetz")    #试图增加一个
    >>> f["author"]                   #坑就在这里
    ['qiwsir']
    >>> f.close()

当试图修改一个已有键的值时没有报错，但是并没有修改成功。要填平这个坑，需要这样做：
    
    >>> f = shelve.open("22901.db", writeback=True)    #多一个参数True
    >>> f["author"].append("Hetz")
    >>> f["author"]                   #没有坑了
    ['qiwsir', 'Hetz']
    >>> f.close()

还用`for`循环一下：

    >>> f = shelve.open("22901.db")
    >>> for k,v in f.items():
    ...     print k,": ",v        #Python 3: print(k,": ",v)
    ... 
    contents :  {'second': 'day day up', 'first': 'base knowledge'}
    lang :  python
    pages :  1000
    author :  ['qiwsir', 'Hetz']
    name :  www.itdiffer.com
```


