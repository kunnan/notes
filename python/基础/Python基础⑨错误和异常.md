Python基础之(九)错误和异常
===

# 错误

```python
>>> for i in range(10)
 File "<stdin>", line 1
   for i in range(10)
                    ^
SyntaxError: invalid syntax
```

上面那句话因为缺少冒号`:`，导致解释器无法解释，于是报错。这个报错行为是由Python的语法分析器完成的，并且检测到了错误所在文件和行号（`File "<stdin>", line 1`），还以向上箭头`^`标识错误位置（后面缺少`:`），最后显示错误类型。

另一种常见错误是逻辑错误。逻辑错误可能是由于不完整或者不合法的输入导致，也可能是无法生成、计算等，或者是其它逻辑问题。

当Python检测到一个错误时，解释器就无法继续执行下去，于是抛出提示信息，即为异常。

# 异常

下表中列出常见的异常

|异常 |	描述|
|-----|-----|
|NameError |尝试访问一个没有申明的变量|
|ZeroDivisionError |	除数为0|
|SyntaxError| 	语法错误|
|IndexError| 	索引超出序列范围|
|KeyError| 	请求一个不存在的字典关键字|
|IOError| 	输入输出错误（比如你要读的文件不存在）|
|AttributeError| 	尝试访问未知的对象属性|

## NameError

```python
>>> bar
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
NameError: name 'bar' is not defined
```

Python中变量虽然不需在使用变量之前先声明类型，但也需要对变量进行赋值，然后才能使用。不被赋值的变量，不能再Python中存在，因为变量相当于一个标签，要把它贴到对象上才有意义。

## ZeroDivisionError

```python
>>> 1/0
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
ZeroDivisionError: integer division or modulo by zero
```

## SyntaxError

```python
    >>> for i in range(10)
      File "<stdin>", line 1
        for i in range(10)
                         ^
    SyntaxError: invalid syntax
```

这种错误发生在Python代码编译的时候，当编译到这一句时，解释器不能讲代码转化为Python字节码，就报错。

## IndexError和KeyError

```python
    >>> a = [1,2,3]
    >>> a[4]
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    IndexError: list index out of range
    
    >>> d = {"python":"itdiffer.com"}
    >>> d["java"]
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    KeyError: 'java'
```

## IOError

```python
    >>> f = open("foo")
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    IOError: [Errno 2] No such file or directory: 'foo'
```

## AttributeError

```python
    >>> class A(object): pass        #Python 3: class A: pass
    ... 
    >>> a = A()
    >>> a.foo
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'A' object has no attribute 'foo'
```

# 处理异常

```python
#!/usr/bin/env python
# coding=utf-8

while 1:
   print "this is a division program."
   c = raw_input("input 'c' continue, otherwise logout:")
   if c == 'c':
       a = raw_input("first number:")
       b = raw_input("second number:")
       try:
           print float(a)/float(b)
           print "*************************"
       except ZeroDivisionError:
           print "The second number can't be zero!"
           print "*************************"
   else:
       break
```

## try...except

对于上述程序，只看`try`和`except`部分，如果没有异常发生，`except`子句在`try`语句执行之后被忽略；如果`try`子句中有异常可，该部分的其它语句被忽略，直接跳到`except`部分，执行其后面指定的异常类型及其子句。

`except`后面也可以没有任何异常类型，即无异常参数。如果这样，不论`try`部分发生什么异常，都会执行`except`。

在`except`子句中，可以根据异常或者别的需要，进行更多的操作。比如：

```python
#!/usr/bin/env python
# coding=utf-8

class Calculator(object):
   is_raise = True
   def calc(self, express):
       try:
           return eval(express) #运行表达式
       except ZeroDivisionError:
           if self.is_raise:
               print "zero can not be division."        #Python 3:  "zero can not be division."
           else:
               raise #抛出异常信息
```

## 处理多个异常

```python
Python 2:

    #!/usr/bin/env python
    # coding=utf-8

    while 1:
        print "this is a division program."
        c = raw_input("input 'c' continue, otherwise logout:")
        if c == 'c':
            a = raw_input("first number:")
            b = raw_input("second number:")
            try:
                print float(a)/float(b)
                print "*************************"
            except ZeroDivisionError:
                print "The second number can't be zero!"
                print "*************************"
            except ValueError:
                print "please input number."
                print "************************"
        else:
            break
            
 or
 
except (ZeroDivisionError, ValueError): #括号内也可以包含多个异常
   print "please input rightly."
   print "********************"
```

## 打印异常，但程序不中断

```python
    while 1:
        print "this is a division program."
        c = raw_input("input 'c' continue, otherwise logout:")
        if c == 'c':
            a = raw_input("first number:")
            b = raw_input("second number:")
            try:
                print float(a)/float(b)
                print "*************************"
            except (ZeroDivisionError, ValueError), e: #类似java
                print e
                print "********************"
        else:
            break

Python 3:

    while 1:
        print("this is a division program.")
        c = input("input 'c' continue, otherwise logout:")
        if c == 'c':
            a = input("first number:")
            b = input("second number:")
            try:
                print(float(a)/float(b))
                print("*************************")
            except (ZeroDivisionError, ValueError) as e:
                print(e)
                print("********************")
        else:
            break
```

## else语句

```python
    >>> try:
    ...     print "I am try"        #Python 3: print("I am try")，
    ... except:                
    ...     print "I am except"
    ... else:                     #处理except就不会运行else
    ...     print "I am else"
    ... 
    I am try
    I am else
```
else语句应用，只有输入正确的内容，循环才会终止

```python
    #!/usr/bin/env python
    # coding=utf-8
    while 1:
        try:
            x = raw_input("the first number:")
            y = raw_input("the second number:")

            r = float(x)/float(y)
            print r
        except Exception, e:  #python3为 Exception as e:
            print e
            print "try again."
        else:
            break
```

## finally语句

如果有了`finally`，不管前面执行的是`try`，还是`except`，最终都要执行它。类似java

```python
    >>> x = 10

    >>> try:
    ...     x = 1/0
    ... except Exception, e:        #Python 3:  except Exception as e:
    ...     print e        #Python 3: print(e)
    ... finally:
    ...     print "del x"        #Python 3:  print(e)
    ...     del x
    ... 
    integer division or modulo by zero
    del x
```

## assert

`assert`是一句等价于布尔真的判定，发生异常就意味着表达式为假。当程序运行到某个节点的时候，就断定某个变量的值必然是什么，或者对象必然拥有某个属性等，简单说就是断定什么东西必然是什么，如果不是，就抛出异常。

```python
#!/usr/bin/env python
# coding=utf-8
            
if __name__ == "__main__":
    a = 8
    assert a < 0
    print a
    
Traceback (most recent call last):
  File "/Users/liuguoquan/Documents/workspace/PythonDemo/main.py", line 6, in <module>
    assert a < 0
AssertionError
```

这就是断言assert的引用。什么是使用断言的最佳时机？有文章做了总结：

如果没有特别的目的，断言应该用于如下情况：

- 防御性的编程
- 运行时对程序逻辑的检测
- 合约性检查（比如前置条件，后置条件）
- 程序中的常量
- 检查文档

