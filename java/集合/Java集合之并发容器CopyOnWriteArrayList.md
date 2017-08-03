Copy-On-Write简称COW，是一种用于程序设计中的优化策略。其基本思路是，从一开始大家都在共享一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后在此基础上进行修改，这是一种延时懒惰策略。从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器，它们是CopyOnWriteArrayList和CopyOnWriteArraySet。CopyOnWrite容器非常有用，可以在非常多的并发场景中使用到。

# 概念

CopyOnWrite容器即写时复制的容器。通俗的理解是当我向一个容器中添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后往新的容器里添加元素，添加元素后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素，所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

# 继承关系

```
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

# 实现原理

## add()

以下代码是向CopyOnWriteArrayList中添加元素add方法的实现，可以发现在添加的时候是需要加锁的，否则多线程写的时候会复制出N个副本出来。

```Java
public boolean add(E e) {
	final ReentrantLock lock = this.lock;
	lock.lock();
	try {
		Object[] elements = getArray();
		int len = elements.length;
		Object[] newElements = Arrays.copyOf(elements, len + 1);
		newElements[len] = e;
		setArray(newElements);
		return true;
	} finally {
		lock.unlock();
	}
}
```
## get()

而读数据的时候不需要加锁，如果读的时候有多个线程正在向CopyOnWriteArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的CopyOnWriteArrayList。

```Java
public E get(int index) {
	return get(getArray(), index);
}
```

# 应用场景

CopyOnWrite并发容器用于读多写少的并发场景，比如白名单、黑名单、商场类目的访问和更新场景。

使用CopyOnWrite需要注意两件事情：
1. 减少扩容开销。根据时间需要初始化CopyOnWriteArrayList的大小，避免写数据时扩容的开销；
2. 使用批量添加。因为每次添加，容器每次都会进行复制，所以减少添加次数，可以减少容器的复制次数。

# 缺点

## 内存占用问题

因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。之前我们系统中使用了一个服务由于每晚使用CopyOnWrite机制更新大对象，造成了每晚15秒的Full GC，应用响应时间也随之变长。

　　针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。或者不使用CopyOnWrite容器，而使用其他的并发容器如ConcurrentHashMap。

## 数据一致性问题

CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

参考文章[Java并发编程：并发容器之CopyOnWriteArrayList](http://www.cnblogs.com/dolphin0520/p/3938914.html)

下面这篇文章验证了CopyOnWriteArrayList和同步容器的性能：
[http://blog.csdn.net/wind5shy/article/details/5396887](http://blog.csdn.net/wind5shy/article/details/5396887)

下面这篇文章简单描述了CopyOnWriteArrayList的使用:
[http://blog.csdn.net/imzoer/article/details/9751591](http://blog.csdn.net/imzoer/article/details/9751591)

