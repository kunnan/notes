[TOC]

# 简介
LinkedHashMap是HashMap的一个子类，它在HashMap的基础上维持了一个双向链表（hash表+双向链表），在遍历的时候可以使用插入顺序（先进先出），或者是最近最少使用（LRU）的顺序。

LinkedHashMap是key键有序的一种集合，使用双向链表来保证key的顺序。

# 特点

一般来说，如果需要使用的Map中的key无序，选择HashMap；如果要求key有序，则选择TreeMap。
但是选择TreeMap就会有性能问题，因为TreeMap的get操作的时间复杂度是O(log(n))的，相比于HashMap的O(1)还是差不少的，LinkedHashMap的出现就是为了平衡这些因素，使得能够以O(1)时间复杂度增加查找元素，又能够保证key的有序性。

此外，LinkedHashMap提供了两种key的顺序：
-	访问顺序（access order）。可以使用这种顺序实现LRU（Least Recently Used）缓存
-	插入顺序（insertion orde）。同一key的多次插入，并不会影响其顺序

# 签名

```Java
public class LinkedHashMap<K,V>
    extends HashMap<K,V> implements Map<K,V>
```
从定义可以看到LinkedHashMap继承于HashMap，且实现了Map接口。这也就意味着HashMap的一些优秀因素可以被继承下来，比如hash寻址，使用链表解决hash冲突等实现的快速查找，对于HashMap中一些效率较低的内容，比如容器扩容过程，遍历方式，LinkedHashMap是否做了一些优化呢。继续看代码吧。

# 底层存储

LinkedHashMap是基于HashMap，并在其基础上维持了一个双向链表，也就是说LinkedHashMap是一个hash表（数组+单向链表） +双向链表的实现，到底实现方式是怎么样的，来看一下：

```Java
//双向链表的头结点
private transient Entry<K,V> header ;

//true 表示最近较少使用顺序，false表示插入顺序
private final boolean accessOrder;
```

下面来看看Entry这个节点类：

```Java
/**
     * LinkedHashMap entry.
     */
    private static class Entry<K,V> extends HashMap.Entry<K,V> {
        // These fields comprise the doubly linked list used for iteration.
        // 双向链表的上一个节点before和下一个节点after
        Entry<K,V> before, after ;
 
       // 构造方法直接调用父类HashMap的构造方法（super）
       Entry( int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }
 
        /**
         * 从链表中删除当前节点的方法
         */
        private void remove() {
            // 改变当前节点前后两个节点的引用关系，当前节点没有被引用后，gc可以回收
            // 将上一个节点的after指向下一个节点
            before.after = after;
            // 将下一个节点的before指向前一个节点
            after.before = before;
        }
 
        /**
         * 在指定的节点前加入一个节点到链表中（也就是加入到链表尾部）
         */
        private void addBefore(Entry<K,V> existingEntry) {
            // 下面改变自己对前后的指向
            // 将当前节点的after指向给定的节点（加入到existingEntry前面嘛）
            after  = existingEntry;
            // 将当前节点的before指向给定节点的上一个节点
            before = existingEntry.before ;
 
            // 下面改变前后最自己的指向
            // 上一个节点的after指向自己
            before.after = this;
            // 下一个几点的before指向自己
            after.before = this;
        }
 
        // 当向Map中获取查询元素或修改元素（put相同key）的时候调用这个方法
        void recordAccess(HashMap<K,V> m) {
            LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
            // 如果accessOrder为true，也就是使用最近较少使用顺序
            if (lm.accessOrder ) {
                lm. modCount++;
                // 先删除，再添加，也就相当于移动了
                // 删除当前元素
                remove();
                // 将当前元素加入到header前（也就是链表尾部）
                addBefore(lm. header);
            }
        }
 
        // 当从Map中删除元素的时候调动这个方法
        void recordRemoval(HashMap<K,V> m) {
            remove();
        }
}
```
可以看到Entry继承了HashMap中的Entry，但是LinkedHashMap中的Entry多了两个属性指向上一个节点的before和指向下一个节点的after，也正是这两个属性组成了一个双向链表。等等。。。Entry还有一个继承下来的next属性，这个next是单向链表中用来指向下一个节点的，怎么回事嘛，怎么又是单向链表又是双向链表呢，都要晕了对不对，其实想的没错，这里的节点即是Hash表中的单向链表中的一个节点，它又是LinkedHashMap维护的双向链表中的一个节点。

![](http://7xs7a3.com1.z0.glb.clouddn.com/LinkedHashMap.png)

注：黑色箭头指向表示单向链表的next指向，红色箭头指向表示双向链表的before指向，蓝色箭头指向表示双向链表的after指向。另外LinkedHashMap种还有一个header节点是不保存数据的，这里没有画出来。

从上图可以看出LinkedHashMap仍然是一个Hash表，底层由一个数组组成，而数组的每一项都是个单向链表，由next指向下一个节点。但是LinkedHashMap所不同的是，在节点中多了两个属性before和after，由这两个属性组成了一个双向循环链表（你怎么知道是循环，下面在说喽），而由这个双向链表维持着Map容器中元素的顺序。看下Entry中的recordRemoval方法，该方法将在节点被删除时候调用，Hash表中链表节点被正常删除后，调用该方法修正由于节点被删除后双向链表的前后指向关系，从这一点来看，LinkedHashMap比HashMap的add、remove、set等操作要慢一些（因为要维护双向链表 ）。

# 源码解析

## 构造方法

```Java
/**
     * 构造一个指定初始容量和加载因子的LinkedHashMap，默认accessOrder为false
     */
    public LinkedHashMap( int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
 
    /**
     * 构造一个指定初始容量的LinkedHashMap，默认accessOrder为false
     */
    public LinkedHashMap( int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }
 
    /**
     * 构造一个使用默认初始容量(16)和默认加载因子(0.75)的LinkedHashMap，默认accessOrder为false
     */
    public LinkedHashMap() {
        super();
        accessOrder = false;  //默认false
    }
 
    /**
     * 构造一个指定map的LinkedHashMap，所创建LinkedHashMap使用默认加载因子(0.75)和足以容纳指定map的初始容量，默认accessOrder为false 。
     */
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super(m);
        accessOrder = false;
    }
 
    /**
     * 构造一个指定初始容量、加载因子和accessOrder的LinkedHashMap
     */
    public LinkedHashMap( int initialCapacity,
                      float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
}

//重写了HashMap的init方法
@Override
void init() {
        // 初始化话header，将hash设置为-1，key、value、next设置为null
        header = new Entry<K,V>(-1, null, null, null);
        // header的before和after都指向header自身
        header.before = header. after = header ;
}

```

## 增加

LinkedHashMap没有重写HashMap的put方法，只是重写了HashMap被put调用的addEntry方法

```Java
void addEntry( int hash, K key, V value, int bucketIndex) {
	// 调用createEntry方法创建一个新的节点
	createEntry(hash, key, value, bucketIndex);

	// Remove eldest entry if instructed, else grow capacity if appropriate
	// 取出header后的第一个节点（因为header不保存数据，所以取header后的第一个节点）
	Entry<K,V> eldest = header.after ;
	// 判断是容量不够了是要删除第一个节点还是需要扩容
	if (removeEldestEntry(eldest)) {
		// 删除第一个节点（可实现FIFO、LRU策略的Cache）
		removeEntryForKey(eldest. key);
	} else {
		// 和HashMap一样进行扩容
		if (size >= threshold)
			resize(2 * table.length );
	}
}

/**
 * This override differs from addEntry in that it doesn't resize the
 * table or remove the eldest entry.
 */
void createEntry( int hash, K key, V value, int bucketIndex) {
	// 下面三行代码的逻辑是，创建一个新节点放到单向链表的头部
	// 取出数组bucketIndex位置的旧节点 
	HashMap.Entry<K,V> old = table[bucketIndex];
	// 创建一个新的节点，并将next指向旧节点
   Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
	// 将新创建的节点放到数组的bucketIndex位置
	table[bucketIndex] = e;

	// 维护双向链表，将新节点添加在双向链表header前面（链表尾部）
	e.addBefore( header);
	// 计数器size加1
	size++;
}

/**
 * 默认返回false，也就是不会进行元素删除了。如果想实现cache功能，只需重写该方法
 */
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
	return false;
}
```

可以看到，在添加方法上，比HashMap中多了两个逻辑，一个是当Map容量不足后判断是删除第一个元素，还是进行扩容，另一个是维护双向链表。而在判断是否删除元素的时候，我们发现removeEldestEntry这个方法竟然是永远返回false，原来想要实现Cache功能，需要自己继承LinkedHashMap然后重写removeEldestEntry方法，这里默认提供的是容器的功能。

## 删除

LinkedHashMap没有重写remove方法，只是在实现了Entry类的recordRemoval方法，该方法是HashMap的提供的一个回调方法，在HashMap的remove方法进行回调，而LinkedHashMap中recordRemoval的主要当然是要维护双向链表了，返回上面去看下Entry类的recordRemoval方法吧。

## 查找

LinkedHashMap重写了get方法，但是的确复用了HashMap中的getEntry方法，LinkedHashMap是在get方法中指加入了调用recoreAccess方法的逻辑，recoreAccess方法的目的当然也是维护双向链表了，具体逻辑返回上面去看下Entry类的recoreAccess方法吧。

```Java
public V get(Object key) {
        Entry<K,V> e = (Entry<K,V>)getEntry(key);
        if (e == null)
            return null;
        e.recordAccess( this);
        return e.value ;
}
```
## 是否包含

```Java
public boolean containsValue(Object value) {
	// Overridden to take advantage of faster iterator
	// 遍历双向链表，查找指定的value
	if (value==null) { 
		for (Entry e = header .after; e != header; e = e.after )
			if (e.value ==null)
				return true;
	} else {
		for (Entry e = header .after; e != header; e = e.after )
			if (value.equals(e.value ))
				return true;
	}
	return false;
}
```
LinkedHashMap对containsValue进行了重写，我们在HashMap中说过，HashMap的containsValue需要遍历整个hash表，这样是十分低效的。而LinkedHashMap中重写后，不再遍历hash表，而是遍历其维护的双向链表，这样在效率上难道就有所改善吗？我们分析下：hash表是由数组+单向链表组成，而由于使用hash算法，可能会导致散列不均匀，甚至数组的有些项是没有元素的（没有hash出对应的散列值），而LinkedHashMap的双向链表呢，是不存在空项的，所以LinkedHashMap的containsValue比HashMap的containsValue效率要好一些。

# 自定义LruCache

```Java
package com.michael.java.construct;

import java.util.LinkedHashMap;
import java.util.Map;

public class LruCache extends LinkedHashMap<String, Object>{

	/**
	 * 
	 */
	private static final long serialVersionUID = -2725884916293330545L;
	
	private static final int DEFAULT_MAX_CAPACITY = 1024;
	private static final float DEFAULT_LOAD_FACTOR = 0.75f;
	
	private int maxCapacity;
	
	public LruCache(boolean accessOrder) {
		super(DEFAULT_MAX_CAPACITY, DEFAULT_LOAD_FACTOR, accessOrder);
		this.maxCapacity = DEFAULT_MAX_CAPACITY;
	}

	public LruCache(int capacity,boolean accessOrder) {
		super(DEFAULT_MAX_CAPACITY, DEFAULT_LOAD_FACTOR, accessOrder);
		this.maxCapacity = capacity;
	}
	
	@Override
	protected boolean removeEldestEntry(
			java.util.Map.Entry<String, Object> eldest) {
		// TODO Auto-generated method stub
		return this.size() > maxCapacity;
	}
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		LruCache cache = new LruCache(5, true);
		
		for(int i = 0;i < 6;i++) {
			cache.put("k"+i, "v"+i);
		}

		System.out.println("size: " + cache.size());
		
		for(Map.Entry<String, Object> entry : cache.entrySet()) {
			System.out.println(entry.getKey() + " = " + entry.getValue());
		}
		
		System.out.println("------------");
		System.out.println("k3 = " + cache.get("k3"));
		System.out.println("------------");
		
		cache.put("k6", "v6");
		
		for(Map.Entry<String, Object> entry : cache.entrySet()) {
			System.out.println(entry.getKey() + " = " + entry.getValue());
		}
	}

}

结果打印：

size: 5
k1 = v1
k2 = v2
k3 = v3
k4 = v4
k5 = v5
------------
k3 = v3
------------
k2 = v2
k4 = v4
k5 = v5
k3 = v3
k6 = v6
```
序列中的第一个元素时最近使用最少的元素

# 参考文章

[给jdk写注释系列之jdk1.6容器(5)-LinkedHashMap源码解析](http://www.importnew.com/17561.html)

[Java LinkedHashMap源码解析](http://liujiacai.net/blog/2015/09/12/java-linkedhashmap/)

