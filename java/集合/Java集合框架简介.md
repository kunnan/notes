# 集合框架

集合代表了一组对象。Java中的集合框架定义了一套规范，用来表示、操作集合，使具体操作与实现细节解耦。

# 两大基类Collection与Map

在集合框架的类继承体系中，最顶层有两个接口：

-	Collection表示一组纯数据
-	Map表示一组key-value键值对

一般继承自Collection或Map的集合类，会提供两个标准的构造函数：

-	没有参数的构造函数，创建一个空的集合类，如ArrayList()
-	有一个类型与基类（Collection或Map）相同的构造函数，创建一个与给定参数具有相同元素的新集合类，如ArrayList(Collection<? extends E> c)

## Collection

先看看常用的Collection集合类

![Collection](http://7xs7a3.com1.z0.glb.clouddn.com/Collection.png)

Collection类主要有三个接口：

-	Set表示不允许有重复元素的集合
-	List表示允许有重复元素的集合
-	Queue主要用于存储数据而不是处理数据

## Map

下图为常用的Map集合类

![Map](http://7xs7a3.com1.z0.glb.clouddn.com/Map.png)

Map并不是一个真正意义上的集合，但是这个接口提供了三种“集合视角”，使得可以像操作集合一样操作他们。具体如下：

-	把Map的内容看成key的集合
-	把Map的内容看成value的集合
-	把Map的内容看成key-value映射的集合

# Java集合与数据结构

## 数组

特点：可以随机访问，查询效率较高，增删效率较低、内存固定

Java集合：ArrayList、Vector

## 链表

特点：插入和删除效率高，查询效率低

Java集合：LinkedList、LinkedHashMap、LinkedHashSet

## 哈希表

特点：查找效率高，插入和删除较快，内存固定，存在散列冲突。

Java集合：HashMap、HashSet、HashTable、LinkedHashMap、LinkedHashSet

## 堆

特点：插入、删除效率高，对最大项、最小项存储快，其他项存取较慢。

Java集合：PriorityQueue（二叉堆实现的优先队列）

## 栈

特点：先进后出（FILO）

Java集合：Stack

## 队列

特点：先进先出（FIFO）

Java集合：ArrayDeque（双端队列）、LinkedList（双端队列）

## 树

特点：查询、删除、插入都比较快，但算法复杂

Java集合：TreeMap（红黑树）、TreeSet（红黑树）