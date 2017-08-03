Java集合之并发容器ConcurrentLinkedQueue

[TOC]

在并发编程中我们有时候需要使用线程安全的队列。如果我们要实现一个线程安全的队列，有两种实现方式，一种是阻塞式队列，另外一种是非阻塞式队列。使用阻塞算法的队列可以用一个锁（入队和出对用同一把锁）或两个锁（入队和出对用不同的锁）等方式来实现，而非阻塞的实现则可以使用CAS的方式来实现。ConcurrentLinkedQueue是基于非阻塞式来实现的线程安全队列。

# 简介

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，当我们获取一个元素的时候，它会返回队列头部的元素。它采用了“wait-free”算法来实现。

# 签名

```Java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable
```

# 结构

首先来看看ConcurrentLinkedQueue的类图

![](http://7xs7a3.com1.z0.glb.clouddn.com/ConcurrentLinkedQueue-%E7%B1%BB%E5%9B%BE.jpg)

ConcurrentLinkedQueue由head节点和tail节点组成，每个节点（Node）由节点元素（item）和指向下一个节点的引用（next）组成，从而组成一张链表结构的队列。默认情况下head节点存储的元素为空，tail节点等于header节点

```Java
private transient volatile Node<E> head;

private transient volatile Node<E> tail;

public ConcurrentLinkedQueue() {
	head = tail = new Node<E>(null);
}
```

# offer入队列

入队列就是将新元素添加到队列的尾部，如下图所示

![](http://7xs7a3.com1.z0.glb.clouddn.com/ConcurrentLinekedQueue-%E9%98%9F%E5%88%97%E5%85%A5%E9%98%9F%E7%BB%93%E6%9E%84%E5%8F%98%E5%8C%96%E5%9B%BE.jpg)

-	第一步添加元素1。队列更新head节点的next节点为元素1节点。又因为tail节点默认情况下等于head节点，所以它们的next节点都指向元素1节点。
-	第二步添加元素2。队列首先设置元素1节点的next节点为元素2节点，然后更新tail节点指向元素2节点。
-	第三步添加元素3，设置tail节点的next节点为元素3节点。
-	第四步添加元素4，设置元素3的next节点为元素4节点，然后将tail节点指向元素4节点。

入队过程主要做两件事情：第一是将入队节点设置成当前队列尾节点的下一个节点。第二是更新tail节点，如果tail节点的next节点不为空，则将入队节点设置成tail节点；如果tail节点的next节点为空，则将入队节点设置成tail的next节点，所以tail节点不总是尾节点。

上面的分析让我们从单线程入队的角度来理解入队过程，但是多个线程同时进行入队情况就变得更加复杂，因为可能会出现其他线程插队的情况。如果有一个线程正在入队，那么它必须先获取尾节点，然后设置尾节点的下一个节点为入队节点，但这时可能有另外一个线程插队了，那么队列的尾节点就会发生变化，这时当前线程要暂停入队操作，然后重新获取尾节点。让我们再通过源码来详细分析下它是如何使用CAS算法来入队的。

源码分析：

```Java
public boolean offer(E e) {
	checkNotNull(e);
	//创建一个新节点
	final Node<E> newNode = new Node<E>(e);

	for (Node<E> t = tail, p = t;;) {
		Node<E> q = p.next;
		if (q == null) {
			// p is last node
			if (p.casNext(null, newNode)) {
				// Successful CAS is the linearization point
				// for e to become an element of this queue,
				// and for newNode to become "live".
				if (p != t) // hop two nodes at a time
					casTail(t, newNode);  // Failure is OK.
				return true;
			}
			// Lost CAS race to another thread; re-read next
		}
		else if (p == q)
			// We have fallen off list.  If tail is unchanged, it
			// will also be off-list, in which case we need to
			// jump to head, from which all live nodes are always
			// reachable.  Else the new tail is a better bet.
			p = (t != (t = tail)) ? t : head;
		else
			// Check for tail updates after two hops.
			p = (p != t && t != (t = tail)) ? t : q;
	}
}
```

# poll出队列

出队列的就是从队列里返回一个节点元素，并清空该节点对元素的引用。

![](http://7xs7a3.com1.z0.glb.clouddn.com/ConcurrentLinekedQueue-%E9%98%9F%E5%88%97%E5%87%BA%E9%98%9F%E7%BB%93%E6%9E%84%E5%8F%98%E5%8C%96%E5%9B%BE.jpg)

```Java
public E poll() {
	restartFromHead:
	for (;;) {
		for (Node<E> h = head, p = h, q;;) {
			E item = p.item;

			if (item != null && p.casItem(item, null)) {
				// Successful CAS is the linearization point
				// for item to be removed from this queue.
				if (p != h) // hop two nodes at a time
					updateHead(h, ((q = p.next) != null) ? q : p);
				return item;
			}
			else if ((q = p.next) == null) {
				updateHead(h, p);
				return null;
			}
			else if (p == q)
				continue restartFromHead;
			else
				p = q;
		}
	}
}
```

