Python基础之(七)函数
====

# 函数

## 建立函数

在Python中，规定了一种定义函数的格式，下面的举例就是一个函数，以这个函数为例来说明定义函数的格式和调用函数的方法。

```python
def add_function(a, b): #冒号必须
   c = a + b  #缩进必须
   return c

if __name__ == "__main__":
   result = add_function(2, 3)
   print result               #python3: print(result)
```

定义函数的格式为：

```
def 函数名(参数1，参数2，...，参数n)：

   函数体（语句块）
```

几点说明：

- 函数名的命名规则要符合Python中的命名要求。一般用小写字母和单下划线、数字等组合，有人习惯用aaBb的样式，但我不推荐
- def是定义函数的关键词，这个简写来自英文单词define
- 函数名后面是圆括号，括号里面，可以有参数列表，也可以没有参数
- 千万不要忘记了括号后面的冒号
- 函数体（语句块），相对于def缩进，按照python习惯，缩进四个空格

## 函数命名

Python对命名的一般要求:

- 文件名:全小写,可使用下划线

- 函数名:小写，可以用下划线风格单词以增加可读性。如：myfunction，my_example_function。*注意*：混合大小写仅被允许用于这种风格已经占据优势的时候，以便保持向后兼容。有的人，喜欢用这样的命名风格：myFunction，除了第一个单词首字母外，后面的单词首字母大写。这也是可以的，因为在某些语言中就习惯如此。但我不提倡，这是我非常鲜明的观点。

- 函数的参数：命名方式同变量（本质上就是变量）。如果一个参数名称和Python保留的关键字冲突，通常使用一个后缀下划线会好于使用缩写或奇怪的拼写。

- 变量:变量名全部小写，由下划线连接各个单词。如color = WHITE，this_is_a_variable = 1。

## 调用函数

定义函数

```python
>>> def add(x,y):       #为了能够更明了显示参数赋值特点，重写此函数
...     print "x=",x         #分别打印参数赋值结果
...     print "y=",y
...     return x+y
... 
```

普通调用

```python
>>> add(10, 3)           #x=10,y=3
x= 10
y= 3
13
```

还可以直接把赋值语句写到里面，就明确了参数和对象的关系。当然，这时候顺序就不重要了

```python
>>> add(x=10, y=3)       
x= 10
y= 3
13

>>> add(y=10, x=3)
x= 3
y= 10
13
```

多态调用

```python
>>> def times(x, y=2):       #y的默认值为2
    ...     print "x=",x                 #Python 3: print("x={}".format(x))，以下类似，从略。
    ...     print "y=",y
    ...     return x*y
    ... 
    >>> times(3)                #x=3,y=2
    x= 3
    y= 2
    6

    >>> times(x=3)              #同上
    x= 3
    y= 2
    6
    
>>> times(3, 4)              #x=3,y=4,y的值不再是2
x= 3
y= 4
12

>>> times("qiwsir")         #再次体现了多态特点
x= qiwsir
y= 2
'qiwsirqiwsir'
```

##注意事项

下面的若干条，是常见编写代码的注意事项：

1. 别忘了冒号。一定要记住复合语句首行末尾输入“：”（if,while,for等的第一行）
2. 从第一行开始。要确定顶层（无嵌套）程序代码从第一行开始。
3. 空白行在交互模式提示符下很重要。模块文件中符合语句内的空白行常被忽视。但是，当你在交互模式提示符下输入代码时，空白行则是会结束语句。
4. 缩进要一致。避免在块缩进中混合制表符和空格。
5. 使用简洁的for循环，而不是while or range.相比，for循环更易写，运行起来也更快
6. 要注意赋值语句中的可变对象。
7. 不要期待在原处修改的函数会返回结果,比如list.append()，这在可修改的对象中特别注意
8. 调用函数是，函数名后面一定要跟随着括号，有时候括号里面就是空空的，有时候里面放参数。
9. 不要在导入和重载中使用扩展名或路径。

## 返回值

所谓返回值，就是函数向调用函数的地方返回的数据。

编写一个斐波那契数列函数:

```python
#!/usr/bin/env python
# coding=utf-8

def fibs(n):
   result = [0,1]
   for i in range(n-2):
       result.append(result[-2] + result[-1])
   return result

if __name__ == "__main__":
   lst = fibs(10)
   print lst
```

### 返回多个值元组

```python
    >>> def my_fun():
    ...     return 1, 2, 3
    ... 
    >>> a = my_fun()
    >>> a
    (1, 2, 3)
```

对这个函数，我们还可以用这样的方式来接收函数的返回值。

```python
 >>> x, y, z = my_fun()
    >>> x
    1
    >>> y
    2
    >>> z
    3
```

## 函数文档

```python
#!/usr/bin/env python
# coding=utf-8

def fibs(n):
   """
   This is a Fibonacci sequence. #函数文档
   """
   result = [0,1]
   for i in range(n-2):
       result.append(result[-2] + result[-1])
   return result

if __name__ == "__main__":
   lst = fibs(10)
   print lst
```

```python
    >>> def my_fun():
    ...     """
    ...     This is my function.
    ...     """
    ...     print "I am a craft."
    ... 
    >>> my_fun.__doc__  #调用打印函数文档
    '\n    This is my function.\n    '
```

# 参数收集

函数参数的个数也有不确定的时候，怎么解决这个问题呢？Python用这样的方式解决参数个数的不确定性。

## 元组形式

```python
def func(x, *arg): 
    print x         #Python 3请自动修改为print()的格式，下同，从略。
    result = x
    print arg       #输出通过*arg方式得到的值
    for i in arg:
        result +=i
    return result

print func(1, 2, 3, 4, 5, 6, 7, 8, 9)    #赋给函数的参数个数不仅仅是2个
```
## 字典形式

```python
   >>> def foo(**kargs):
    ...     print kargs        #Python 3:  print(kargs)
    ...
    >>> foo(a=1,b=2,c=3)    #注意观察这次赋值的方式和打印的结果
    {'a': 1, 'c': 3, 'b': 2}
```

## 一种优雅的方式

```python
>>> def add(x, y):
...     return x + y
... 
>>> add(2, 3)
5

>>> bars = (2, 3)
>>> add(*bars)
5
    
>>> bars = (2, 3, 4) #元组中元素的个数，要跟函数所要求的变量个数一致,不然如下报错
>>> add(*bars)
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
TypeError: add() takes exactly 2 arguments (3 given)

```

## 综合

### def foo(p1, p2, p3, ...)

```python
>>> def foo(p1, p2, p3):
...     print "p1==>",p1        #Python 3用户修改为print()格式，下同
...     print "p2==>",p2
...     print "p3==>",p3
... 
>>> foo("python", 1, ["qiwsir","github","io"])   
p1==> python
p2==> 1
p3==> ['qiwsir', 'github', 'io']
```

### def foo(p1=value1, p2=value2, ...)

```python

>>> foo(p3=3, p1=10, p2=222)
    p1==> 10
    p2==> 222
    p3==> 3

>>> def foo(p1, p2=22, p3=33):    #设置了两个参数p2, p3的默认值
...     print "p1==>",p1
...     print "p2==>",p2
...     print "p3==>",p3
... 
>>> foo(11)     #p1=11，其它的参数为默认赋值
p1==> 11
p2==> 22
p3==> 33
>>> foo(11, 222)     #按照顺序，p2=222, p3依旧维持原默认值
p1==> 11
p2==> 222
p3==> 33
>>> foo(11, 222, 333)  #按顺序赋值
p1==> 11
p2==> 222
p3==> 333

>>> foo(11, p2=122)
p1==> 11
p2==> 122
p3==> 33

>>> foo(p2=122)     #p1没有默认值，必须要赋值的，否则报错
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
TypeError: foo() takes at least 1 argument (1 given)
```
### def foo(*args)

这种方式适合于不确定参数个数的时候，在参数args前面加一个`*`

```python
>>> def foo(*args): 
...     print args
... 
>>> foo("qiwsir.github.io")
('qiwsir.github.io',)
>>> foo("qiwsir.github.io","python")
('qiwsir.github.io', 'python')
```


### def foo(**args)

这种方式跟上面的区别在于，必须接收类似`arg=val`形式的。

```python
>>> def foo(**args):    
...     print args
... 

>>> foo(1,2,3)   
Traceback (most recent call last):
 File "<stdin>", line 1, in <module>
TypeError: foo() takes exactly 0 arguments (3 given)

>>> foo(a=1,b=2,c=3)
{'a': 1, 'c': 3, 'b': 2}
```

# 特殊函数

## lambda

```python
# !/usr/bin/env python
#coding=utf-8

def add(x,y = 3):
    return x + y

ret = add(5)
print ret

lam = lambda x : x + 3
ret = lam(5)
print ret

lam = lambda x,y : x + y
ret = lam(5,5)
print ret

8
8
10
```

lambda函数的使用方法：

-	lambda后面直接跟变量；
-  变量后面是冒号；
-  冒号后面是表达式，表达式计算结果就是本函数的返回值；

>lambda函数不能包含太多的命令，包含的表达式不能超过一个，不要试图向lambda函数中塞入太多的东西，如果需要更复杂的东西，应该定义一个普通的函数。

## map

```python
# !/usr/bin/env python
#coding=utf-8

def add(x,y = 3):
    return x + y

numbers = range(9)
print numbers

ret = map(add, numbers) #只引用函数名即可
print ret

ret = map(lambda x : x + 4, numbers) #
print ret

ret = [x + 4 for x in numbers] #列表解析的方式实现
print ret
```

map()是Python的一个内置函数，它的基本样式是：
`map(fun,seq)`

func是一个函数，seq是一个序列对象。在执行的时候，序列对象中的每个对象，按照从左到右的顺序依次被取出来，塞入到func函数里面，并将func的返回值依次存到一个列表中。

## reduce

reduce()是横着逐个元素进行运算

```python
# !/usr/bin/env python
#coding=utf-8

def add(x,y): #连续相加
    return x + y

def mul(x,y): #连续相乘
    return x * y

numbers = range(9)
print numbers

ret = reduce(add, numbers) 
print ret

ret = reduce(mul, numbers) 
print ret

[0, 1, 2, 3, 4, 5, 6, 7, 8]
36
0
```

## filter

```python
# !/usr/bin/env python
#coding=utf-8

numbers = range(-5,5)
print numbers

ret = filter(lambda x : x > 0, numbers) #过滤掉x < 0的数
print ret

ret = [x for x in numbers if x > 0]
print ret

ret = filter(lambda c : c != 'i', "liuguoquan") #过滤掉字符i
print ret
```

# 练习

## 求解一元二次方程

```python
# !/usr/bin/env python
#coding=utf-8

"""
 求解一元二次方程
 
"""

from __future__ import division
import math

def quadratic_equation(a,b,c):
    delta = b * b - 4 * a * c
    if delta < 0:
        return False
    elif delta == 0:
        return -(b / (2 * a))
    else:
        sqrt_delat = math.sqrt(delta)
        x1 = (-b + sqrt_delat) / (2 * a)
        x2 = (-b - sqrt_delat) / (2 * a)
        return x1,x2

if __name__ == "__main__":
    print "a quadratic equation: x^2 + 2x + 1 = 0"
    coefficients = (1,2,1)
    roots = quadratic_equation(*coefficients)
    if roots:
        print "the result is: ",roots
    else:
        print "this equation has no solution"
        
a quadratic equation: x^2 + 2x + 1 = 0
the result is:  -1.0
```

## 统计考试成绩

```python
# !/usr/bin/env python
#coding=utf-8

"""
 统计考试成绩
 
"""

from __future__ import division
import math

def average_score(scores):
    """
        统计平均分
    """
    score_values = scores.values()
    sum_scores = sum(score_values)
    average = sum_scores / len(score_values)
    return average

def sorted_score(scores):
    """
        对成绩从高到低排序呢
    """
    score_list = [(scores[k],k) for k in scores]  #将键-值互换位置 score_list是列表,里面的元素是一个元组
    sort_lst = sorted(score_list,reverse = True)
    return [(i[1],i[0]) for i in sort_lst] #将键-值互换位置

def max_score(scores):
    """
        成绩最高的姓名和分数
    """
    lst = sorted_score(scores)
    max_score = lst[0][1]
    return [(i[0],i[1]) for i in lst if i[1] == max_score]

def min_scroe(scores):
    """
        成绩最低的姓名和分数
    """
    lst = sorted_score(scores)
    min_score = lst[len(lst) - 1][1]
    return [(i[0],i[1]) for i in lst if i[1] == min_score]

if __name__ == "__main__":
    scores = {"google":98,"facebook":99,"baidu":52,"alibab":80,"yahoo":49,"android":76,"apple":99,"amazon":99}
    
    ret = average_score(scores) #平均分
    print "average is: ",ret

    ret = sorted_score(scores) #成绩表
    print "list of scores is: ",ret
    
    ret = max_score(scores) #学霸们
    print "学霸是: ",ret
    
    ret = min_scroe(scores) #学渣
    print "学渣是: ",ret

average is:  81.5
list of scores is:  [('facebook', 99), ('apple', 99), ('amazon', 99), ('google', 98), ('alibab', 80), ('android', 76), ('baidu', 52), ('yahoo', 49)]
学霸是:  [('facebook', 99), ('apple', 99), ('amazon', 99)]
学渣是:  [('yahoo', 49)]
```

## 找质数

质数又称素数，指在大于1的自然数中，除了1和此整数自身外，无法被其他自然整数整除的数（也可定义为只有1和本身两个因数的数）

```python
# !/usr/bin/env python
#coding=utf-8

"""
寻找质数
"""
import math

def is_prime(n):
    """
    判断一个数是否是质数
    """
    if n <=1:
        return False
    for i in range(2,int(math.sqrt(n) + 1)):
        if n % i == 0:
            return False
        return True

if __name__ == "__main__":
    
    primes = [i for i in range(2,100) if is_prime(i)]
    print primes

[5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31, 33, 35, 37, 39, 41, 43, 45, 47, 49, 51, 53, 55, 57, 59, 61, 63, 65, 67, 69, 71, 73, 75, 77, 79, 81, 83, 85, 87, 89, 91, 93, 95, 97, 99]     
```
## 编写函数的注意事项

-	尽量不要使用全局变量
- 	如果参数是可变数据类型，则在函数内不要修改它
-  每个函数的功能和目的要单一，不要一个函数试图做很多事情
-  函数的代码行数尽量少
-  函数的独立性越强越好，不要跟其他的外部东西产生关联


