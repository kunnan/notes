**Java8特性① Lambda 表达式**

[TOC]

# 简介

## 概念

**Lambda 表达式**可以理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。

* **匿名**：它不像普通方法那样有一个明确的名称；
* **函数**：Lambda 表达式是函数是因为它不像方法那样属于某个特定的类，但和方法一样，Lambda 有参数列表、函数主体、返回类型，还可能有可以抛出的异常列表；
* **传递**：Lambda 表达式可以作为参数传递给方法或存储在变量中；
* **简洁**：无需像匿名类那样写很多模板代码；

## 组成

Lambda 表达式由参数列表、箭头和 Lambda 主体组成。

```java
(Apple o1, Apple o2) -> Integer.valueOf(o1.getWeight()).compareTo(Integer.valueOf(o2.getWeight()))
```
* **参数列表：**这里采用了 Comparator 中 compareTo 方法的参数；
* **箭头：**箭头把参数列表和 Lambda 主体分开；
* **Lambda 主体：**表达式就是 Lambda 的返回值；

## 表达式

Java8中有效的 Lambda 表达式如下：

| Lambda 表达式 | 含义 
| --- | --- 
| `(String s) -> s.length()` |  表达式具有一个 String 类型的参数并返回一个 int。 Lambda 没有 return 语句，因为已经隐含的 return，可以显示调用 return。
| `(Apple a) -> a.getWeight() > 150` |  表达式有一个 Apple 类型的参数并返回一个 boolean 值
| `(int x, int y) ->`<br> `{ System.out.printn("Result"); `<br>`System.out.printn(x + y)}` |  表达式具有两个 int 类型的参数而没有返回值（void返回），**Lambda 表达式可以包含多行语句，但必须要使用大括号包起来。**
| `() -> 42` |  表达式没有参数，返回一个 int 类型的值。
| `(Apple o1, Apple o2) ->`<br> `Integer.valueOf(o1.getWeight())`<br>`.compareTo`<br>`(Integer.valueOf(o2.getWeight()))` |  表达式具有两个 Apple 类型的参数，返回一个 int 比较重要。

下面提供一些 Lambda 表达式的使用案例：


| 使用案例 | Lambda 示例 |
| --- | --- |
| 布尔表达式 | `(List<String> list) -> list.isEmpty()` |
| 创建对象 | `() -> new Apple(10)` |
| 消费对象 | `(Apple a) -> { System.out.println(a.getWeight) }` |
| 从一个对象中选择/抽取 | `(String s) -> s.lenght()` |
| 组合两个值 | `(int a, int b) -> a * b` |
| 比较两个对象 | ``(Apple o1, Apple o2) ->`<br> `Integer.valueOf(o1.getWeight())`<br>`.compareTo(Integer.valueOf(o2.getWeight()))` |

# 如何使用 Lambda

到底在哪里可以使用 Lambda 呢？你可以在**函数式接口上使用 Lambda 表达式**。

## 函数式接口

**函数式接口**就是只定义一个抽象方法的接口，比如 Java API 中的 Predicate、Comparator 和 Runnable 等。

```java

public interface Predicate<T> {
    boolean test(T t);
}

public interface Comparator<T> {
    int compare(T o1, T o2);
}

public interface Runnable {
    void run();
}
```
用函数式接口可以干什么呢？Lambda 表达式允许你直接以内联的形式为函数式接口的抽象方法提供实现，并把整个表达式作为函数式接口的实例（具体说来，是函数式接口一个具体实现 的实例）。你用匿名内部类也可以完成同样的事情，只不过比较笨拙：需要提供一个实现，然后 再直接内联将它实例化。下面的代码是有效的，因为Runnable是一个只定义了一个抽象方法run 的函数式接口：

```java

//使用Lambda
Runnable r1 = () -> System.out.println("Hello World 1");

//匿名类
Runnable r2 = new Runnable(){ 
    public void run(){ 
        System.out.println("Hello World 2"); 
    } 
};

public static void process(Runnable r){ 
    r.run(); 
} 

process(r1); //打印 "Hello World 1"
process(r2); //打印 "Hello World 2"
//利用直接传递的 Lambda 打印 "Hello World 3"
process(() -> System.out.println("Hello World 3"));
```

## 函数描述符

函数式接口的抽象方法的签名基本上就是 Lambda 表达式的签名。我们将这种抽象方法叫作**函数描述符**。例如，Runnable 接口可以看作一个什么也不接受什么也不返回（void）的函数的签名，因为它只有一个叫作 run 的抽象方法，这个方法什么也不接受，什么也不返回（void）。

# Lambda 实践

让我们通过一个例子，看看在实践中如何利用Lambda和行为参数化来让代码更为灵活，更为简洁。

资源处理（例如处理文件或数据库）时一个常见的模式就是打开一个资源，做一些处理，然后关闭资源。这个设置和清理阶段总是很类似，并且会围绕着执行处理的那些重要代码。这就是所谓的**环绕执行（execute around）模式**。

例如，在以下代码中，高亮显示的`BufferedReader reader = new BufferedReader(new FileReader("data.txt"))`就是从一个文件中读取一行所需的模板代码（注意你使用了Java 7中的带资源的try语句，它已经简化了代码，因为你不需要显式地关闭资源了）。

```java 
public static String processFile() throws IOException {

   try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
       return reader.readLine();
   }
}
```

## 第1步：行为参数化

现在上述代码是有局限的。你只能读文件的第一行。如果你想要返回头两行，甚至是返回使用最频繁的词， 该怎么办呢？在理想的情况下， 你要重用执行设置和清理的代码， 并告诉 processFile 方法对文件执行不同的操作。是的，你需要把 processFile 的行为参数化，你需要一种方法把行为传递给 processFile ， 以便它可以利用 BufferedReader执行不同的行为。

传递行为正是 Lambda 的优势。那要是想一次读两行，这个新的processFile方法看起来又该是什么样的呢? 你需要一个接收BufferedReader并返回String的Lambda。例如，下面就是从 BufferedReader 中打印两行的写法：

```java
String result = processFile((BufferedReader r) -> r.readLine() +r.readLine());
```

## 第2步：函数式接口传递行为

Lambda 仅可用于上下文是函数式接口的情况。你需要创建一个能匹配 `BufferedReader -> String`，还可以抛出 IOException 异常的接口。让我们把这一接口称为 BufferedReaderProcessor。

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader reader) throws IOException;
}
```

## 第3步：执行一个行为

任何`BufferedReader -> String`形式的 Lambda 都可以作为参数来传递，因为它们符合 BufferedReaderProcessor 接口中定义的 process 方法的签名。现在只需要编写一种方法在 processFile主体内执行 Lambda 所代表的代码。

```java
public static String processFile(BufferedReaderProcessor processor) throws IOException {

   try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
       return processor.process(reader); //处理 BufferedReader 对象
   }
}
```

## 第4步：传递 Lambda

现在就可以通过传递不同的 Lambda 重用 processFile 方法，并以不同的方式处理文件了。

```java
//打印一行
String result = processFile((BufferedReader r) -> r.readLine());
System.out.println(result);

//打印2行
result = processFile((BufferedReader r) -> r.readLine() +r.readLine());
```

# 使用函数式接口

Java 8的库帮你在`java.util.function`包中引入了几个新的函数式接口。我们接下来介绍 Predicate、Consumer和Function 三种函数式接口。

## Predicate

`java.util.function.Predicate<T>`接口定义了一个名叫 test 的抽象方法，它接受泛型 T对象，并返回一个 boolean。这恰恰和你先前创建的一样，现在就可以直接使用了。在你需要 表示一个涉及类型T的布尔表达式时，就可以使用这个接口。比如，你可以定义一个接受String 对象的Lambda表达式，如下所示。

```java
@FunctionalInterface 
public interface Predicate<T>{ boolean test(T t); }

public static <T> List<T> filter(List<T> list, Predicate<T> p) { 

   List<T> results = new ArrayList<>(); 
   for(T s: list){ 
       if(p.test(s)){  
           results.add(s); 
       } 
    }
    return results;
}

Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
```

## Consumer

`java.util.function.Consumer<T>` 定义了一个名叫 accept 的抽象方法，它接受泛型 T 的对象，没有返回（void）。你如果需要访问类型T的对象，并对其执行某些操作，就可以使用 这个接口。比如，你可以用它来创建一个forEach方法，接受一个Integers的列表，并对其中 每个元素执行操作。在下面的代码中，你就可以使用这个forEach方法，并配合Lambda来打印 列表中的所有元素。

```java
@FunctionalInterface 
public interface Consumer<T> { 
    void accept(T t); 
} 

public static <T> void forEach(List<T> list, Consumer<T> c){
    for(T i: list){ 
    c.accept(i); 
    }
}

forEach(Arrays.asList(1,2,3,4,5), (Integer i) -> System.out.println(i) );
```

## Function

`java.util.function.Function<T, R>`接口定义了一个叫作apply的方法，它接受一个
泛型 T 的对象，并返回一个泛型 R 的对象。如果你需要定义一个Lambda，将输入对象的信息映射到输出，就可以使用这个接口（比如提取苹果的重量，或把字符串映射为它的长度）。在下面的代码中，我们向你展示如何利用它来创建一个map方法，以将一个String列表映射到包含每个 String长度的Integer列表。

```java
@FunctionalInterface 
public interface Function<T, R>{
    R apply(T t); 
} 

public static <T, R> List<R> map(List<T> list, Function<T, R> f) {

    List<R> result = new ArrayList<>();

    for(T s: list) {
        result.add(f.apply(s));
    }
    return result; 
} 

// [7, 2, 6] 
List<Integer> l = map( Arrays.asList("lambdas","in","action"), (String s) -> s.length() );
```

## 原始类型特化

Java类型要么是引用类型（比如Byte、Integer、Object、List），要么是原始类型（比如int、double、byte、char）。但是泛型（比如Consumer<T>中的T）只能绑定到引用类型。这是由泛型内部的实现方式造成的。因此，在Java里有一个将原始类型转换为对应的引用类型的机制。这个机制叫作**装箱（boxing）**。相反的操作，也就是将引用类型转换为对应的原始类型，叫作**拆箱（unboxing）**。Java还有一个自动装箱机制来帮助程序员执行这一任务：装箱和拆箱操作是自动完成的。比如一个int被装箱成为 Integer。**但这在性能方面是要付出代价的**。装箱后的值本质上就是把原始类型包裹起来，并保存在堆里。因此，装箱后的值需要更多的内存，并需要额外的内存搜索来获取被包裹的原始值。

Java 8为我们前面所说的函数式接口带来了一个专门的版本，以便在输入和输出都是原始类型时避免自动装箱的操作。比如，使用 IntPredicate 就避免了对值 1000 进行装箱操作，但要是用 Predicate<Integer> 就会把参数 1000 装箱到一个 Integer 对象中。一般来说，针对专门的输入参数类型的函数式接口的名称都要加上对应的原始类型前缀，比 如 DoublePredicate、IntConsumer、LongBinaryOperator、IntFunction等。Function 接口还有针对输出参数类型的变种：ToIntFunction<T>、IntToDoubleFunction等。

## 常用的函数式接口

下表中列出 Java 8 中常用的函数式接口：

| 函数式接口 | 函数描述符 | 原始类型特化 |
| --- | --- | --- |
| `Predicate<T>` | T -> boolean | IntPredicate,LongPredicate, DoublePredicate |
| `Consumer<T>` | T -> void | IntConsumer,LongConsumer, DoubleConsumer |
| `Function<T,R>` | T -> R | `IntFunction<R>, IntToDoubleFunction, IntToLongFunction, LongFunction<R>, LongToDoubleFunction, LongToIntFunction, DoubleFunction<R>, ToIntFunction<T>, ToDoubleFunction<T>, ToLongFunction<T>` |
| `Supplier<T>` | () -> T | BooleanSupplier,IntSupplier, LongSupplier, DoubleSupplier |
| `UnaryOperator<T>` | T -> T | `IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator` |
| `BinaryOperator<T>` | (T,T) -> T | IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator |
| `BiPredicate<L,R>` | (L,R) -> boolean |  |
| `BiConsumer<T,U>` | (T,U) -> R | `ObjIntConsumer<T>, ObjLongConsumer<T>, ObjDoubleConsumer<T>` |
| `BiFunction<T,U,R>` | (T,U) -> R | `ToIntBiFunction<T,U>, ToLongBiFunction<T,U>, ToDoubleBiFunction<T,U>` |

# 类型检查、推断以及限制

## 类型检查

Lambda 的类型是从使用 Lambda 的上下文推断出来的。上下文（比如接受它传递的方法的参数，或接受它的值的局部变量）中 Lambda 表达式需要的类型称为**目标类型**。下图表示了代码的类型检查过程：

![](http://ooqmyazc5.bkt.clouddn.com/Lambda%E8%A1%A8%E8%BE%BE%E5%BC%8F%E7%9A%84%E7%B1%BB%E5%9E%8B%E6%A3%80%E6%9F%A5%E8%BF%87%E7%A8%8B.jpg)

类型检查过程可以分解为如下所示：

* 首先，找出 filter 方法的声明；
* 第二，找出目标类型 `Predicate<Apple>`。
* 第三，`Predicate<Apple>`是一个函数式接口，定义了一个叫作 test 的抽象方法。
* 第四，test 方法描述了一个函数描述符，它可以接受一个 Apple，并返回一个 boolean。
* 最后，filter 的任何实际参数都必须匹配这个要求。

## 同样的 Lambda，不同的函数式接口

用一个 Lambda 表达式就可以与不同的函数式接口联系起来，只要它们的抽象方法签名能够兼容。比如，前面提到的 Callable 和 PrivilegeAction，这两个接口都代表着什么也不接受且返回一个泛型 T 的函数。如下代码所示两个赋值时有效的：

```java
Callable<Integer> c = () -> 42;
PrivilegeAction<Integer> p = () -> 42;
```

**特殊的void兼容规则**如果一个Lambda的主体是一个语句表达式， 它就和一个返回void的函数描述符兼容（当然需要参数列表也兼容）。例如，以下两行都是合法的，尽管 List 的 add 方法返回了一个 boolean，而不是 Consumer 上下文（T -> void）所要求的void：

```java
//Predicate 返回一个 boolean
Predicate<String> p = s -> list.add(s);
//Consumer 返回一个 void
Consumer<String> b = s -> list.add(s);
```

## 类型推断

Java编译器会从上下文（目标类型）推断出用什么函数式接口来配合 Lambda 表达式，这意味着它也可以推断出适合Lambda 的签名，因为函数描述符可以通过目标类型来得到。这样做的好处在于，编译器可以了解Lambda表达式的参数类型，这样就可以在Lambda语法中省去标注参数类型。

```java
List<Apple> greenApples = filter(inventory, a -> "green".equals(a.getColor())); //参数a没有显示类型
Comparator<Apple> c = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()); //无类型推断
Comparator<Apple> c = (a1, a2) -> a1.getWeight().compareTo(a2.getWeight()); //类型推断
```

## 使用局部变量

Lambda表达式也允许使用自由变量（不是参数，而是在外层作用域中定义的变量），就像匿名类一样。 它们被称作**捕获Lambda**。例如，下面的Lambda捕获了portNumber变量：

```java
int num = 1337;
Runnable r = () -> System.out.println(num);
```
Lambda可以没有限制地捕获（也就是在其主体中引用）实例变量和静态变量。但局部变量必须显式声明为final， 或事实上是final。换句话说，Lambda表达式只能捕获指派给它们的局部变量一次。（注：捕获 实例变量可以被看作捕获最终局部变量this。） 例如，下面的代码无法编译，因为portNumber 变量被赋值两次：

```java
int portNumber = 1337; 
Runnable r = () -> System.out.println(portNumber); 
portNumber = 31337; //错误：Lambda表达式引用的局 部变量必须是最终的（final） 或事实上最终的
```

**为什么局部变量有这些限制？****第一**，实例变量和局部变量背后的实现有一 个关键不同。实例变量都存储在堆中，而局部变量则保存在栈上。如果Lambda可以直接访问局部变量，而且Lambda是在一个线程中使用的，则使用Lambda的线程，可能会在分配该变量的线程将这个变量收回之后，去访问该变量。因此，Java在访问自由局部变量时，实际上是在访问它的副本，而不是访问原始变量。如果局部变量仅仅赋值一次那就没有什么区别了——因此就有了这个限制。**第二**，这一限制不鼓励你使用改变外部变量的典型命令式编程模式（这种模式会阻碍很容易做到的并行处理）。

# 方法引用

方法引用让你可以重复使用现有的方法定义，并像Lambda一样传递它们。在一些情况下，比起使用 Lambda 表达式，它们似乎更易读，感觉也更自然。下面就是我们借助更新的Java 8 API，用方法引用写的一个排序的例子：

```java
lists.sort(comparing(Apple::getWeight);
```

## 如何使用

方法引用可以被看作仅仅调用特定方法的Lambda的一种快捷写法。它的基本思想是，如果一个Lambda代表的只是“直接调用这个方法”，那最好还是用名称来调用它，而不是去描述如何调用它。事实上，方法引用就是让你根据已有的方法实现来创建 Lambda表达式。但是，显式地指明方法的名称，你的代码的可读性会更好。它是如何工作的呢？ 当你需要使用方法引用时， **目标引用放在分隔符 :: 前， 方法的名称放在后面。** 例如， `Apple::getWeight`就是引用了Apple类中定义的方法`getWeight`。请记住，不需要括号，因为 你没有实际调用这个方法。方法引用就是Lambda表达式`(Apple a) -> a.getWeight()`的快捷写法，下表给出了Java 8中方法引用的其他一些例子。


| Lambda | 等效的引用方法 |
| --- | --- |
| `(Apple a) -> a.getWeight()` | Apple::getWeight |
| `() -> Thread.currentThread().dumpStack()` | Thread.currentThread()::dumpStack |
| `(str,i) -> str.substring(i)` | String::substring |
| `(String i) -> System.out.println(s)` | System.out::println |

## 分类

方法引用主要分为三类：

*   指向静态方法的引用（例如 Integer 的 parseInt 方法，写作 `Integer::parseInt`）
*   指向任意类型实例方法的方法引用（例如 String 的 length 方法，写作 `String::length`）
*   指向现有对象的实例方法的引用(假设有一个局部变量 expensiveTransaction 用于存放 Transaction 类型的对象，它支持实例方法 getValue，那么就可以写 `expensiveTransaction::getValue`)

>注意，编译器会进行一种与Lambda表达式类似的类型检查过程，来确定对于给定的函数 式接口，这个方法引用是否有效：方法引用的签名必须和上下文类型匹配。

## 构造函数引用

对于一个现有构造函数，可以利用它的名称和关键字 new 来创建它的一个引用：` ClassName::new`。它的功能与指向静态方法的引用类似。

例如，假设有一个构造函数没有参数。 它适合 Supplier 的签名`() -> Apple`。可以这样做：

```java
Supplier<Apple> c1 = Apple::new; //构造函数引用指向默认的 Apple() 构造函数
Apple a1 = c1.get(); //产生一个新的对象

//等价于：

Supplier<Apple> c1 = () -> new Apple(); //利用默认构造函数创建 Apple 的 Lambda 表达式
Apple a1 = c1.get();
```

如果你的构造函数的签名是`Apple(Integer weight)`，那么它就适合 Function 接口的签名，于是可以这样写：

```java
Function<Integer, Apple> c2 = Apple::new; //构造函数引用指向 Apple(Integer weight) 构造函数
Apple a2 = c2.apple(100);

//等价于：

Function<Integer, Apple> c2 = (Integer weight) -> new Apple(weight);
Apple a2 = c2.apple(100);
```

如果你有一个具有两个参数的构造函数`Apple(String color, Integer weight)`，那么它就适合BiFunction接口的签名，于是可以这样写：

```java
BiFunction<Integer, Integer, Apple> c3 = Apple::new; 
Apple a3 = c23.apple("green", 100);

//等价于：

BiFunction<Integer, Apple> c3 = (String color, Integer weight) -> new Apple(color, weight);
Apple a3 = c3.apple("green", 100);
```

# Lambda 和方法引用实战

## 第1步：传递代码

Java 8的API已经为你提供了一个 List 可用的 sort 方法，那么如何把排序策略传递给 sort 方法呢？sort方法的签名是这样的：

```java
void sort(Comparator<? super E> c)
```
它需要一个 Comparator 对象来比较两个Apple！这就是在Java中传递策略的方式：它们必须包裹在一个对象里。**我们说 sort 的行为被参数化了**：传递给它的排序策略不同，其行为也会 不同。
第一个解决方案可以是这样的：

```java
public class AppleComparator implements Comparator<Apple> {

        @Override
        public int compare(Apple o1, Apple o2) {
            return o1.getWeight().compareTo(o2.getWeight());
        }
}

apples.sort(new AppleComparator())
```

## 第2步：使用匿名类

可以使用匿名类来改进方案，而不是实现一个 Comparator 却只实例化一次：

```java
apples.sort(new Comparator<Apple>() {
  @Override
  public int compare(Apple o1, Apple o2) {
      return o1.getWeight().compareTo(o2.getWeight());
  }
});
```

## 第3步：使用 Lambda 表达式

接下来使用 Lambda 表达式来改进方案：

```java
apples.sort((Apple a1,Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));
```
Comparator 具有一个叫作 comparing 的静态辅助方法，它可以接受一个 Function 来提取 Comparable 键值，并生成一个 Comparator 对象，它可以像下面这样用（注意你现在传递的Lambda只有一 个参数：Lambda说明了如何从苹果中提取需要比较的键值）：

```java
apples.sort(Comparator.comparing(((Apple apple) -> apple.getWeight())));
```

## 第4步：使用方法引用

方法引用就是替代那些转发参数的 Lambda 表达式的语法糖。可以用方法引 用改进方案如下：

```java
apples.sort(Comparator.comparing(Apple::getWeight));
```

# 复合 Lambda 表达式

## 比较器复合

*   **逆序：**Comparator 接口有一个默认方法 reversed 可以使给定的比较器逆序。

```java
apples.sort(Comparator.comparing(Apple::getWeight).reversed()); //按重量递减排序
```

*   **比较器链：**Comparator 接口的 thenComparing 方法接受一个函数作为参数（就像 comparing方法一样），如果两个对象用第一个Comparator比较之后是相等的，就提供第二个 Comparator。

```java
apples.sort(Comparator.comparing(Apple::getWeight).reversed().thenComparing(Apple::getColor)); //按重量递减排序，一样重时，按颜色排序
```

## 谓词复合

谓词接口包括三个方法：negate、and和or。

```java
 //苹果不是红的
Predicate<Apple> notRedApple = redApple.negate();

//苹果是红色并且重量大于150
Predicate<Apple> redAndHeavyApple = redApple.and(a -> a.getWeight() > 150); 

//要么是150g以上的红苹果，要么是绿苹果
Predicate<Apple> redAndHeavyAppleOrGreen = redApple.and(a -> a.getWeight() > 150) .or(a -> "green".equals(a.getColor()));
```

## 函数复合

Function 接口的 andThen 方法`Function<T, V> andThen(Function<? super R, ? extends V> after)`会返回一个函数，它先计算 andThen 的调用函数，将输入函数的结果应用于 andThen 方法的 after 函数。

```java
Function<Integer, Integer> f = x -> x + 1; 
Function<Integer, Integer> g = x -> x * 2; 
Function<Integer, Integer> h = f.andThen(g); //g(f(x))
int result = h.apply(1); //result = 4
```

Function 接口的 Compose 方法`Function<V, R> compose(Function<? super V, ? extends T> before)`先计算 compose 的参数里面给的那个函数，然后再把结果用于 compose 的调用函数。

```java
Function<Integer, Integer> f = x -> x + 1; 
Function<Integer, Integer> g = x -> x * 2; 
Function<Integer, Integer> h = f.compose(g); //f(g(x))
int result = h.apply(1); //result = 3
```

# 小结

* Lambda表达式可以理解为一种匿名函数：它没有名称，但有参数列表、函数主体、返回 类型，可能还有一个可以抛出的异常的列表。
* Lambda表达式让你可以简洁地传递代码。
* **函数式接口**就是仅仅声明了一个抽象方法的接口。
* 只有在接受函数式接口的地方才可以使用Lambda表达式。
* Lambda表达式允许你直接内联，为函数式接口的抽象方法提供实现，并且将整个表达式作为函数式接口的一个实例。
* Java 8自带一些常用的函数式接口，放在`java.util.function`包里，包括`Predicate<T>、Function<T,R>、Supplier<T>、Consumer<T>`和`BinaryOperator<T>`。
* 为了避免装箱操作，对`Predicate<T>`和`Function<T, R>`等通用函数式接口的原始类型特化：IntPredicate、IntToLongFunction等。
* 环绕执行模式（即在方法所必需的代码中间，你需要执行点儿什么操作，比如资源分配 和清理）可以配合 Lambda 提高灵活性和可重用性。
* Lambda 表达式所需要代表的类型称为目标类型。
* 方法引用让你重复使用现有的方法实现并直接传递它们。
* Comparator、Predicate和Function等函数式接口都有几个可以用来结合 Lambda 表达式的默认方法。


# 参考资料

《Java 8 实战》

