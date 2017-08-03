
**Java8特性③Stream的使用**

[TOC]

# 筛选和切片

* filter 方法
* distinct 方法
* limit 方法
* skip 方法

## 谓词筛选

Stream 接口支持 filter 方法，该操作会接受一个谓词（一个返回 boolean的函数）作为参数，并返回一个包括所有符合谓词的元素的流。

```java
List<Dish> dishes = Dish.menu.stream()
      .filter(Dish::isVegetarian)
      .collect(Collectors.toList());
```

## 筛选重复的元素

Stream 接口支持 distinct 的方法， 它会返回一个元素各异（根据流所生成元素的 hashCode和equals方法实现）的流。例如，以下代码会筛选出列表中所有的偶数，并确保没有 重复。

```java
List<Integer> numbers = Arrays.asList(1,2,1,3,3,2,4);
numbers.stream().filter(i -> i % 2 == 0)
      .distinct() //去重元素2
      .forEach(System.out::println);
```

## 限制元素数量

Stream 支持limit(n)方法，该方法会返回一个不超过给定长度的流。所需的长度作为参数传递 给limit。如果流是有序的，则最多会返回前n个元素。

```java
List<Dish> dishLimits = Dish.menu.stream()
        .filter(d -> d.getCalories() > 300)
        .limit(3) //只返回符合要求的前3个元素
        .collect(Collectors.toList());
```

## 跳过指定数量的元素

Stream 支持 skip(n) 方法，返回一个扔掉了前n个元素的流。如果流中元素不足n个，则返回一 个空流。limit(n) 和 skip(n) 是互补的。

```java
List<Dish> dishSkip = Dish.menu.stream()
        .filter(d -> d.getCalories() > 300)
        .skip(2) //去掉符合要求的集合中的前2个元素后返回
        .collect(Collectors.toList());
dishSkip.forEach(System.out::println);
```

# 映射

## map 操作

Stream 支持 map 方法，它会接受一个函数作为参数。这个函数会被应用到每个元素上，并将其映 射成一个新的元素

```java
List<Integer> dishNames = Dish.menu.stream()
      .map(Dish::getName)
      .map(String::length)
      .collect(Collectors.toList());
 
List<String> words = Arrays.asList("Hello", "World");
List<Integer> wordLens = words.stream()
      .map(String::length) //转为字符串长度的集合
      .collect(Collectors.toList());
```

## flatMap 操作

flatmap 方法让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流。

```java
//使用flatMap找出单词列表中各不相同的字符
List<String> words = Arrays.asList("Hello", "World");
List<String> wordMap = words.stream()
      .map(word -> word.split(""))
      .flatMap(Arrays::stream)
      .distinct()
      .collect(Collectors.toList());
```
![](http://ooqmyazc5.bkt.clouddn.com/flatMap.jpg)

>给定两个数字列表，如何返回所有的数对呢？例如，给定列表[1, 2, 3]和列表[3, 4]，应该返回[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]。

```java
List<Integer> num1 = Arrays.asList(1, 2, 3);
List<Integer> num2 = Arrays.asList(4, 5);
List<int[]> pairs = num1.stream()
        .flatMap(i -> num2.stream().map(j -> new int[]{i, j}))
        .collect(Collectors.toList());
pairs.stream().forEach( i -> {
    Arrays.stream(i).forEach(System.out::println);
```

# 查找和匹配

## anyMatch

流中是否有一个元素能匹配给定的谓词。

```java
 if (Dish.menu.stream().anyMatch(Dish::isVegetarian)) {
     System.out.println("Vegetarion");
 }
```

## allMatch

流中是否有所有元素能匹配给定的谓词。

```java
 if (Dish.menu.stream().allMatch(d -> d.getCalories() < 1000)) {

     System.out.println("都有利于健康");
 }
```

## nonMatch

流中是否有没有任何元素能匹配给定的谓词。

```java
 if (Dish.menu.stream().noneMatch(d -> d.getCalories() >= 1000)) {

     System.out.println("都有利于健康");
 }
```

## findAny

findAny 方法将返回当前流中的任意一个元素。

```java
 Optional<Dish> dish = Dish.menu.stream().filter(Dish::isVegetarian)
         .findAny();
 dish.ifPresent(d -> System.out.println(d.toString()));
```

## findFirst

findAny 方法将返回当前流中的第一个元素。

```java
 List<Integer> num1 = Arrays.asList(1, 2, 3, 4, 5);
 num1.stream().map(x -> x * x)
         .filter(x -> x % 3 == 0) //平方能被3整除的数
         .findFirst().ifPresent(x -> System.out.println(x));
      }
```

## Optional

`Optional<T>`类（java.util.Optional）是一个容器类，代表一个值存在或不存在。Optional里面y有几种显式地检查值是否存在或处理值不存在的情形的方法：

* `isPresent()`将在Optional包含值的时候返回true, 否则返回false。
* `ifPresent(Consumer<T> block)`)会在值存在的时候执行给定的代码块。
* `T get()`会在值存在时返回值，否则抛出一个NoSuchElement异常。
* `T orElse(T other)`会在值存在时返回值，否则返回一个默认值。

# 归约（reduce）

把一个流中的元素组合起来，使用 reduce 操作来表达更复杂的查 询，比如“计算菜单中的总卡路里”或“菜单中卡路里最高的菜是哪一个”。此类查询需要将流中所有元素反复结合起来，得到一个值，比如一个Integer。这样的查询可以被归类为归约操作 （将流归约成一个值）。

reduce操作是如何作用于一个流的：Lambda反复结合每个元素，直到流被归约成一个值。reduce方法接受两个参数：一个初始值，这里是0；一个 `BinaryOperator<T>` 来将两个元素结合起来产生一个新值， 这里我们用的是 `lambda (a, b) -> a + b`。

## 元素求和

```java
List<Integer> numbers = Arrays.asList(3,4,5,1,2);
int sum1 = numbers.stream().reduce(0,(a, b) -> a + b);
System.out.println(sum1);

int sum2 = numbers.stream().reduce(0,Integer::sum);
System.out.println(sum2);
```

## 最大值

```java
int max = numbers.stream().reduce(0,Integer::max);
System.out.println(max);
```

## 最小值

```java
//reduce不接受初始值，返回一个Optional对象（考虑流中没有任何元素的情况）
Optional<Integer> min = numbers.stream().reduce(Integer::min);
min.ifPresent(System.out::println);
```

# 数值流

## 原始类型流特化

Java 8引入了三个原始类型特化流接口来解决这个问题： IntStream 、 DoubleStream 和 LongStream，分别将流中的元素特化为int、long和double，从而避免了暗含的装箱成本。每 个接口都带来了进行常用数值归约的新方法，比如对数值流求和的sum，找到最大元素的max。 此外还有在必要时再把它们转换回对象流的方法。这些特化的原因并不在于流的复杂性，而是装箱造成的复杂性——即类似int和Integer之间的效率差异。

- **映射到数值流：**将流转换为特化版本的常用方法是mapToInt、mapToDouble和mapToLong。这些方法和前 面说的map方法的工作方式一样，只是它们返回的是一个特化流，而不是`Stream<T>`。

```java
int colories = Dish.menu.stream()
        .mapToInt(Dish::getCalories) //返回IntStream
        .sum();
```

- **转换回对象流**

通过 box 方法可以将数值流转化为 Stream 非特化流。

```java
IntStream intStream = menu.stream().mapToInt(Dish::getCalories); //将Strean转化为数值流
Stream<Integer> stream = intStream.boxed(); //将数值流转化为Stream
```

- **默认值 OptionalInt**

Optional 可以用 Integer、String等参考类型来参数化。对于三种原始流特化，也分别有一个Optional原始类 型特化版本：OptionalInt、OptionalDouble和OptionalLong。

```java
Dish.menu.stream()
        .mapToInt(Dish::getCalories) //返回IntStream
        .max().ifPresent(System.out::println);
```

## 数值范围

```java
IntStream.rangeClosed(1, 100)
        .filter(x -> x % 10 == 0)
        .forEach(System.out::println);

```

Java 8引入了两个可以用于IntStream和LongStream的静态方法，帮助生成这种范围： range和rangeClosed。这两个方法都是第一个参数接受起始值，第二个参数接受结束值。但 range是不包含结束值的，而rangeClosed则包含结束值。

## 数值流应用：勾股数

生成 (5, 12, 13)、(6, 8, 10)和(7, 24, 25) 这样有效的勾股数数组集合。

```java
Stream<int[]> pythagoreanTriples = IntStream.rangeClosed(1, 100).boxed()
        .flatMap(a -> IntStream.rangeClosed(a,100)
                                .filter(b -> Math.sqrt(a * a + b * b) % 1 == 0).boxed()
                                .map(b -> new int[]{a,b,(int)Math.sqrt(a * a + b * b)})
        );
pythagoreanTriples.forEach(t -> System.out.println(t[0] + ";" + t[1] +";" + t[2]));
```

# 构建流

## 值创建流

```java
Stream<String> streams = Stream.of("Java", "Python");
streams.map(String::toUpperCase).forEach(System.out::println);

Stream.concat(Stream.of("Java", "Python"), Stream.of("C++", "Ruby")).forEach(System.out::println);
```

## 数组创建流

```java
int[] numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9};
int sum = Arrays.stream(numbers).sum();
```

## 文件生成流

```java
String ret = Files.lines(Paths.get("/Users/liuguoquan/Java/java8/src/com/company/data.txt"), Charset.defaultCharset())
        .reduce("", (a, b) -> a + " " + b);
```

## 函数生成流：创建无限流

```java
//迭代
Stream.iterate(0, n -> n + 2)
        .limit(10)
        .forEach(System.out::println);

//生成
Stream.generate(Math::random)
        .limit(5)
        .forEach(System.out::println);

    }
```

# 示例实战

假设你是执行交易的交易员。你的经理让你为八个查询找到答案。你能做到吗？

* (1) 找出2016年发生的所有交易，并按交易额排序（从低到高）。
* (2) 交易员都在哪些不同的城市工作过？
* (3) 查找所有来自于北京的交易员，并按姓名排序。
* (4) 返回所有交易员的姓名字符串，按字母顺序排序。
* (5) 有没有交易员是在深圳工作的？
* (6) 打印生活在北京的交易员的所有交易额。
* (7) 所有交易中，最高的交易额是多少？
* (8) 找到交易额最小的交易。

**交易员类**

```java
/**
 * 交易人
 * Created by liuguoquan on 2017/4/28.
 */
public class Trader {

    private String name;
    private String city;

    public Trader(String name, String city) {
        this.name = name;
        this.city = city;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    @Override
    public String toString() {
        return "Trader{" +
                "name='" + name + '\'' +
                ", city='" + city + '\'' +
                '}';
    }
}

```

**交易类**

```java
/**
 * 交易单
 * Created by liuguoquan on 2017/4/28.
 */
public class Transaction {

    private Trader trader;
    private int year;
    private int value;

    public Transaction(Trader trader, int year, int value) {
        this.trader = trader;
        this.year = year;
        this.value = value;
    }

    public Trader getTrader() {
        return trader;
    }

    public void setTrader(Trader trader) {
        this.trader = trader;
    }

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    public int getValue() {
        return value;
    }

    public void setValue(int value) {
        this.value = value;
    }

    @Override
    public String toString() {
        return "Transaction{" +
                "trader=" + trader +
                ", year='" + year + '\'' +
                ", value=" + value +
                '}';
    }
}

```

**计算**

```java
public class TransactionProcess {

    public static void main(String[] args) {

        Trader liu = new Trader("Lau","Beijing");
        Trader lee = new Trader("Lee","Shanghai");
        Trader zhang = new Trader("Zhang","Guangzhou");
        Trader wang = new Trader("Wang","Beijing");

        List<Transaction> transactions = Arrays.asList(
                new Transaction(liu,2016,300),
                new Transaction(lee,2015,100),
                new Transaction(lee,2016,500),
                new Transaction(zhang,2016,9000),
                new Transaction(wang,2017,1000),
                new Transaction(liu,2016,1500)
        );

        // (1) 找出2016年发生的所有交易，并按交易额排序（从低到高）。
        transactions.stream().filter(t -> t.getYear() == 2016)
                .sorted(Comparator.comparing(Transaction::getValue))
                .collect(Collectors.toList());

        // (2) 交易员都在哪些不同的城市工作过？
        transactions.stream().map(t -> t.getTrader().getCity())
                .distinct()
                .collect(Collectors.toList());

        // (3) 查找所有来自于北京的交易员，并按姓名排序。
        transactions.stream().map(t -> t.getTrader())
                .filter(t -> t.getCity().equals("Beijing"))
                .distinct()
                .sorted(Comparator.comparing(Trader::getName))
                .collect(Collectors.toList());

        // (4) 返回所有交易员的姓名字符串，按字母顺序排序。
        transactions.stream().map(t -> t.getTrader())
                .map(t -> t.getName())
                .distinct()
                .sorted()
                .collect(Collectors.toList());

        // (5) 有没有交易员是在深圳工作的？
        boolean isExist = transactions.stream().anyMatch(t -> t.getTrader().getCity().equals("Shenzhen"));
        if (isExist) {
            System.out.println("有在深圳工作的");
        } else {
            System.out.println("没有在深圳工作的");
        }

        // (6) 打印生活在北京的交易员的所有交易额。
        int sum = transactions.stream().filter(t -> t.getTrader().getCity().equals("Beijing"))
                .map(t -> t.getValue())
                .reduce(0,Integer::sum);
        System.out.println(sum);

        // (7) 所有交易中，最高的交易额是多少？
        int max = transactions.stream().map(t -> t.getValue())
                .reduce(0,Integer::max);
        System.out.println(max);

        // (8) 找到交易额最小的交易。
        int min = transactions.stream().map(t -> t.getValue())
                .reduce(0,Integer::min);
        System.out.println(min);
    }
}

```

# 小结

- 中间操作表

| 操作  | 类型 | 返回类型 | 目的 |
| --- | --- | --- | --- |
| filter | 中间操作 | `Stream<T>` | 过滤元素 |
| distinct | 中间操作 | `Stream<T>` | 过滤重复的元素 |
| skip | 中间操作 | `Stream<T>` | 跳过指定数量的元素 |
| limit | 中间操作 | `Stream<T>` | 限制元素的数量 |
| map | 中间操作 | `Stream<T> `| 流的转化 |
| flatmap | 中间操作 | `Stream<T>` | 流的扁平化 |
| sorted | 中间操作 | `Stream<T>` | 元素排序 |

- 终端操作表

| 操作 | 类型 | 返回类型 | 目的 |
| --- | --- | --- | --- |
| forEach | 终端操作 | void | 消费流中的每个元素，返回void |
| count | 终端操作 | long | 返回流中元素的个数，返回long |
| collect | 终端操作 | R | 把流归约为一个集合 |
| anyMatch | 终端操作 | boolean | 流中是否有符合要求的元素 |
| noneMatch | 终端操作 | boolean | 流中是否没有任何符合要求的元素  |
| allMatch | 终端操作 | boolean |  流中是否所有元素都是符合要求的 |
| findAny | 终端操作 | Optional<T> | 查找符合要求的元素  |
| findFirst | 终端操作 | Optional<T> | 查找第一个符合要求的元素  |
| reduce | 终端操作 | Optional<T> | 归约  |

