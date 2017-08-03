TreeMap是基于红黑树结构的一种Map，要分析TreeMap的实现首先要了解红黑树这种数据结构。

# 二叉树、红黑树简介

先简单总结一下数组，链表，Hash表以及树的优缺点:

-	数组
	-	优点：随机访问效率高（根据下标查询）；搜索效率较高（可使用折半方法）
	-	缺点：内存连续且固定，存储效率低；插入和删除效率低（可能会进行数组扩容或者拷贝）

-	链表
	-	优点：不要求连续内存，存储效率高；插入和删除效率高（只需要改变指针指向）
	-	缺点：不支持随机访问；搜索效率低（需要遍历）

-	哈希表
	-	优点：搜索效率高；增删效率高
	-	缺点：内存利用率低（基于数组）；存在散列冲突

-	二叉树
	-	优点：查询效率高；增删效率高；存储效率高；
	-	缺点：算法复杂

## 二叉树

二叉树的特点：

-	若左子树不为空，则左子树上所有结点的值均小于它的根结点的值
-	若右子树不为空，则右子树上所有结点的值均大于它的根结点的值；
-	左右子树也分别为二叉查找树
-	没有键值相等的节点

![二叉树](http://7xs7a3.com1.z0.glb.clouddn.com/treemap-%E4%BA%8C%E5%8F%89%E6%A0%91.jpg)

按照二叉查找树存储的数据，对元素的搜索效率是非常高的，比如上图中如果要查找值为48的节点，只需要遍历4个节点就能完成。理论上，一颗平衡的二叉查找树的任意节点平均查找效率为树的高度h，即O(lgn)。但是如果二叉查找树的失去平衡（元素全在一侧），搜索效率就退化为O(n)，因此二叉查找树的平衡是搜索效率的关键所在。而红黑树就是靠红黑规则来维持二叉查找树的平衡性。

![失去平衡的二叉树](http://7xs7a3.com1.z0.glb.clouddn.com/treemap-%E5%A4%B1%E5%8E%BB%E5%B9%B3%E8%A1%A1%E7%9A%84%E4%BA%8C%E5%8F%89%E6%A0%91.png)


## 红黑树

红黑树的红黑规则：

-	节点是红色或黑色
-	根节点是黑色
-	每个叶子节点（NIL节点，空节点）是黑色的
-	每个红色节点的两个子节点都是黑色（从每个叶子到根的所有路径上不能有两个连续的红色节点）
-	从任意节点到其每个叶子的所有路径都包含相同数目的黑色节点

![s](http://7xs7a3.com1.z0.glb.clouddn.com/treemap-%E7%BA%A2%E9%BB%91%E6%A0%91.png)

第5条规则到底是什么情况，下面简单解释下，比如图中红8到1左边的叶子节点的路径包含两个黑色节点，到6下面的叶子节点的路径也包含两个黑色节点。

但是在在添加或删除节点后，红黑树就发生了变化，可能不再满足上面的5个特性，为了保持红黑树的以上特性，就有了三个动作：左旋、右旋、着色。

下面来看下什么是红黑树的左旋和右旋：

![](http://7xs7a3.com1.z0.glb.clouddn.com/treemap-%E7%BA%A2%E9%BB%91%E6%A0%91%E5%B7%A6%E6%97%8B.jpg)

对x进行左旋，意味着将x变成一个左节点。

![](http://7xs7a3.com1.z0.glb.clouddn.com/treemap-%E7%BA%A2%E9%BB%91%E6%A0%91%E5%8F%B3%E6%97%8B.jpg)

对y进行右旋，意味着将y变成一个右节点。

# TreeMap签名

```Java
public class TreeMap<K,V>
       extends AbstractMap<K,V>
       implements NavigableMap<K,V>, Cloneable, java.io.Serializable

```
>HashMap是无序的，TreeMap是有序的

## 接口NavigableMap

```Java
public interface NavigableMap<K,V> extends SortedMap<K,V>

```
发现NavigableMap继承了SortedMap，说明这个Map是有序的。这个顺序一般是指由Comparable接口提供的keys自然序，或者也可以在窗口SortedMap时，指定一个Comparator来决定。

## Comparator和Comparable的区别

-	Comparable一般表示类的自然序，比如定义一个Student类，学号为默认排序
-	Comparator一般表示类在某种场合下的特殊分类，需要定制化排序。比如现在想按照Student类的age来排序。

插入SortedMap中的key的类都必须继承Comparable类（或指定一个comparator），这样才能确定如何比较（通过k1.compareTo(k2)或comparator.compare(k1, k2)）两个key，否则，在插入时，会报ClassCastException的异常。

此外，SortedMap中key的顺序性应该与equals方法保持一致。也就是说k1.compareTo(k2)或comparator.compare(k1, k2)为true时，k1.equals(k2)也应该为true。

介绍完了SortedMap，再来回到我们的NavigableMap上面来。
NavigableMap是JDK1.6新增的，在SortedMap的基础上，增加了一些“导航方法”（navigation methods）来返回与搜索目标最近的元素。例如下面这些方法：

-	lowerEntry，返回所有比给定Map.Entry小的元素
-	floorEntry，返回所有比给定Map.Entry小或相等的元素
-	ceilingEntry，返回所有比给定Map.Entry大或相等的元素
-	higherEntry，返回所有比给定Map.Entry大的元素

# 总结

-	TreeMap的key是有序的，增删改查操作的时间复杂度为O(log(n))，为了保证红黑树平衡，在必要时会进行旋转
-	HashMap的key是无序的，增删改查操作的时间复杂度为O(1)，为了做到动态扩容，在必要时会进行resize。

参考文章:

[给jdk写注释系列之jdk1.6容器(7)-TreeMap源码解析](http://www.importnew.com/17605.html)

