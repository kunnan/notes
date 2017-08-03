[TOC]

# 签名（Signature）

```
public class HashMap<K,V>
       extends AbstractMap<K,V>
       implements Map<K,V>, Cloneable, Serializable

```
可以看到HashMap实现了：

-	接口Cloneable，用于表明HashMap对象会重写`java.lang.Object.clone()`方法，HashMap实现的是浅拷贝
-	接口Serializable：表明HashMap对象可以被序列化

# Map接口

Map接口里包含的成员方法不外乎是“增删改查”，Map虽然并不是Collection，但它提供了三种“集合视角”，与下面三个方法一一对应：

-	`Set<key> keySet()`，提供key的集合视角
-	`Collection<V> values()`，提供value的集合视角
-	`Set<Map.Entry<K,V>> entrySet()`,提供key-value键值对的集合视角

# 设计理念

## 哈希表（hash table）

HashMap是一种基于哈希表实现的Map，哈希表是一种通用的数据结构，其概念是：key经过hash函数作用后得到一个槽（buckets）的索引（index），槽中保存着我们想要获取的值，如下图所示：

![hash table](http://7xs7a3.com1.z0.glb.clouddn.com/hashmap-%E5%93%88%E5%B8%8C%E8%A1%A8.png)

>一些不同的key经过同一hash函数后可能产生相同的索引，也就会产生冲突，所以利用哈希表这种数据结构实现具体类时，需要注意两个问题：
-	设计一个好的hash函数，使冲突尽可能的减少
-	需要解决发生冲突后的处理

## HashMap的特点

-	线程非安全，并且允许key与value都为null值，HashTable与之相反，为线程安全，key与value都不允许null值
-	不保证其内部元素的顺序，而且随着时间的推移，同一元素的位置也可能改变（resize的情况）
-	put、get操作的时间复杂度为O（1）
-	遍历其集合视角的时间复杂度与其容量和现有元素的大小成正比，如果遍历的性能要求很高，不要把capacity设置的过高或者把平衡因子设置的过低。
-	由于HashMap是线程非安全的，意味着如果有多个线程同时对同一HashMap试图做迭代时有结构上的改变（添加、删除entry，只改变entry的value值不算结构改变），那么会报ConcurrentModificationException异常，专业术语叫fail-fast，尽早报错对应多线程程序来说是很有必要的。
-	`Map m = Collections.synchronizedMap(new HashMap(...))`;通过这种方式可以得到一个线程安全的Map。

# 实现原理

## 构造函数

HashMap遵循集合框架的约束，提供一个参数为空的构造函数与有一个参数且参数类型为Map的构造函数。除此之外，还提供了两个构造函数，用于设置HashMap的容量（capacity）和平衡因子（loadFactor）。

```Java
public HashMap() {
	this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}
	
public HashMap(Map<? extends K, ? extends V> m) {
	this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
				  DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
	inflateTable(threshold);

	putAllForCreate(m);
}

public HashMap(int initialCapacity) {
	this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
	//初始容量和加载因子合法校验
	if (initialCapacity < 0)
		throw new IllegalArgumentException("Illegal initial capacity: " +
										   initialCapacity);
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	if (loadFactor <= 0 || Float.isNaN(loadFactor))
		throw new IllegalArgumentException("Illegal load factor: " +
										   loadFactor);

	this.loadFactor = loadFactor;
	threshold = initialCapacity;
	init();
}
```

容量与平衡因子都有个默认值，并且容量有个最大值

```Java
/**
 * 默认初始容量为16，必须为2的指数倍
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * 最大容量为2的30次方
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * 默认加载因子为0.75f
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

    // Entry数组，长度必须为2的n次幂
    transient Entry[] table;

	// 已存储元素的数量
	transient int size ;

	// 下次扩容的临界值，size>=threshold就会扩容，threshold等于capacity*load factor
	int threshold;

	// 加载因子
	final float loadFactor ;
```
可以看到，默认的平衡因子为0.75，这是权衡了时间复杂度与空间复杂度之后的最好取值（官方说法），过高的因子会降低存储空间但是查找的时间就会增加。

此外，我们注意到容量必须为2的指数被（默认16），这是为什么呢？解答这个问题，需要了解HashMap中哈希函数的设计原理

## 哈希函数的设计原理

```Java
final int hash(Object k) {
	int h = hashSeed;
	if (0 != h && k instanceof String) {
		return sun.misc.Hashing.stringHash32((String) k);
	}

	h ^= k.hashCode();

	// This function ensures that hashCodes that differ only by
	// constant multiples at each bit position have a bounded
	// number of collisions (approximately 8 at default load factor).
	h ^= (h >>> 20) ^ (h >>> 12);
	return h ^ (h >>> 7) ^ (h >>> 4);
}

/**
 * Returns index for hash code h.
 */
static int indexFor(int h, int length) {
	// assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
	return h & (length-1);
}
```
在哈希表容量为length的情况下，为了使key都能在冲突最小的情况下映射到[0,length)的索引（index）内，HasMap让length为2的指数倍，然后用`hashCode(key) & (length -1)的方法得到索引。

>因为length为2的指数倍，所以length-1所对应的二进制位都为1，然后与hashCode(key)坐与运算，即可得到[0,length)内的索引。

但是这里有个问题，如果HashCode（key)的值大于length的值，举个例子：
>Java中对象的哈希值都是32位整数，而HashMap的默认大小为16，那么如果有两个对象的哈希值为：0xABAB0000与0xBABA0000，它们的后四位都是一样，那么与16异或后得到结果都是一样的为0，也就是产生了冲突。

造成冲突的原因关键在于16限制了只能用低位来计算，高位直接舍弃了，所以我们需要额外的哈希函数而不只是简单的对象的hashCode方法了。具体来说就是HashMap中hash（）函数所实现的功能了。
>首先有个随机的hashSeed来降低冲突发生的几率
>然后如果是字符串。则用了sun.misc.Hashing.stringHash32((String) k)来获取索引值
>最后通过一系列的无符号右移操作，来把高位与地位进行异或操作，来降低冲突发生的几率。

右移的偏移量20,12,7是怎么来的呢？因为Java中对象的哈希值是32位的，所以这几个数应该就是把高位与地位做异或运算，至于这几个数是如何选取的，就不清楚了。

## HashMap.Entry

HashMap中存放的是HashMap.Entry对象，它继承自Map.Entry，其比较重要的构造函数

```Java
static class Entry<K,V> implements Map.Entry<K,V> {
	final K key;
	V value;
	Entry<K,V> next; //指向下一个节点
	int hash;

	/**
	 * Creates new entry.
	 */
	Entry(int h, K k, V v, Entry<K,V> n) {
		value = v;
		next = n;
		key = k;
		hash = h;
	}

	public final K getKey() {
		return key;
	}

	public final V getValue() {
		return value;
	}

	public final V setValue(V newValue) {
		V oldValue = value;
		value = newValue;
		return oldValue;
	}

	public final boolean equals(Object o) {
		if (!(o instanceof Map.Entry))
			return false;
		Map.Entry e = (Map.Entry)o;
		Object k1 = getKey();
		Object k2 = e.getKey();
		if (k1 == k2 || (k1 != null && k1.equals(k2))) {
			Object v1 = getValue();
			Object v2 = e.getValue();
			if (v1 == v2 || (v1 != null && v1.equals(v2)))
				return true;
		}
		return false;
	}

	public final int hashCode() {
		//用key的hash值与value的hash值与运算的结果作为Entry的hash值
		return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
	}

	public final String toString() {
		return getKey() + "=" + getValue();
	}

	/**
	 * 当向HashMap中添加元素时调用这个方法，这里没有实现是供子类回调
	 */
	void recordAccess(HashMap<K,V> m) {
	}

	/**
	 * 当从HashMap中删除元素时调用这个方法
	 */
	void recordRemoval(HashMap<K,V> m) {
	}
}
```
可以看到，Entry实现了单向链表的功能，用next成员变量来级联起来。也就是说HashMap的底层结构是一个数组，而数组的元素是一个单向链表。

介绍完Entry，下面介绍一个重要的成员变量

```
//HashMap内部维护一个数组类型的Entry变量table，用来保存添加进来的Entry对象
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```
Entry是单链表，怎么这里又需要个数组类型的tabl呢？其实这是解决冲突的一个方式：链地址法（开散列法），效果如下：

![](http://7xs7a3.com1.z0.glb.clouddn.com/hashmap-%E9%93%BE%E5%9C%B0%E5%9D%80%E6%B3%95.gif)

就是相同索引值的Entry会以单向链表的形式存在。
HashMap采用将相同的散列值存储到一个链表中，也就是说在一个链表中的元素他们的散列值绝对是相同的。


## put操作

因为put操作有可能需要对HashMap进行resize，所以实现较复杂

```Java
private void inflateTable(int toSize) {
    //辅助函数，用于填充HashMap到指定的capacity
    // Find a power of 2 >= toSize
    int capacity = roundUpToPowerOf2(toSize);
    //threshold为resize的阈值，超过后HashMap会进行resize，内容的entry会进行rehash
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
}
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 */
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
		
	//使用key的hashCode计算key对应的hash值
    int hash = hash(key);
	
	//通过key的哈希值查找在数组中的index位置
    int i = indexFor(hash, table.length);
    //这里的循环是关键
    //当新增的key所对应的索引i，对应table[i]中已经有值时，进入循环体
	//取出数组index位置的链表，遍历链表查看是否已经存在相同的key
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        //判断是否存在本次插入的key，如果存在用本次的value替换之前oldValue，相当于update操作
        //并返回之前的oldValue
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    //如果本次新增key之前不存在于HashMap中，modCount加1，说明结构改变了
    modCount++;
	//在数组i位置处添加一个新的链表节点
    addEntry(hash, key, value, i);
	//没有相同key的情况，返回null
    return null;
}

private V putForNullKey(V value) {
	for (Entry<K,V> e = table[0]; e != null; e = e.next) {
		if (e.key == null) {
			V oldValue = e.value;
			e.value = value;
			e.recordAccess(this);
			return oldValue;
		}
	}
	modCount++;
	addEntry(0, null, value, 0);
	return null;
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    //如果增加一个元素会后，HashMap的大小超过阈值，需要resize
    if ((size >= threshold) && (null != table[bucketIndex])) {
        //增加的幅度是之前的1倍
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }
    createEntry(hash, key, value, bucketIndex);
}
void createEntry(int hash, K key, V value, int bucketIndex) {
    //首先得到该索引处的冲突链Entries，第一次插入bucketIndex位置时冲突链为null，也就是e为null
    Entry<K,V> e = table[bucketIndex];
    //然后把新的Entry添加到冲突链的开头，也就是说，后插入的反而在前面（第一次还真没看明白）
    //table[bucketIndex]为新加入的Entry，是bucketIndex位置的冲突链的第一个元素
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
//下面看看HashMap是如何进行resize，庐山真面目就要揭晓了😊
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    //如果已经达到最大容量，那么就直接返回
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
	//使用新的容量创建一个新的链表数组
    Entry[] newTable = new Entry[newCapacity];
    //initHashSeedAsNeeded(newCapacity)的返回值决定了是否需要重新计算Entry的hash值
	//将当前数组的元素移动到新的数组
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
	//将当前数组的引用指向新的数组
    table = newTable;
	//重新计算临界值
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
/**
 * Transfers all entries from current table to newTable.
 */
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    //遍历当前的table，将里面的元素添加到新的newTable中
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            //最后这两句用了与put放过相同的技巧
            //将后插入的反而在前面
            newTable[i] = e;
            e = next;
        }
    }
}
/**
 * Initialize the hashing mask value. We defer initialization until we
 * really need it.
 */
final boolean initHashSeedAsNeeded(int capacity) {
    boolean currentAltHashing = hashSeed != 0;
    boolean useAltHashing = sun.misc.VM.isBooted() &&
            (capacity >= Holder.ALTERNATIVE_HASHING_THRESHOLD);
    //这里说明了，在hashSeed不为0或满足useAltHash时，会重算Entry的hash值
    //至于useAltHashing的作用可以参考下面的链接
    // http://stackoverflow.com/questions/29918624/what-is-the-use-of-holder-class-in-hashmap
    boolean switching = currentAltHashing ^ useAltHashing;
    if (switching) {
        hashSeed = useAltHashing
            ? sun.misc.Hashing.randomHashSeed(this)
            : 0;
    }
    return switching;
}
```

>Map中的元素越多，hash冲突的几率也就越大，数组长度是固定的，所以导致链表越来越长，那么查询的效率当然也就越地下了。HasMap的扩容resize，需要将所有的元素重新计算后，一个个重新排列到新的数组中去，这是非常低效的，和ArrayList一样，在可预知容量大小的情况下，提前预设容量会减少HashMap的扩容，提高性能。

## get操作

```Java
public V get(Object key) {
    //单独处理key为null的情况
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);
    return null == entry ? null : entry.getValue();
}
private V getForNullKey() {
    if (size == 0) {
        return null;
    }
    //key为null的Entry用于放在table[0]中，但是在table[0]冲突链中的Entry的key不一定为null
    //所以需要遍历冲突链，查找key是否存在
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }
    int hash = (key == null) ? 0 : hash(key);
    //首先定位到索引在table中的位置
    //然后遍历冲突链，查找key是否存在
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

## remove操作

```Java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    //可以看到删除的key如果存在，就返回其所对应的value
    return (e == null ? null : e.value);
}
final Entry<K,V> removeEntryForKey(Object key) {
    if (size == 0) {
        return null;
    }
    int hash = (key == null) ? 0 : hash(key);
    int i = indexFor(hash, table.length);
    //这里用了两个Entry对象，相当于两个指针，为的是防治冲突链发生断裂的情况
    //这里的思路就是一般的单向链表的删除思路
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;
    //当table[i]中存在冲突链时，开始遍历里面的元素
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
		//如果hash值和key都相等则认为相等
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e) //当冲突链只有一个Entry时
                table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }
    return e;
}
```
一般而言，认为HashMap的这四种操作时间复杂度O（1），因为它hash函数性质较好，保证了冲突发生的几率较小。

>从删除和查找操作可以看出，在根据key查找元素的时候，还是需要通过遍历，但是由于已经通过hash函数对key散列，要遍历的只是发生冲突后生成的链表，这样遍历的结果就已经少很多了，比完全遍历效率提升了N倍。

## fast-fail的HashIterator

集合类用Iterator类来遍历其包含的元素，接口Enumeration以及不推荐使用。相比Enumeration，Iterator有下面两个优势：
-	Iterator允许调用者在遍历集合类时删除集合类中包含的元素
-	比Enumeration命名更简单

HashMap中提供的三种集合视角，底层都是用HashIterator是实现的。

```Java
private abstract class HashIterator<E> implements Iterator<E> {
    Entry<K,V> next;        // next entry to return
    //在初始化Iterator实例时，纪录下当前的修改次数
    int expectedModCount;   // For fast-fail
    int index;              // current slot
    Entry<K,V> current;     // current entry
    HashIterator() {
        expectedModCount = modCount;
        if (size > 0) { // advance to first entry
            Entry[] t = table;
            //遍历HashMap的table，依次查找元素
            while (index < t.length && (next = t[index++]) == null)
                ;
        }
    }
    public final boolean hasNext() {
        return next != null;
    }
    final Entry<K,V> nextEntry() {
        //在访问下一个Entry时，判断是否有其他线程有对集合的修改
        //说明HashMap是线程非安全的
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if ((next = e.next) == null) {
            Entry[] t = table;
            while (index < t.length && (next = t[index++]) == null)
                ;
        }
        current = e;
        return e;
    }
    public void remove() {
        if (current == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        Object k = current.key;
        current = null;
        HashMap.this.removeEntryForKey(k);
        expectedModCount = modCount;
    }
}
private final class ValueIterator extends HashIterator<V> {
    public V next() {
        return nextEntry().value;
    }
}
private final class KeyIterator extends HashIterator<K> {
    public K next() {
        return nextEntry().getKey();
    }
}
private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {
    public Map.Entry<K,V> next() {
        return nextEntry();
    }
}
```

# 序列化

从源码可知，保存Entry的table数组为transient的，也就是说在进行序列化时并不会包含该成员，这是为什么呢？

```Java
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```
为了解答这个问题，我们需要明确下面事实：Object.hasCode方法对于一个类的两个实例返回的是不同的哈希值。

我们可以试想下面的场景：
>我们在机器A上算出对象A的哈希值与索引，然后把它插入到HashMap中，然后把该HashMap序列化后，在机器B上重新算出对象的哈希值与索引，这与机器A上算出的是不一样的，所以我们在机器B上get对象A时，会得到错误的结果。
>所以说，当序列化一个HashMap对象时，保存Entry的table是不需要序列化进来的，因为它在另一台机器上是错误的。
因为这个原因，HashMap重写了writeObject和readObject方法

# HashMap遍历

```Java
public class HashMapDemo {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Map<String, String> map = new HashMap<>();
		
		for(int i = 0;i < 4;i++ ) {
			
			map.put("name"+i, "liuguoquan"+i);
		}
		

		long start = System.nanoTime();
		//键的集合
		Set<String> set = map.keySet();
		Iterator<String> iterator = set.iterator();
		while (iterator.hasNext()) {
			String key = (String) iterator.next();
			String value = map.get(key);
			System.out.println(key + " = " + value);
			
		}
		long end = System.nanoTime();
		System.out.println("keySet(): " + (end - start)+"纳秒");
		
		//foreach keyset
		start = System.nanoTime();
		for(String key : map.keySet()) {
			String value = map.get(key);
			System.out.println(key + " = " + value);
		}
		end = System.nanoTime();
		System.out.println("for keySet(): " + (end - start)+"纳秒");
		
		//Entry集合 效率较高
		start = System.nanoTime();
		Set<Entry<String, String>> entrySet = map.entrySet();
		Iterator<Entry<String, String>> it = entrySet.iterator();
		while (it.hasNext()) {
			Map.Entry<String, String> entry = (Map.Entry<String, String>) it
					.next();
			String key = entry.getKey();
			String value = entry.getValue();
			System.out.println(key + " = " + value);
			
		}
		end = System.nanoTime();
		System.out.println("entrySet(): "+ (end - start)+"纳秒");
		
		start = System.nanoTime();
		//foreach entry 
		for(Entry<String, String> entry : map.entrySet()) {
			String key = entry.getKey();
			String value = entry.getValue();
			System.out.println(key + " = " + value);
		}
		end = System.nanoTime();
		System.out.println("for entrySet(): "+ (end - start)+"纳秒");
		
	}

}

结果打印：

name3 = liuguoquan3
name2 = liuguoquan2
name1 = liuguoquan1
name0 = liuguoquan0
keySet(): 403136纳秒
name3 = liuguoquan3
name2 = liuguoquan2
name1 = liuguoquan1
name0 = liuguoquan0
for keySet(): 84675纳秒
name3 = liuguoquan3
name2 = liuguoquan2
name1 = liuguoquan1
name0 = liuguoquan0
entrySet(): 109194纳秒
name3 = liuguoquan3
name2 = liuguoquan2
name1 = liuguoquan1
name0 = liuguoquan0
for entrySet(): 69850纳秒
```

从上面的结果来看：

-	HashMap遍历，如果既需要可以也需要value,直接用

```Java
for(Entry<String, String> entry : map.entrySet()) {
	String key = entry.getKey();
	String value = entry.getValue();
	System.out.println(key + " = " + value);
}
```
-	如果只是遍历key而无需value的话，可以直接用

```Java
for(String key : map.keySet()) {


}
```

参考文章：

[给jdk写注释系列之jdk1.6容器(4)-HashMap源码解析](http://www.importnew.com/17559.html)

[Java HashMap 源码解析](http://liujiacai.net/blog/2015/09/03/java-hashmap/)

[HashMap的实现原理](http://www.importnew.com/16301.html)

