Python基础之(八)类
===

# 类

## 创建类

-	第一形式

```python
# !/usr/bin/env python
# coding=utf-8

class Person(object): #object表示继承自object类，Python3中可省略次内容
    """
    This is a sample of Class
    """
    breast = 90  #类的属性 是静态变量
    
    def __init__(self, name): #初始化方法  self为对象实例本身
        self.name = name
        
    def get_name(self):  #类的方法
        return self.name
    
    def color(self,color):
        d = {}
        d[self.name] = color;
        return d
    
if __name__ == "__main__":
    girl = Person("songjia")
    print girl.name
    girl.name ="liu"
    print girl.get_name()
    print girl.color("white")
    girl.breast = 80 #修改实例的属性
    print girl.breast
    print Person.breast #类属性不会随实例属性的改变而改变
    Person.breast = 100
    print Person.breast
    print girl.breast
```

-	第二种形式

```python
>>> __metaclass__ = type
>>> class CC:
...     pass
... 
>>> cc = CC()
>>> cc.__class__
<class '__main__.CC'>
>>> type(cc)
<class '__main__.CC'>
```

## 实例

```python
girl = Person("songjia")
```

## 类属性

```python
>>> class A(object):    #Python 3: class A:
...         x = 7  #类的属性
... 
```

下面列出类的几种特殊属性的含义:

- `C.__name__`：以字符串的形式，返回类的名字，注意这时候得到的仅仅是一个字符串，它不是一个类对象
- `C.__doc__`：显示类的文档
- `C.__base__`：类C的所有父类。如果是按照上面方式定义的类，应该显示`object`，因为以上所有类都继承了它。等到学习了“继承”，再来看这个属性，内容就丰富了
- `C.__dict__`：以字典形式显示类的所有属性
- `C.__module__`：类所在的模块

## 实例属性

```python
>>> class A(object):    #Python 3: class A:
...         x = 7  #类的属性
... 
>>> foo = A()
>>> foo.x #实例属性
```
### 类中变量引用可变数据

```python
>>> class B(object):
...     y = [1, 2, 3]
    
   >>> B.y         #类属性
[1, 2, 3]
>>> bar = B()
>>> bar.y       #实例属性
[1, 2, 3]

>>> bar.y.append(4)
>>> bar.y
[1, 2, 3, 4]
>>> B.y
[1, 2, 3, 4]

>>> B.y.append("aa")
>>> B.y
[1, 2, 3, 4, 'aa']
>>> bar.y
[1, 2, 3, 4, 'aa']
```
当类中变量引用的是可变对象是，类属性和实例属性都能直接修改这个对象，从而影响另一方的值。

## 访问限制

```python
# !/usr/bin/env python
# coding=utf-8

class Person(object):
    """
    This is a sample of Class
    """
    
    def __init__(self, name): #初始化方法  self为对象实例本身
        self.__name = name #__xx双下划线表示类的私有变量
        
    def get_name(self):  #类的方法
        return self.__name #类的内部可以访问
    
if __name__ == "__main__":
    girl = Person("songjia")
    print girl.get_name()
    print girl._name #无法访问类的私有变量
```

## 文档字符串

在函数、类或者文件开头的部分写文档字符串说明，一般采用三重引号。这样写的最大好处是能够用help()函数看。

```python
"""This is python lesson"""

def start_func(arg):
   """This is a function."""
   pass

class MyClass:
   """This is my class."""
   def my_method(self,arg):
       """This is my method."""
       pass
```

# 继承

## 单继承

```python
#!/usr/bin/env python
# coding=utf-8

class Person(object):        #Python 3: class Person:
    def __init__(self, name):
        self.name = name
    
    def height(self, m):
        h = dict((["height", m],))
        return h

    def breast(self, n):
        b = dict((["breast", n],))
        return b

class Girl(Person):  #继承
    def get_name(self):
        return self.name

if __name__ == "__main__":
    cang = Girl("liuguoquan")
    print cang.get_name()        #Python 3: print(cang.get_name())，下同，从略
    print cang.height(160)
    print cang.breast(90)
```

## 调用覆盖的方法

```python
class Girl(Person):
   def __init__(self, name):
       #Person.__init__(self, name) #调用父类的方法
       super(Girl, self).__init__(name) #调用父类的方法常用写法
       self.real_name = "Aoi sola"

   def get_name(self):
       return self.name
       
if __name__ == "__main__":
   cang = Girl("canglaoshi")
   print cang.real_name
   print cang.get_name()
   print cang.height(160)
   print cang.breast(90)

执行结果为：

    Aoi sola
    canglaoshi
    {'height': 160}
    {'breast': 90}
```

## 多继承

```python
#!/usr/bin/env python
# coding=utf-8

class Person(object):        #Python 3: class Person:
    def eye(self):
        print "two eyes"

    def breast(self, n):
        print "The breast is: ",n

class Girl(object):        #Python 3: class Gril:
    age = 28
    def color(self):
        print "The girl is white"

class HotGirl(Person, Girl): #多重继承
    pass

if __name__ == "__main__":
    kong = HotGirl()
    kong.eye()
    kong.breast(90)
    kong.color()
    print kong.age
    
two eyes
The breast is:  90
The girl is white
28
```

## 多重继承的顺序-广度优先

```python
class K1(object):        #Python 3: class K1:
   def foo(self):
       print "K1-foo"    #Python 3: print("K1-foo")，下同，从略

class K2(object):        #Python 3: class K2:
   def foo(self):
       print "K2-foo"
   def bar(self):
       print "K2-bar"

class J1(K1, K2):
   pass

class J2(K1, K2):
   def bar(self):
       print "J2-bar"

class C(J1, J2):
pass
   
if __name__ == "__main__":
print C.__mro__
m = C()
m.foo()
m.bar()
   
K1-foo
J2-bar
```
代码中的`print C.__mro__`是要打印出类的继承顺序。从上面清晰看出来了。如果要执行`foo()`方法，首先看`J1`，没有，看`J2`，还没有，看`J1`里面的`K1`，有了，即C==>J1==>J2==>K1；`bar()`也是按照这个顺序，在`J2`中就找到了一个。

这种对继承属性和方法搜索的顺序称之为“广度优先”。
Python 2的新式类，以及Python 3中都是按照此顺序原则搜寻属性和方法的。

# 方法

## 绑定方法

```python
#!/usr/bin/env python
# coding=utf-8

class Person(object):        #Python 3: class Person:
    def eye(self):
        print "two eyes"

    def breast(self, n):
        print "The breast is: ",n

class Girl(object):        #Python 3: class Gril:
    age = 28
    def color(self):
        print "The girl is white"

class HotGirl(Person, Girl): #多重继承
    pass

if __name__ == "__main__":
    kong = HotGirl() #实例化实现了方法和实例的绑
    kong.eye() #调用绑定方法
```

## 非绑定方法

在子类中，父类的方法就是**非绑定方法**，因为在子类中，没有建立父类的实例，却要是用父类的方法。

## 静态方法和类方法

```python
#!/usr/bin/env python
# coding=utf-8

__metaclass__ = type

class StaticMethod: #静态方法
    @staticmethod
    def foo():
        print "This is static method foo()."

class ClassMethod: #类方法
    @classmethod
    def bar(cls): #类方法必须有cls参数
        print "This is class method bar()."
        print "bar() is part of class:", cls.__name__

if __name__ == "__main__":
    static_foo = StaticMethod()    #实例化
    static_foo.foo()               #实例调用静态方法
    StaticMethod.foo()             #通过类来调用静态方法
    print "********"
    class_bar = ClassMethod()
    class_bar.bar()
    ClassMethod.bar()
    
This is static method foo().
This is static method foo().
********
This is class method bar().
bar() is part of class: ClassMethod
This is class method bar().
bar() is part of class: ClassMethod
```

在python中：

- `@staticmethod`表示下面的方法是静态方法
- `@classmethod`表示下面的方法是类方法

# 多态和封装

## 多态

```python
class Cat:
   def speak(self):
       print "meow!"

class Dog:
   def speak(self):
       print "woof!"

class Bob:
   def bow(self):
       print "thank you, thank you!"
   def speak(self):
       print "hello, welcome to the neighborhood!"
   def drive(self):
       print "beep, beep!"

def command(pet):
   pet.speak()

pets = [ Cat(), Dog(), Bob() ]

for pet in pets:
   command(pet)
```

Python中的多态特点,Python不检查传入对象的类型，这种方式被称之为“隐式类型”（laten typing）或者“结构式类型”（structural typing），也被通俗的称为“鸭子类型”(duck typeing)，Python是弱类型语言。

Java会检查传入对象的类型，所以是强类型语言。

## 封装和私有化

要了解封装，离不开“私有化”，就是将类或者函数中的某些属性限制在某个区域之内，外部无法调用。

Python中私有化的方法也比较简单，就是在准备私有化的属性（包括方法、数据）名字前面加双下划线。例如：

```python
#!/usr/bin/env python
# coding=utf-8

class ProtectMe(object):        #Python 3: class ProtectMe:
    def __init__(self):
        self.me = "qiwsir"
        self.__name = "kivi" #私有变量

    def __python(self): #私有方法
        print "I love Python."        

    def code(self):
        print "Which language do you like?"
        self.__python()

if __name__ == "__main__":
    p = ProtectMe()
    print p.me
    print p.code()
```

### 如何将一个方法变成属性调用？

可以使用`property`函数。

```python
#!/usr/bin/env python
# coding=utf-8

class ProtectMe(object):        #Python 3: class ProtectMe:
    def __init__(self):
        self.me = "qiwsir"
        self.__name = "kivi" #私有变量

    def __python(self): #私有方法
        print "I love Python."        

    @property
    def code(self):
        print "Which language do you like?"
        self.__python

if __name__ == "__main__":
    p = ProtectMe()
    print p.me
    print p.code #调用方法名即可
```

