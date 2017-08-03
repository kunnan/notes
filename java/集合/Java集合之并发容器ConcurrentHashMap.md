[TOC]

# 术语定义

| 术语 | 英文 | 解释
|--------|--------|-------
|哈希算法| Hash algorithm|是一种将任意内容的输入转换成相同长度输出的加密方式，<br>其输出被称为哈希值
|哈希表|hash table |根据设定的哈希函数和处理冲突方法将一组关键字映射到一个有限的地址区间上，<br>并以关键字在地址区间中的项作为记录在表中的存储位置，<br>这种表称为哈希表或散列，所得存储位置称为哈希地址或散列地址。

# 线程不安全的HashMap

因为多线程环境下，使用HashMap进行put操作会引起使循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。

# 效率低下的HashTable

HashTable使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用个图方法来获取元素，所以竞争越激烈效率越低。

# ConcurrentHashMap锁分段技术

HashTable在竞争激烈的并发环境下表现出效率低下的原因是因为所有访问HashTable的线程都必须竞争同一把锁。

假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHasMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

# ConcurrentHasMap结构

先看看ConcurrentHasMap的类图

![](http://7xs7a3.com1.z0.glb.clouddn.com/ConcurrentHashMap-%E7%B1%BB%E5%9B%BE.jpg)

ConcurrentHasMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHasMap里扮演锁的角色，HashEntry则用于存储键值对数据。一个ConcurrentHasMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构，一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment的锁，由于每一个segment写操作只锁定自己的HashEntry数组，所以可能存在多个线程同时写的情况。

![](http://7xs7a3.com1.z0.glb.clouddn.com/ConcurrentHashMap-%E7%BB%93%E6%9E%84%E5%9B%BE.jpg)

# ConcurrentHasMap源码

## 初始化

ConcurrentHasMap初始化是通过initialCapacityu、loadFactor、concurrencyLevel几个参数来初始化Segment数组的，段偏移量segmentShift，段掩码segmentMask和每个segment里面的HashEntry数组。

## 初始化Segment数组

```Java

static final int DEFAULT_INITIAL_CAPACITY = 16;

//默认加载因子,加载因子是表示Hsah表中元素的填满的程度.若:加载因子越大,填满的元素越多,好处是,空间利用率高了,但:冲突的机会加大了.反之,加载因子越小,填满的元素越少,好处是:冲突的机会减小了,但:空间浪费多了.
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//默认并发等级
static final int DEFAULT_CONCURRENCY_LEVEL = 16;

//最大容量
static final int MAXIMUM_CAPACITY = 1 << 30;

//最小Segment
static final int MIN_SEGMENT_TABLE_CAPACITY = 2;

//最大Segment
static final int MAX_SEGMENTS = 1 << 16; // slightly conservative


public ConcurrentHashMap(int initialCapacity,
						 float loadFactor, int concurrencyLevel) {
	if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
		throw new IllegalArgumentException();
	if (concurrencyLevel > MAX_SEGMENTS)
		concurrencyLevel = MAX_SEGMENTS;
	// Find power-of-two sizes best matching arguments
	int sshift = 0;
	int ssize = 1;
	while (ssize < concurrencyLevel) {
		++sshift;
		//Segment数组大小 2的N次方
		ssize <<= 1;
	}
	this.segmentShift = 32 - sshift;
	this.segmentMask = ssize - 1;
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	int c = initialCapacity / ssize;
	if (c * ssize < initialCapacity)
		++c;
	int cap = MIN_SEGMENT_TABLE_CAPACITY;
	while (cap < c)
		cap <<= 1;
	// create segments and segments[0]
	Segment<K,V> s0 =
		new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
						 (HashEntry<K,V>[])new HashEntry[cap]);
	Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
	UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
	this.segments = ss; //初始化Segment数组
}
```

ssize数组长度是通过concurrencyLevel计算出来的。为了能通过按位与的哈希算法来定位segments数组的索引，必须保证segments的长度是2的N次方，所以必须计算出一个是大于或等于concurrencyLevel的最小2的N次方来作为segments数组的长度。假如concurrencyLevel等于14、15或16.ssize都会等于16，即容器里锁的个数也是16.注意concurrencyLevel的最大大小为65535，意味着segments数组的最大长度为65536，即2^16次方。

segmentShift用于定位参与hash运算的位数，segmentShift等于32减去sshift，sshift等于ssize从1向左移位的次数，在默认情况下concurrencyLevel等于16,1需要向左移动4位，所以sshift等于4，因此segmentShift等于28.
segmentMask是哈希运算的掩码，等于ssize-1，即15，掩码的二进制各个位的值都是1。
因为ssize的最长度是65536，所以segmentShift的最大值是16，segmentMask的最大值是65535，即2^16 -1，每位都是1.

## get操作

Segment的get操作实现非常简单和高效，先获取key的哈希值，然后使用这个哈希值通过哈希运算定位到segment，然后遍历该segment的HashEntry数组找到指定的key。

```Java
//jdk 1.7
public V get(Object key) {
	Segment<K,V> s; // manually integrate access methods to reduce overhead
	HashEntry<K,V>[] tab;
	int h = hash(key);
	//定位Segment
	long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
	if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
		(tab = s.table) != null) {
		//遍历HashEntry
		for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
				 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
			 e != null; e = e.next) {
			K k;
			if ((k = e.key) == key || (e.hash == h && key.equals(k)))
				return e.value;
		}
	}
	return null;
}
```
get操作的高效之处在于整个get过程不需要加锁，除非读到的值是空的才会加锁重读。

jdk1.7中的get方法没有使用锁同步，而是使用轻量级同步volatile原语sun.misc.Unsafe.getObjectVolatile(Object, long)，保证读到的是最新的对象。

jdk1.6中get方法里将要使用的共享变量都定义成volatile，如用于统计当前Segement大小的count字段和用于存储值的HashEntry的value。定义成volatile的变量，能够在线程之间保持可见性，能够被多线程同时读，并且保证不会读到过期的值，但是只能被单线程写（有一种情况可以被多线程写，就是写入的值不依赖于原值），在get操作里只需要读不需要写共享变量count和value，所以可以不用加锁。之所以不会读到过期的值，是根据java内存模型的happen before原则，对volatile字段的写入操作先于读操作，即使两个线程同时修改和获取volatile变量，get操作也能拿到最新的值，这是用volatile替换锁的经典应用场景。

## put操作

```Java
public V put(K key, V value) {
	Segment<K,V> s;
	if (value == null)
		throw new NullPointerException();
	int hash = hash(key);
	int j = (hash >>> segmentShift) & segmentMask;
	if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
		 (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
		s = ensureSegment(j);
	return s.put(key, hash, value, false);
}
```

如果段不为空，那么进入java.util.concurrent.ConcurrentHashMap.Segment.put(K, int, V, boolean)，否则构造段。由此可以看出段的构造是以懒加载的方式，按需构造。

```Java
  final V put(K key, int hash, V value, boolean onlyIfAbsent) {
		HashEntry<K,V> node = tryLock() ? null :
			scanAndLockForPut(key, hash, value);
		V oldValue;
		try {
			HashEntry<K,V>[] tab = table;
			int index = (tab.length - 1) & hash;
			HashEntry<K,V> first = entryAt(tab, index);
			for (HashEntry<K,V> e = first;;) {
				if (e != null) {
					K k;
					if ((k = e.key) == key ||
						(e.hash == hash && key.equals(k))) {
						oldValue = e.value;
						if (!onlyIfAbsent) {
							e.value = value;
							++modCount;
						}
						break;
					}
					e = e.next;
				}
				else {
					if (node != null)
						node.setNext(first);
					else
						node = new HashEntry<K,V>(hash, key, value, first);
					int c = count + 1;
					if (c > threshold && tab.length < MAXIMUM_CAPACITY)
						rehash(node);
					else
						setEntryAt(tab, index, node);
					++modCount;
					count = c;
					oldValue = null;
					break;
				}
			}
		} finally {
			unlock();
		}
		return oldValue;
	}
```
进入段中执行最终put的时候，会使用可重入锁进行tryLock可轮询请求锁，如果成功获取锁，那么条目的插入方式和普通hashmap没多大区别。
如果此时有其他线程也再对这个段进行更新操作，那么执行scanAndLockForPut进行重试。

重试的处理逻辑:

```Java
	private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
		HashEntry<K,V> first = entryForHash(this, hash);
		HashEntry<K,V> e = first;
		HashEntry<K,V> node = null;
		int retries = -1; // negative while locating node
		while (!tryLock()) {
			HashEntry<K,V> f; // to recheck first below
			if (retries < 0) {
				if (e == null) {
					if (node == null) // speculatively create node
						node = new HashEntry<K,V>(hash, key, value, null);
					retries = 0;
				}
				else if (key.equals(e.key))
					retries = 0;
				else
					e = e.next;
			}
			else if (++retries > MAX_SCAN_RETRIES) {
				lock();
				break;
			}
			else if ((retries & 1) == 0 &&
					 (f = entryForHash(this, hash)) != first) {
				e = first = f; // re-traverse if entry changed
				retries = -1;
			}
		}
		return node;
	}
```
一旦未获得锁 while (!tryLock()) 则进行重试循环。

第一次重试中retries < 0，如果桶条目不为空，那么遍历桶中条目链表，如果key已经存在，那么直接进入下一个循环，否则构造新条目，进入下一个循环；如果重试次数达到极限，那么使用阻塞同步方法；每隔一次循环，校验下所在桶有没有更新，如果更新了，那么重试次数重置，重新开始。
一旦获得锁，直接返回，进行常规的hash put操作。

总的来说，put的同步机制是如果没有其他线程在更新该段，那么直接put。否则轮询请求锁，直至获得锁。


# ConcurrentHasMap是弱一致性的迭代器

java.util.concurrent 集合返回的迭代器称为弱一致的（weakly consistent）迭代器

ConcurrentHashMap与其他并发容器所提供的多线程环境下不会抛出并发修改异常的迭代器是由其返回的弱一致性迭代器决定的，弱一致性迭代器可以容许并发修改。当迭代器创建的时，它会遍历已有元素，并且可以感应到在迭代器被创建后对容器的修改。这种弱一致性在调用那些需要对整个容器进行加锁的方法如size或isEmpty时可能提供不精确的值，因此只有当程序需要在独占访问中加锁时，才不能使用ConcurrentHashMap，而在绝大多数情况下ConcurrentHashMap可以带来更好的伸缩性。

参考文章

[《Java并发编程实践》笔记2——基础同步类](http://blog.csdn.net/chjttony/article/details/46608271)

[聊聊并发（4）：深入分析ConcurrentHashMap](http://ifeve.com/concurrenthashmap/)

