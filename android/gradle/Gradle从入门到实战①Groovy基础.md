**Gradle从入门到实战①Groovy基础**

[TOC]

[原文摘自Gradle从入门到实战 - Groovy基础](https://mp.weixin.qq.com/s?__biz=MzIwMTAzMTMxMg==&mid=2649492338&idx=1&sn=49cb619fb057720db505b7c3b8f894e8&chksm=8eec808db99b099b6b0bc5e983fc10df48a085a78ca935593737ec9d76b373188e20cf1042d9&scene=0&key=46241b832ae08f161e07f45ac94703881edae3083d433193a3d89056423ae63ce7b358170efb8ccab3c6947d314c362d6c00f74017a5c55c6aaabe7d34388854a1c488b2ad9912169a1119b0465778d6&ascene=0&uin=MjU2OTM5MzI4MA%3D%3D&devicetype=iMac+MacBookPro11%2C4+OSX+OSX+10.12.6+build(16G29)&version=12020810&nettype=WIFI&fontScale=100&pass_ticket=L9cUO41Tj2B0EVKt8%2BEKYyzy9xZJH%2FQgcZApOiz%2FXoSMFV1n8BLRJOudwXwZ%2BiCZ)

本篇文章讲解Groovy基础。为什么是Groovy基础呢，因为玩转Gradle并不需要学习Groovy的全部细节。Groovy是一门jvm语言，功能比较强大，细节也很多，全部学习的话比较耗时，对我们来说收益较小。

# 为什么是Gradle？

Gradle是目前Android主流的构建工具，不管你是通过命令行还是通过AndroidStudio来build，最终都是通过Gradle来实现的。所以学习Gradle非常重要。

目前国内对Android领域的探索已经越来越深，不少技术领域如插件化、热修复、构建系统等都对Gradle有迫切的需求，不懂Gradle将无法完成上述事情。所以Gradle必须要学习。

# 如何学习Gradle？

Gradle不单单是一个配置脚本，它的背后是几门语言，如果硬让我说，我认为是三门语言。

* Groovy Language
* Gradle DSL
* Android DSL

DSL的全称是Domain Specific Language，即领域特定语言，或者直接翻译成“特定领域的语言”，算了，再直接点，其实就是这个语言不通用，只能用于特定的某个领域，俗称“小语言”。因此DSL也是语言。

在你不懂这三门语言的情况下，你很难达到精通Gradle的程度。这个时候从网上搜索，或者自己记忆的一些配置，其实对你来说是很大的负担。但是把它们当做语言来学习，则不需要记忆这些配置，因为语言都是有文档的，我们只需要学语法然后查文档即可，没错，这就是学习方法，这就是正道。

下面来开始学习Groovy的基本语法。

# Groovy和Java的关系

Groovy是一门jvm语言，它最终是要编译成class文件然后在jvm上执行，所以Java语言的特性Groovy都支持，我们完全可以混写Java和Groovy。

既然如此，那Groovy的优势是什么呢？简单来说，Groovy提供了更加灵活简单的语法，大量的语法糖以及闭包特性可以让你用更少的代码来实现和Java同样的功能。比如解析xml文件，Groovy就非常方便，只需要几行代码就能搞定，而如果用Java则需要几十行代码。

# Groovy的变量和方法声明

在Groovy中，通过 def 关键字来声明变量和方法，比如：

```Java
def a = 1;
def b = "hello world";
def int c = 1;

def hello() {
    println ("hello world");
    return 1;
}
```
在Groovy中，很多东西都是可以省略的，比如

* 语句后面的分号是可以省略的
* 变量的类型和方法的返回值也是可以省略的
* 方法调用时，括号也是可以省略的
* 甚至语句中的return都是可以省略的

所以上面的代码也可以写成如下形式：

```Java
def a = 1
def b = "hello world"
def int c = 1

def hello() {
    println "hello world" // 方法调用省略括号
    1;                    // 方法返回值省略return
}
def hello(String msg) {
    println (msg)
}

// 方法省略参数类型
int hello(msg) {
    println (msg)
    return 1
}

// 方法省略参数类型
int hello(msg) {
    println msg
    return 1 // 这个return不能省略
    println "done"
}
```
**总结**

* 在Groovy中，类型是弱化的，所有的类型都可以动态推断，但是Groovy仍然是强类型的语言，类型不匹配仍然会报错；
* 在Groovy中很多东西都可以省略，所以寻找一种自己喜欢的写法；
* Groovy中的注释和Java中相同。

# Groovy的数据类型

在Groovy中，数据类型有：

* Java中的基本数据类型
* Java中的对象
* Closure（闭包）
* 加强的List、Map等集合类型
* 加强的File、Stream等IO类型

>类型可以显示声明，也可以用 def 来声明，用 def 声明的类型Groovy将会进行类型推断。

基本数据类型和对象这里不再多说，和Java中的一致，只不过在Gradle中，对象默认的修饰符为public。下面主要说下String、闭包、集合和IO等。

## String

String的特色在于字符串的拼接，比如

```Java
def a = 1
def b = "hello"
def c = "a=${a}, b=${b}"
println c

outputs:
a=1, b=hello
```

## 闭包

Groovy中有一种特殊的类型，叫做Closure，翻译过来就是闭包，这是一种类似于C语言中函数指针的东西。闭包用起来非常方便，在Groovy中，闭包作为一种特殊的数据类型而存在，闭包可以作为方法的参数和返回值，也可以作为一个变量而存在。

如何声明闭包？

```Java
{ parameters ->
    code
}
```

闭包可以有返回值和参数，当然也可以没有。下面是几个具体的例子：

```Java
def closure = { int a, String b ->
    println "a=${a}, b=${b}, I am a closure!"
}

// 这里省略了闭包的参数类型
def test = { a, b ->
    println "a=${a}, b=${b}, I am a closure!"
}

def ryg = { a, b ->
    a + b
}

closure(100, "renyugang")
test.call(100, 200)
def c = ryg(100,200)
println c
```
闭包可以当做函数一样使用，在上面的例子中，将会得到如下输出：

```
a=100, b=renyugang, I am a closure!
a=100, b=200, I am a closure!
300
```
另外，如果闭包不指定参数，那么它会有一个隐含的参数 it

```Java
// 这里省略了闭包的参数类型
def test = {
    println "find ${it}, I am a closure!"
}

test(100)

outputs:
find 100, I am a closure! 
```
闭包的一个难题是如何确定闭包的参数，尤其当我们调用Groovy的API时，这个时候没有其他办法，只有查询Groovy的文档：

[http://www.groovy-lang.org/api.html](http://www.groovy-lang.org/api.html)
[http://docs.groovy-lang.org/latest/html/groovy-jdk/index-all.html](http://docs.groovy-lang.org/latest/html/groovy-jdk/index-all.html)

## List和Map

Groovy加强了Java中的集合类，比如List、Map、Set等。

List的使用如下：

```Java
def emptyList = []
def test = [100, "hello", true]
test[1] = "world"
println test[0]
println test[1]
test << 200 //添加新元素
println test.size

outputs:
100
world
4
```
List还有一种看起来很奇怪的操作符<<，其实这并没有什么大不了，左移位表示向List中添加新元素的意思，这一点从文档当也能查到。

![list_shift](http://7xs7a3.com1.z0.glb.clouddn.com/list_shift.png)

Map的使用如下：

```Java
def emptyMap = [:]
def test = ["id":1, "name":"renyugang", "isMale":true]
test["id"] = 2
test.id = 900
println test.id
println test.isMale

outputs:
900
true
```

这里借助Map再讲述下如何确定闭包的参数。比如我们想遍历一个Map，我们想采用Groovy的方式，通过查看文档，发现它有如下两个方法，看起来和遍历有关：

![map_closure](http://7xs7a3.com1.z0.glb.clouddn.com/map_closure.png)

可以发现，这两个each方法的参数都是一个闭包，那么我们如何知道闭包的参数呢？当然不能靠猜，还是要查文档。

![map_each](http://7xs7a3.com1.z0.glb.clouddn.com/map_each.png)

通过文档可以发现，这个闭包的参数还是不确定的，如果我们传递的闭包是一个参数，那么它就把entry作为参数；如果我们传递的闭包是2个参数，那么它就把key和value作为参数。

按照这种提示，我们来尝试遍历下：

```Java
def emptyMap = [:]
def test = ["id":1, "name":"renyugang", "isMale":true]

test.each { key, value ->
    println "two parameters, find [${key} : ${value}]"
}

test.each {
    println "one parameters, find [${it.key} : ${it.value}]"
}

outputs:
two parameters, find [id : 1]
two parameters, find [name : renyugang]
two parameters, find [isMale : true]

one parameters, find [id : 1]
one parameters, find [name : renyugang]
one parameters, find [isMale : true]
```

## 加强的IO

在Groovy中，文件访问要比Java简单的多，不管是普通文件还是xml文件。怎么使用呢？还是来查文档。

![io_eachline](http://7xs7a3.com1.z0.glb.clouddn.com/io_eachline.png)

根据File的eachLine方法，我们可以写出如下遍历代码，可以看到，eachLine方法也是支持1个或2个参数的，这两个参数分别是什么意思，就需要我们学会读文档了，一味地从网上搜例子，多累啊，而且很难彻底掌握：

```Java
def file = new File("a.txt")

println "read file using two parameters"
file.eachLine { line, lineNo ->
    println "${lineNo} ${line}"
}

println "read file using one parameters"
file.eachLine { line ->
    println "${line}"
}

outputs:
read file using two parameters
1 欢迎
2 关注
3 玉刚说

read file using one parameters
欢迎
关注
玉刚说
```

除了eachLine，File还提供了很多Java所没有的方法，大家需要浏览下大概有哪些方法，然后需要用的时候再去查就行了，这就是学习Groovy的正道。

下面我们再来看看访问xml文件，也是比Java中简单多了。
Groovy访问xml有两个类：XmlParser和XmlSlurper，二者几乎一样，在性能上有细微的差别，如果大家感兴趣可以从文档上去了解细节，不过这对于本文不重要。

在下面的链接中找到XmlParser的API文档，参照例子即可编程，
[http://docs.groovy-lang.org/docs/latest/html/api/](http://docs.groovy-lang.org/docs/latest/html/api/)。

假设我们有一个xml，attrs.xml，如下所示：

```xml
<resources>

<declare-styleable name="CircleView">
    <attr name="circle_color" format="color">#98ff02</attr>
    <attr name="circle_size" format="integer">100</attr>
    <attr name="circle_title" format="string">renyugang</attr>
</declare-styleable>

</resources>
```

```Java
def xml = new XmlParser().parse(new File("attrs.xml"))
// 访问declare-styleable节点的name属性
println xml['declare-styleable'].@name[0]

// 访问declare-styleable的第三个子节点的内容
println xml['declare-styleable'].attr[2].text()

outputs：
CircleView
renyugang
```

# Groovy的其他特性

除了本文中已经分析的特性外，Groovy还有其他特性。

## Class是一等公民

在Groovy中，所有的Class类型，都可以省略.class，比如：

```Java
func(File.class)
func(File)

def func(Class clazz) {
}
```

## Getter和Setter

在Groovy中，Getter/Setter和属性是默认关联的，比如：

```Java
class Book {
    private String name
    String getName() { return name }
    void setName(String name) { this.name = name }
}

class Book {
    String name
}
```
上述两个类完全一致，只有有属性就有Getter/Setter；同理，只要有Getter/Setter，那么它就有隐含属性。

## with操作符

在Groovy中，当对同一个对象进行操作时，可以使用with，比如：

```Java
Book bk = new Book()
bk.id = 1
bk.name = "android art"
bk.press = "china press"

可以简写为：
Book bk = new Book() 
bk.with {
    id = 1
    name = "android art"
    press = "china press"
}
```

## 判断是否为真

```Java
if (name != null && name.length > 0) {}

可以替换为：
if (name) {}
```

## 简洁的三元表达式

```Java
def result = name != null ? name : "Unknown"

// 省略了name
def result = name ?: "Unknown"
```

## 简洁的非空判断

在Groovy中，非空判断可以用?表达式，比如

```Java
if (order != null) {
    if (order.getCustomer() != null) {
        if (order.getCustomer().getAddress() != null) {
        System.out.println(order.getCustomer().getAddress());
        }
    }
}

可以简写为：
println order?.customer?.address
```

## 使用断言

在Groovy中，可以使用assert来设置断言，当断言的条件为false时，程序将会抛出异常：

```Java
def check(String name) {
    // name non-null and non-empty according to Gro    ovy Truth
    assert name
    // safe navigation + Groovy Truth to check
    assert name?.size() > 3
}
```
## switch方法

在Groovy中，switch方法变得更加灵活，可以同时支持更多的参数类型：

```Java
def x = 1.23
def result = ""
switch (x) {
    case "foo": result = "found foo"
    // lets fall through
    case "bar": result += "bar"
    case [4, 5, 6, 'inList']: result = "list"
    break
    case 12..30: result = "range"
    break
    case Integer: result = "integer"
    break
    case Number: result = "number"
    break
    case { it > 3 }: result = "number > 3"
    break
    default: result = "default"
}
assert result == "number"
```

## ==和equals

在Groovy中，==相当于Java的equals，，如果需要比较对个对象是否是同一个，需要使用.is()。

```Java
Object a = new Object()
Object b = a.clone()

assert a == b
assert !a.is(b)
```

# 编译、运行Groovy

* 下载[gralde](http://services.gradle.org/distributions/)，并将gradle命令加入到环境变量
* 在当面目录下创建build.gradle文件，在里面创建一个task，然后在task中编写Groovy代码即可
* gradle 任务名 //编译运行

```Java
task(yugangshuo).doLast {
    println "start execute yuangshuo"
    haveFun()
}

def haveFun() {
    println "have fun!"
    System.out.println("have fun!");
    1
    def file1 = new File("a.txt")
    def file2 = new File("a.txt")
    assert file1 == file2
    assert !file1.is(file2)
}

class Book {
    private String name
    String getName() { return name }
    void setName(String name) { this.name = name }
}

```
只需要在haveFun方法中编写Groovy代码即可，如下命令即可运行：

```Java
gradle yugangshuo
```
打印：

```
:yugangshuo
start execute yuangshuo
have fun!
have fun!
```


