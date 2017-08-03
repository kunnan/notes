[TOC]

# ArrayList

## 概述

ArrayList是 Java 集合框架中使用最为普遍的集合类之一。ArrayList 是一种 List 实现，允许包括null在内的所有元素，它的内部用一个动态数组来存储元素，因此 ArrayList 能够在添加和移除元素的时候进行动态的扩展和缩减。

ArrayList有容量限制。超出限制时会增加50%容量，用System.arraycopy()复制到新的数组，因此最好能给出数组大小的预估值。默认第一次插入元素时创建大小为10的数组。

ArrayList按数组下标访问元素--get(i)/set(i,e) 的性能很高，这是数组的基本优势。

ArrayList直接在数组末尾加入元素--add(e)的性能也高，但如果按下标插入、删除元素--add(i,e), remove(i), remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是基本劣势。

## 定义
```Java
public class ArrayList<E>
	extends AbstractList<E>
	implements List<E>, RandomAccess, Cloneable, Serializable
```

## ArrayList实现

对应ArrayList而言，它实现List接口、底层使用数组保存所有元素，其操作基本上都是对数组的操作。下面我们来分析ArrayList的源码：

### 底层数组实现

```Java
private transient Object[] elementData;
```

### 构造方法

ArrayList提供了三种方式的构造器：
1. 构造一个默认初始容量为10的空列表；
2. 构造一个指定初始容量的空列表；
3. 构造一个包含指定collection集合的元素的列表，这些元素按照该collection的迭代器返回它们的顺序排列的。

```Java

//指定初始容量的空列表
public ArrayList(int initialCapacity) {
	super();
	if (initialCapacity < 0)
		throw new IllegalArgumentException("Illegal Capacity: "+
										   initialCapacity);
	this.elementData = new Object[initialCapacity];
}

/**
 * 初始容量为10的空列表.
 */
public ArrayList() {
	super();
	this.elementData = EMPTY_ELEMENTDATA;
}

public ArrayList(Collection<? extends E> c) {
	elementData = c.toArray();
	size = elementData.length;
	// c.toArray might (incorrectly) not return Object[] (see 6260652)
	if (elementData.getClass() != Object[].class)
		elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

### 存储

ArrayList提供了set、add、addAll这些方法添加元素

#### set()

```Java
// 用指定的元素替代此列表中指定位置上的元素，并返回以前位于该位置上的元素。  
public E set(int index, E element) {  
    RangeCheck(index);  
  
    E oldValue = (E) elementData[index];  
    elementData[index] = element;  
    return oldValue;  
}  
```

#### add()

```Java
// 将指定的元素添加到此列表的尾部。  
public boolean add(E e) {  
    ensureCapacity(size + 1);   
    elementData[size++] = e;  
    return true;  
} 

// 将指定的元素插入此列表中的指定位置。  
// 如果当前位置有元素，则向右移动当前位于该位置的元素以及所有后续元素（将其索引加1）。  
public void add(int index, E element) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException("Index: "+index+", Size: "+size);  
    // 如果数组长度不足，将进行扩容。  
    ensureCapacity(size+1);  // Increments modCount!!  
    // 将 elementData中从Index位置开始、长度为size-index的元素拷贝到从下标为index+1位置开始的新的elementData数组中。即将当前位于该位置的元素以及所有后续元素右移一个位置。  
    System.arraycopy(elementData, index, elementData, index + 1, size - index);  
    elementData[index] = element;  
    size++;  
}  
```

#### addAll()

```Java
// 按照指定collection的迭代器所返回的元素顺序，将该collection中的所有元素添加到此列表的尾部。  
public boolean addAll(Collection<? extends E> c) {  
    Object[] a = c.toArray();  
    int numNew = a.length;  
    ensureCapacity(size + numNew);  // Increments modCount  
    System.arraycopy(a, 0, elementData, size, numNew);  
    size += numNew;  
    return numNew != 0;  
}

// 从指定的位置开始，将指定collection中的所有元素插入到此列表中。  
public boolean addAll(int index, Collection<? extends E> c) {  
    if (index > size || index < 0)  
        throw new IndexOutOfBoundsException(  
            "Index: " + index + ", Size: " + size);  
  
    Object[] a = c.toArray();  
    int numNew = a.length;  
    ensureCapacity(size + numNew);  // Increments modCount  
  
    int numMoved = size - index;  
    if (numMoved > 0)  
        System.arraycopy(elementData, index, elementData, index + numNew, numMoved);  
  
    System.arraycopy(a, 0, elementData, index, numNew);  
    size += numNew;  
    return numNew != 0;  
} 
```

### 读取元素

```Java
// 返回此列表中指定位置上的元素。  
public E get(int index) {  
    RangeCheck(index);  
  
    return (E) elementData[index];  
}  
```

### 删除元素

```Java
// 移除此列表中指定位置上的元素。  
public E remove(int index) {  
    RangeCheck(index);  
  
    modCount++;  
    E oldValue = (E) elementData[index];  
  
    int numMoved = size - index - 1;  
    if (numMoved > 0)  
        System.arraycopy(elementData, index+1, elementData, index, numMoved);  
    elementData[--size] = null; // Let gc do its work  
  
    return oldValue;  
} 

// 移除此列表中首次出现的指定元素（如果存在）。这是应为ArrayList中允许存放重复的元素。  
public boolean remove(Object o) {  
    // 由于ArrayList中允许存放null，因此下面通过两种情况来分别处理。  
    if (o == null) {  
        for (int index = 0; index < size; index++)  
            if (elementData[index] == null) {  
                // 类似remove(int index)，移除列表中指定位置上的元素。  
                fastRemove(index);  
                return true;  
            }  
} else {  
    for (int index = 0; index < size; index++)  
        if (o.equals(elementData[index])) {  
            fastRemove(index);  
            return true;  
        }  
    }  
    return false;  
}  
```

注意：从数组中移除元素的操作，也会导致被移除的元素以后的所有元素的向左移动一个位置。

### 调整数组容量

每次向ArrayList中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出数组将会进行扩容，以满足添加数据的需求。数组扩容通过一个公开的方法ensureCapacity()来实现。在实际添加大量元素前，我们也可以使用该方法手动增加ArrayList的容量，以减少递增式再分配的数量。

```Java
public void ensureCapacity(int minCapacity) {
	//是否是默认容量
	int minExpand = (elementData != EMPTY_ELEMENTDATA) ? 0 : DEFAULT_CAPACITY;
	
	if (minCapacity > minExpand) {
		//扩容
		ensureExplicitCapacity(minCapacity);
	}
}

private void ensureExplicitCapacity(int minCapacity) {
	modCount++;

	//判断是否大于当前容量
	if (minCapacity - elementData.length > 0)
		grow(minCapacity);
}

private void grow(int minCapacity) {
	//
	int oldCapacity = elementData.length;
	//扩展容量为当前容量的一半
	int newCapacity = oldCapacity + (oldCapacity >> 1);
	
	//新容量小于最小容量要求
	if (newCapacity - minCapacity < 0)
		newCapacity = minCapacity;
	if (newCapacity - MAX_ARRAY_SIZE > 0)
		newCapacity = hugeCapacity(minCapacity);
	//
	elementData = Arrays.copyOf(elementData, newCapacity);
}
```

从上述代码可以看出，数组进行扩容时，会将老数组中的元素拷贝一份到新的数组中，每次数组容量的增长大约是其原容量的1.5倍。这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。当我们可预知要保存的元素有多少时，要在构造ArrayList实例时就指定其容量大小，以避免数组扩容的发送。或者根据实际需求，通过调用ensureCapacity()方法手动增加ArrayList实例的容量。

ArrayList还给我们提供了将底层数组的容量调整为当前列表保存的实际元素的大小的功能，可以通过trimToSize方法来实现：

```Java
public void trimToSize() {  
    modCount++;  
    int oldCapacity = elementData.length;  
    if (size < oldCapacity) {  
        elementData = Arrays.copyOf(elementData, size);  
    }  
}  
```

### Fail-Fast机制

ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会抛出失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

```Java
private void checkForComodification() {
	if (this.modCount != l.modCount)
		throw new ConcurrentModificationException();
}
```
## ArrayList遍历

有三种方法可以遍历ArrayList数组，分别是for、foreach、Iterator。

```Java
public class ArrayListDemo {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<String> list = new ArrayList<String>();

		for (int i = 0; i < 10; i++) {
			list.add("liu" + i);
		}

		// for循环  优先用这种方式
		int len = list.size();
		for (int i = 0; i < len; i++) {
			System.out.println("for: " + list.get(i));
		}

		// foreach语句
		for (String str : list) {

			System.out.println("foreach: " + str);
		}

		// 显示调用集合迭代器
		Iterator<String> it = list.iterator();
		while (it.hasNext()) {
			System.out.println("iterator while: " + it.next());
		}

		for (Iterator<String> iterator = list.iterator(); iterator.hasNext();) {

			System.out.println("iterator for: " + iterator.next());


		}

	}
}
```

# LinkedList

LinkedList是一种基于链表结构的一中List，具体是基于双向循环列表设计的。

## 定义

```Java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

## LinkedList实现

### 底层存储

```Java

transient int size = 0;  //元素数量

transient Node<E> first; //链表的头结点

transient Node<E> last; //链表的尾结点
```
Node表示链表的节点对象。

```Java
    private static class Node<E> {
        E item; //当前存储元素
        Node<E> next; //下一个元素节点
        Node<E> prev; //上一个元素节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
Node是LinkedList的内部类，其中定义了当前存储的元素，以及该元素的上一个元素和下一个元素。

### 构造方法

```Java
/*
* 构造一个空链表
*/
public LinkedList() {
}

/**
* 构造一个包含指定 collection 中的元素的列表，这些元素按其 collection 的迭代器返回的顺序排列
*/
public LinkedList(Collection<? extends E> c) {
	this();
	addAll(c);
}
```

### add()

```Java

//添加到链表末尾
public boolean add(E e) {
	linkLast(e);
	return true;
}

//在指定位置添加元素
public void add(int index, E element) {
	checkPositionIndex(index);

	if (index == size)
		linkLast(element);
	else
		linkBefore(element, node(index));
}
```

### addAll()

```Java

//添加一个集合
public boolean addAll(Collection<? extends E> c) {
	return addAll(size, c);
}


//在指定位置添加一个集合
public boolean addAll(int index, Collection<? extends E> c) {
	checkPositionIndex(index);

	Object[] a = c.toArray();
	int numNew = a.length;
	if (numNew == 0)
		return false;

	Node<E> pred, succ;
	if (index == size) {
		succ = null;
		pred = last;
	} else {
		succ = node(index);
		pred = succ.prev;
	}

	for (Object o : a) {
		@SuppressWarnings("unchecked") E e = (E) o;
		Node<E> newNode = new Node<>(pred, e, null);
		if (pred == null)
			first = newNode;
		else
			pred.next = newNode;
		pred = newNode;
	}

	if (succ == null) {
		last = pred;
	} else {
		pred.next = succ;
		succ.prev = pred;
	}
`
	size += numNew;
	modCount++;
	return true;
}
```

### 删除

```Java
public boolean remove(Object o) {
	if (o == null) {
		for (Node<E> x = first; x != null; x = x.next) {
			if (x.item == null) {
				unlink(x);
				return true;
			}
		}
	} else {
		for (Node<E> x = first; x != null; x = x.next) {
			if (o.equals(x.item)) {
				unlink(x);
				return true;
			}
		}
	}
	return false;
}

E unlink(Node<E> x) {
	// assert x != null;
	final E element = x.item;
	final Node<E> next = x.next;
	final Node<E> prev = x.prev;

	if (prev == null) {
		first = next;
	} else {
		prev.next = next;
		x.prev = null;
	}

	if (next == null) {
		last = prev;
	} else {
		next.prev = prev;
		x.next = null;
	}

	x.item = null;
	size--;
	modCount++;
	return element;
}
```

### 修改

```Java
public E set(int index, E element) {
	checkElementIndex(index);
	//查找index对应的节点
	Node<E> x = node(index);
	E oldVal = x.item;
	//替换旧元素
	x.item = element;
	return oldVal;
}
```

### 查询

```Java
public E get(int index) {
	checkElementIndex(index);
	return node(index).item;
}

Node<E> node(int index) {
	// assert isElementIndex(index);

	// 二分法
	if (index < (size >> 1)) {
		Node<E> x = first;
		for (int i = 0; i < index; i++)
			x = x.next;
		return x;
	} else {
		Node<E> x = last;
		for (int i = size - 1; i > index; i--)
			x = x.prev;
		return x;
	}
}
```
基于双向循环链表实现的LinkedList，通过索引Index的操作时低效的，index所对应的元素越靠近中间所费时间越长。而向链表两端插入和删除元素则是非常高效的（如果不是两端的话，都需要对链表进行遍历查找）。

# ArrayList vs LinkedList

-	ArrayList底层实现是数组，LinkedList实现是链表
-	ArrayList的查找效率高于LinkedList
-	LinkedList的增删效率高于ArrayList

ArrayList操作：

-	查询操作的时间复杂度是O(1)
-	增删操作的时间复杂度是O(n)

LinkedList：

-	查询操作的时间复杂度是O(n)
-	增删操作的时间复杂度是O(1)

# 面试题

## ArrayList的大小是如何自动增加的？

当我们试图在ArrayList中增加一个对象时，首先会检查ArrayList的容量，已确保已存在的数组中有足够的容量来存储新的对象。如果没有足够容量的话，就会新建一个长度更长的数组（长度是原数组长度的1.5倍），然后使用Arrays.copyOf()方法将旧的数组赋值到新的数组中去，并将现有的数组引用指向新的数组。

```Java
public boolean add(E e) {  
    ensureCapacity(size + 1);   
    elementData[size++] = e;  
    return true;  
} 

private void ensureCapacity(int minCapacity) {
	modCount++;

	//判断是否大于当前容量
	if (minCapacity - elementData.length > 0)
		grow(minCapacity);
}

private void grow(int minCapacity) {
	//
	int oldCapacity = elementData.length;
	//扩展容量为当前容量的一半
	int newCapacity = oldCapacity + (oldCapacity >> 1);
	
	//新容量小于最小容量要求
	if (newCapacity - minCapacity < 0)
		newCapacity = minCapacity;
	if (newCapacity - MAX_ARRAY_SIZE > 0)
		newCapacity = hugeCapacity(minCapacity);
	//返回一个新的数组对象，包括原数组中的内容
	elementData = Arrays.copyOf(elementData, newCapacity);
}
```

## 什么情况下使用ArrayList？什么情况下使用LinkedList？

多数情况下，当你遇到访问元素比插入或删除元素操作更频繁的时候，你应该使用ArrayList。当你遇到插入或者删除元素操作更加频繁，或者根本不需要访问元素的时候，你应该使用LinkedList。主要原因在于，在ArrayList中访问元素的最糟糕的时间复杂度为1，而在LinkedList中可能就是n；在LinkedList中插入和删除的时间复杂度为1，而在ArrayList中可能就是n。

## 当传递ArrayList到某个方法中，或者某个方法返回ArrayList，什么时候要考虑安全隐患？如何修护安全违规这个问题？

当ArrayList被当做参数传递到某个方法中，如果ArrayList在没有被复制的情况下直接被分配给成员变量，那么久可能发生这种情况，即当原始的ArrayList被改变时，传递到这个方法的数组也会改变。下面来看看实例：

安全隐患的情况：

```Java
public class ArrayListDemo {
	
	
	private static List<String> mList;

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<String> list = new ArrayList<String>();

		for (int i = 0; i < 3; i++) {
			list.add("liu" + i);
		}
	
		setList(list);
		
		//改变成员变量
		mList.set(0, "lee");
		
		for(String str : mList) {
			System.out.println("mList: "+str);
		}
		
		for(String str : list) {
			System.out.println("list: " + str);
		}
		
		System.out.println(list == mList);
		
		//改变原数组
		list.set(0, "Zhang");
		for(String str : mList) {
			System.out.println("mList: "+str);
		}
		
		for(String str : list) {
			System.out.println("list: " + str);
		}
		
		System.out.println(list == mList);

	}
	
	public static void setList(List<String> list) {
		mList = list;
	}
	
}

结果打印:

mList: lee
mList: liu1
mList: liu2
list: lee
list: liu1
list: liu2
true
mList: Zhang
mList: liu1
mList: liu2
list: Zhang
list: liu1
list: liu2
true
```
从结果可以看出，原数组和成员变量数组同时发送改变，这是因为在setList()方法中是将数组的引用赋值给了成员变量。

修复安全隐患后的代码为：

```Java
public class ArrayListDemo {
	
	
	private static List<String> mList;

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		List<String> list = new ArrayList<String>();

		for (int i = 0; i < 3; i++) {
			list.add("liu" + i);
		}
	
		setList(list);
		
		//改变成员变量
		mList.set(0, "lee");
		
		for(String str : mList) {
			System.out.println("mList: "+str);
		}
		
		for(String str : list) {
			System.out.println("list: " + str);
		}
		
		System.out.println(list == mList);
		
		//改变原数组
		list.set(0, "Zhang");
		for(String str : mList) {
			System.out.println("mList: "+str);
		}
		
		for(String str : list) {
			System.out.println("list: " + str);
		}
		
		System.out.println(list == mList);

	}
	
	public static void setList(List<String> list) {
		if (list == null) {
			mList = new ArrayList<String>();
		} else {
			//创建新的对象,并复制个mList
			mList = new ArrayList<String>(list);
		}
	}
	
}

打印:

mList: lee
mList: liu1
mList: liu2
list: liu0
list: liu1
list: liu2
false
mList: lee
mList: liu1
mList: liu2
list: Zhang
list: liu1
list: liu2
false
```
从结果可以看出，现在原数组和成员变量mList相互独立，改变自己的同时不会改变对方的数组内容。

>数组[]也是如此

## 如何复制一个ArrayList到另一个ArrayList中去?

1. 使用clone()方法，
2. 使用ArrayList构造方法，
3. 使用Collection的copy方法。

```Java
public class ArrayListDemo {
	
	

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ArrayList<String> list = new ArrayList<String>();

		for (int i = 0; i < 3; i++) {
			list.add("liu" + i);
		}
	
		
		//clone()
		ArrayList<String> list1 = (ArrayList<String>) list.clone();
		
		
		for(String str : list1) {
			System.out.println("list1: "+str);
		}
		
		System.out.println(list1 == list);
		
		//构造方法
		ArrayList<String> list2 = new ArrayList<String>(list);
		
		for(String str : list2) {
			System.out.println("list2: "+str);
		}
		
		System.out.println(list2 == list);
		
		//
		ArrayList<String> list3 = new ArrayList<String>(list.size());
		list3.add("1");
		list3.add("2");
		list3.add("3");
		
		Collections.copy(list3, list);
		
		for(String str : list3) {
			System.out.println("list3: "+str);
		}
		
		System.out.println(list3 == list);
	}

}

结果打印：

list1: liu0
list1: liu1
list1: liu2
false
list2: liu0
list2: liu1
list2: liu2
false
list3: liu0
list3: liu1
list3: liu2
false
```

## 在索引中ArrayList的增加或者删除某个对象的运行过程？效率很低吗？解释一下为什么？

在ArrayList中增加或删除元素的时候要调用System.arrayCopy()这个数值拷贝函数，每次增加或删除元素都要进行数组的拷贝操作，相对效率较低。如果遇到频繁插入或删除操作的时候，可以考虑使用LinkedList来代替。

在ArrayList的某个索引i处添加元素：

```Java
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

在ArrayList的某个索引i处删除元素：

```Java
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

