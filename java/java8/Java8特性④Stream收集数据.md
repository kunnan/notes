**Java8特性④Stream收集数据**
[TOC]

>收集器可以简洁而灵活地定义collect用来生成结果集合的标准。更具体地说，对流调用 collect 方法将对流中的元素触发一个归约操作（由Collector来参数化）。一般来说，Collector 会对元素应用一个转换函数（很多时候是不体现任何效果的恒等转换， 例如 toList ），并将结果累积在一个数据结构中，从而产生这一过程的最终输出。下面就来学习那些可以从Collectors 类提供的工厂方法（例如groupingBy）创建的收集器。

# 归约和汇总

## 查找流中的最大值和最小值

Collectors.maxBy 和 Collectors.minBy 来计算流中的最大或最小值。

```java
Optional<Dish> maxDish = Dish.menu.stream().
      collect(Collectors.maxBy(Comparator.comparing(Dish::getCalories)));
maxDish.ifPresent(System.out::println);

Optional<Dish> minDish = Dish.menu.stream().
      collect(Collectors.minBy(Comparator.comparing(Dish::getCalories)));
minDish.ifPresent(System.out::println);
```

## 汇总

Collectors.summingInt 汇总求和；
Collectors.averagingInt 汇总求平均值；
Collectors.summarizingInt 汇总所有信息包括数量、求和、平均值、最小值、最大值；

```java
//求总热量
int totalColories = Dish.menu.stream().collect(Collectors.summingInt(Dish::getCalories));
System.out.println(totalColories);

//求平均热量
double averageColories = Dish.menu.stream().collect(Collectors.averagingInt(Dish::getCalories));
System.out.println(averageColories);

//汇总
IntSummaryStatistics menuStatistics = Dish.menu.stream().collect(Collectors.summarizingInt(Dish::getCalories));
System.out.println(menuStatistics);
IntSummaryStatistics{count=9, sum=4300, min=120, average=477.777778, max=800}
```

## 连接字符串

joining 工厂方法返回的收集器会把对流中每一个对象应用toString方法得到的所有字符串连接成一个字符串。

```java
String menu = Dish.menu.stream().map(Dish::getName).collect(Collectors.joining(","));
System.out.println(menu);
//pork,beef,chicken,french fries,rice,season fruit,pizza,prawns,salmon
```

## Collectors.reducing

Collectors.reducing 工厂方法是上面所有工厂方法的一般情况，它完全可以实现上述方法的功能。它需要三个参数：

*   第一个参数是归约操作的起始值，也是流中没有元素时的返回值，所以很显然对于数值和而言0是一个合适的值。 
*   第二个参数是一个 Function，就是具体的取值函数。
*   第三个参数是一个 BinaryOperator，将两个项目累积成一个同类型的值。。

```java
int totalCalories = Dish.menu.stream().collect(Collectors.reducing( 0, Dish::getCalories, (i, j) -> i + j));
```

# 分组

用Collectors.groupingBy工厂方法返回的收集器可以实现分组任务，分组操作的结果是一个Map，把分组函数返回的值作为映射的键，把流中 所有具有这个分类值的项目的列表作为对应的映射值。

## 多级分组

```java
//Dish的Type为键，Dish类型所对应的dish集合为值
Map<Dish.Type, List<Dish>> dishesByType = Dish.menu.stream().collect(Collectors.groupingBy(Dish::getType));
System.out.println(dishesByType);
//{FISH=[prawns, salmon], OTHER=[french fries, rice, season fruit, pizza], MEAT=[pork, beef, chicken]}

//一级分类为Dish的Type，二级分类为Dish的CaloricLevel
Map<Dish.Type, Map<Dish.CaloricLevel, List<Dish>>> dishes = Dish.menu.stream()
      .collect(Collectors.groupingBy(Dish::getType, Collectors.groupingBy(Dish::getLevel)));
System.out.println(dishes);
//{FISH={NORMAL=[salmon], DIET=[prawns]}, OTHER={NORMAL=[french fries, pizza], DIET=[rice, season fruit]}, MEAT={NORMAL=[beef], FAT=[pork], DIET=[chicken]}}
```

## 按子集收集数据

```java
//Dish的Type为键，Dish类型所对应的dish集合的size为值
Map<Dish.Type, Long> dishTypeCount = Dish.menu.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.counting()));
System.out.println(dishTypeCount);
//{FISH=2, OTHER=4, MEAT=3}
```

# 分区

分区是分组的特殊情况：由一个谓词（返回一个布尔值的函数）作为分类函数，它称分区函数。分区函数返回一个布尔值，这意味着得到的分组 Map 的键类型是 Boolean，于是它最多可以分为两组——true是一组，false是一组。分区的好处在于保留了分区函数返回true或false的两套流元素列表。

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> partitioningDish = Dish.menu.stream().collect(Collectors.partitioningBy(Dish::isVegetarian, Collectors.groupingBy(Dish::getType)));
System.out.println(partitioningDish);
//false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, 
//true={OTHER=[french fries, rice, season fruit, pizza]}
```

# 小结

下表展示 Collectors 类的静态工厂方法。

| 工厂方法 | 返回类型 | 作用 
| --- | --- | --- 
| toList | `List<T>` | 把流中所有项目收集到一个 List 
| toSet | `Set<T>` | 把流中所有项目收集到一个 Set，<br>删除重复项 
| toCollection | `Collection<T>` | 把流中所有项目收集到给定的<br>供应源创建的集合<br>menuStream.collect(toCollection(), ArrayList::new)
| counting | Long | 计算流中元素的个数 
| sumInt | Integer | 对流中项目的一个整数属性求和 
| averagingInt | Double | 计算流中项目 Integer 属性的平均值 
| summarizingInt | IntSummaryStatistics | 收集关于流中项目 Integer 属性的统计值，<br>例如最大、最小、 总和与平均值 
| joining | String | 连接对流中每个项目调用 toString<br>方法所生成的字符串`collect(joining(", "))` 
| maxBy | `Optional<T>` | 一个包裹了流中按照给定比较器<br>选出的最大元素的 Optional,<br> 或如果流为空则为 Optional.empty() 
| minBy | `Optional<T>` | 一个包裹了流中按照给定比较器<br>选出的最小元素的Optional，<br> 或如果流为空则为Optional.empty() 
| reducing | 归约操作产生的类型 | 从一个作为累加器的初始值开始，<br>利用 BinaryOperator 与流中的元素逐个结合，<br>从而将流归约为单个值累加<br>int totalCalories = <br>menuStream.collect(reducing(0, Dish::getCalories, Integer::sum)); 
| collectingAndThen | 转换函数返回的类型 | 包裹另一个收集器，<br>对其结果应用转换函数<br>int howManyDishes = menuStream.collect(<br>collectingAndThen(toList(), List::size))
| groupingBy | `Map<K, List<T>>` | 根据项目的一个属性的值对流中的项目作问组，<br>并将属性值作为结果 Map 的键 
| partitioningBy | `Map<Boolean,List<T>>` | 根据对流中每个项目应用谓词的结果<br>来对项目进行分区 


# 附录：Dish类

```java
/**
 * Created by liuguoquan on 2017/4/26.
 */
public class Dish {

    private String name;
    private boolean vegetarian;
    private int calories;
    private Type type;
    private CaloricLevel level;

    public CaloricLevel getLevel() {

        if (calories <= 400) {

            return CaloricLevel.DIET;
        } else if (calories <= 700) {

            return CaloricLevel.NORMAL;
        }
        return CaloricLevel.FAT;
    }

    public void setLevel(CaloricLevel level) {
        this.level = level;
    }

    public enum Type { MEAT, FISH, OTHER }
    public enum CaloricLevel { DIET, NORMAL, FAT }

    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    public String getName() {
        return name;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public int getCalories() {
        return calories;
    }

    public Type getType() {
        return type;
    }

    @Override
    public String toString() {
        return name;
    }

    public static final List<Dish> menu =
            Arrays.asList( new Dish("pork", false, 800, Dish.Type.MEAT),
                    new Dish("beef", false, 700, Dish.Type.MEAT),
                    new Dish("chicken", false, 400, Dish.Type.MEAT),
                    new Dish("french fries", true, 530, Dish.Type.OTHER),
                    new Dish("rice", true, 350, Dish.Type.OTHER),
                    new Dish("season fruit", true, 120, Dish.Type.OTHER),
                    new Dish("pizza", true, 550, Dish.Type.OTHER),
                    new Dish("prawns", false, 400, Dish.Type.FISH),
                    new Dish("salmon", false, 450, Dish.Type.FISH));
}
```

