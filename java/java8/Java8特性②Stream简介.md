**Java8特性②Stream简介**

[TOC]

# 流是什么

**流**是Java API的新成员，它允许你以声明性方式处理数据集合（通过查询语句来表达，而不是临时编写一个实现）。可以把它们看成遍历数据集的高级迭代器。此外流还可以**透明地并行处理**，无需写任何多线程代码了。如下面代码所示：

```java
public static List<String> getLowCalorisInJava8(List<Dish> dishes) {

    List<String> lowColorisDish = dishes.stream() //parallelStream() 并行流
                .filter((Dish d) -> d.getCalories() < 400) //筛选
                .sorted(Comparator.comparing(Dish::getCalories)) //排序
                .map(Dish::getName) //提取名称
                .collect(Collectors.toList()); //将所有名称存入List中

    return lowColorisDish;
}
```

Dish类

```java
package com.company.bean;

import java.util.Arrays;
import java.util.List;

/**
 * Created by liuguoquan on 2017/4/26.
 */
public class Dish {

    private String name;
    private boolean vegetarian;
    private int calories;
    private Type type;

    public enum Type { MEAT, FISH, OTHER }

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

# 流简介

流就是从支持数据处理操作的源生成的元素序列。

* 元素序列：流也提供了一个接口，可以访问特定元素类型的一组有序 值。因为集合是数据结构，所以它的主要目的是以特定的时间/空间复杂度存储和访问元 素（如 ArrayList 与 LinkedList ）。但流的目的在于表达计算， 比如你前面见到的 filter、sorted和map。集合讲的是数据，流讲的是计算。

* 源：流会使用一个提供数据的源，如集合、数组或输入/输出资源。 请注意，从有序集 合生成流时会保留原有的顺序。由列表生成的流，其元素顺序与列表一致。

* 数据处理操作：流的数据处理功能支持类似于数据库的操作，以及函数式编程语言中 的常用操作，如filter、map、reduce、find、match、sort等。流操作可以顺序执 行，也可并行执行。

流操作的两个重要的特点：

* **流水线：**很多流操作本身会返回一个流，这样多个操作就可以链接起来，形成一个大的流水线。流水线的操作可以 看作对数据源进行数据库式查询。

* **内部迭代：**与使用迭代器显式迭代的集合不同，流的迭代操作是在背后进行的。

# 流与集合

**集合与流之间的差异就在于什么时候进行计算。**集合是一个内存中的数据结构， 它包含数据结构中目前所有的值——集合中的每个元素都得先算出来才能添加到集合中。流则是在概念上固定的数据结构（你不能添加或删除元素），其元素则是**按需计算**的。

**流只能遍历一次。**遍历完之后，这个流已经被消费掉了。可以从原始数据源那里再获得一个新的流来重新遍历一遍。

```java
List<String> title = Arrays.asList("Java8", "In", "Action"); Stream<String> s = title.stream()
s.forEach(System.out::println); 
s.forEach(System.out::println); //java.lang.IllegalStateException:流已被操作
```

Collection接口需要用户去做迭代（比如用for-each），这称为**外部迭代**。Streams库使用内部迭代——它帮你把迭代做了，还把得到的流值存在了某个地方，你只要给出一个函数说要干什么就可以了。Streams库的内部迭代可以自动选择一种适 合你硬件的数据表示和并行实现。与此相反，一旦通过写for-each而选择了外部迭代，那你基 本上就要自己管理所有的并行问题了。

# 流操作

java.util.stream.Stream 中的 Stream 接口定义了许多操作。它们可以分为两大类。

*   **中间操作：**filter、map、limit等可以连成一条流水线的操作；
*   **终端操作：**collect等触发流水线执行并关闭流的操作；

## 中间操作

诸如filter或sorted等中间操作会返回另一个流。这让多个操作可以连接起来形成一个查 询。重要的是，除非流水线上触发一个终端操作，否则中间操作不会执行任何处理，这是因为中间操作一般都可以合并起来，在终端操作时一次性全部处理。尽管filter和map是两个独立的操作，但它们合并到同一次遍历中了（我们把这种技术叫作循环 合并）。

## 终端操作

终端操作会从流的流水线生成结果。其结果是任何不是流的值，比如List、Integer，甚至void。

## 使用流

流的使用一般包括三件事：

- 一个数据源（如集合）来执行一个查询；
- 一个中间操作链，形成一条流的流水线；
- 一个终端操作，执行流水线，并能生成结果；

流的流水线背后的理念类似于构建器模式。在构建器模式中有一个调用链用来设置一套配 置（对流来说这就是一个中间操作链），接着是调用built方法（对流来说就是终端操作）。

# 参考资料

《Java 8 实战》

