**Java线程同步**

[TOC]

>线程安全就是防止某个对象或者值在多个线程中被修改而导致的数据不一致问题，因此我们就需要通过同步机制保证在同一时刻只有一个线程能够访问到该对象或数据，修改数据完毕之后，再将最新数据同步到主存中，使得其他线程都能够得到这个最新数据。下面我们就来了解Java一些基本的同步机制。

# volatile关键字

Java提供了一种稍弱的同步机制即volatile变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为volatile类型后，编译器与运行时都会注意到这个变量是共享的。然而，在访问volatile变量时不会执行加锁操作，因此也就不会使线程阻塞，因此volatile变量是一种比synchronized关键字更轻量级的同步机制。

volatile变量对所有的线程都是可见的，对volatile变量所有的写操作都能立即反应到其他线程之中，即volatile变量在各个线程中是一致的。

有一种情况需要注意：volatile的语义不能确保递增（count++）的原子性，除非你能确保只有一个线程对变量执行写操作。

```Java
public class VolatileTest{
    
    public static volatile int  i;

    public static void increase(){
        i++;
    }
}

查看字节码: javap -c -l VolatileTest.class

public class VolatileTest {
  public static volatile int i;
 
  public VolatileTest();
    Code:
       0: aload_0       
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return        
    LineNumberTable:
      line 1: 0
 
  public static void increase();
    Code:
       0: getstatic     #2                  // Field i:I, 把i的值取到了操作栈顶,volatile保证了i值此时是正确的. 
       3: iconst_1      
       4: iadd                              // increase,但其他线程此时可能已经把i值加大了好多
       5: putstatic     #2                  // Field i:I ,把这个已经out of date的i值同步回主内存中,i值被破坏了.
       8: return        
    LineNumberTable:
      line 6: 0
      line 7: 8
}
```
>加锁机制即可以确保原子性又可以确保可见性，而volatile变量只能确保可见性。

# 内置锁-synchronized

Java中最常用的同步机制就是synchronized关键字，它是一种基于语言的粗略锁，能够作用于对象、函数、Class。每个对象都只有一个锁，谁能够拿到这个锁谁就得到了访问权限。当synchronized作用于函数时，实际上锁的也是对象，锁定的对象是该函数所在类的对象。而synchronized作用于Class时则锁的是这个Class类，并非某个具体对象。

## synchronized同步方法和同步块

```Java
public class SynchronizedDemo {
	
	
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		final Test test = new Test();

		new Thread(new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				test.syncMethod(Thread.currentThread());
				
			}
		}).start();
		
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				test.syncMethod(Thread.currentThread());
			}
		}).start();
		
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				test.asyncMethod(Thread.currentThread());
			}
		}).start();
		
	}
}

class Test {
	
	public synchronized void syncMethod(Thread thread) {
		for(int i = 0;i < 3;i++) {
			System.out.println(thread.getName());
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	public void asyncMethod(Thread thread) {
		
		synchronized (this) {
			
			for(int i = 0;i < 3;i++) {
				System.out.println(thread.getName()+2);
			}
		}
	}
}

syncMethod和asyncMethod代码块都加锁时结果:

Thread-0
Thread-0
Thread-0
Thread-1
Thread-1
Thread-1
Thread-2
Thread-2
Thread-2  #多个线程不能同时访问同一个对象中的synchronized锁的方法或代码块

syncMethod加锁和asyncMethod代码块不加锁时结果:

class Test {
	
	public synchronized void syncMethod(Thread thread) {
		for(int i = 0;i < 3;i++) {
			System.out.println(thread.getName());
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	public void asyncMethod(Thread thread) {
		
		synchronized (this) {
			
			for(int i = 0;i < 3;i++) {
				System.out.println(thread.getName());
			}
		}
	}
}

Thread-0
Thread-22
Thread-22
Thread-22
Thread-0
Thread-0
Thread-1
Thread-1
Thread-1  #其他线程可以访问同一个对象的非同步方法或代码块

syncMethod不加锁和asyncMethod代码块不加锁时结果:

class Test {
	
	public void syncMethod(Thread thread) {
		for(int i = 0;i < 3;i++) {
			System.out.println(thread.getName());
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	public void asyncMethod(Thread thread) {
		
			for(int i = 0;i < 3;i++) {
				System.out.println(thread.getName()+2);
			}
	}
}

Thread-0
Thread-1
Thread-22
Thread-22
Thread-22
Thread-0
Thread-1
Thread-1
Thread-0

```
synchronized同步方法和同步块锁定的是引用对象，synchronized作用于引用对象是防止其他线程访问同一个对象的synchronized代码块或方法，但可以访问其他非同步代码块或方法。


## synchronized同步Class对象和静态方法

```Java
public class SynchronizedDemo {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		final Test test = new Test();

		new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				Test.syncStaticMethod(Thread.currentThread());

			}
		}).start();

		new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				Test.syncStaticMethod(Thread.currentThread());
			}
		}).start();

		new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				Test.asyncStaticMethod(Thread.currentThread());
			}
		}).start();

	}
}

class Test {

	public synchronized static void syncStaticMethod(Thread thread) {

		for (int i = 0; i < 3; i++) {
			System.out.println(thread.getName());
			try {
				Thread.sleep(50);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	public static void asyncStaticMethod(Thread thread) {

		synchronized (Test.class) {
			for (int i = 0; i < 3; i++) {
				System.out.println(thread.getName() + 22);
				try {
					Thread.sleep(50);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}

	}
}
syncStaticMethod和asyncStaticMethod代码块都加锁的结果:

Thread-0
Thread-0
Thread-0
Thread-222
Thread-222
Thread-222
Thread-1
Thread-1
Thread-1  ##多个线程不能同时访问添加了synchronized锁的代码块和方法。

syncStaticMethod加锁和asyncStaticMethod代码块不加锁的结果:

class Test {

	public synchronized static void syncStaticMethod(Thread thread) {

		for (int i = 0; i < 3; i++) {
			System.out.println(thread.getName());
			try {
				Thread.sleep(50);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	public static void asyncStaticMethod(Thread thread) {

			for (int i = 0; i < 3; i++) {
				System.out.println(thread.getName() + 22);
				try {
					Thread.sleep(50);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
	}
}

Thread-0
Thread-222
Thread-222
Thread-0
Thread-0
Thread-222
Thread-1
Thread-1
Thread-1 ##多个线程可以同时访问非同步的代码块和方法

syncStaticMethod加锁和asyncStaticMethod代码块都不加锁的结果:

class Test {

	public static void syncStaticMethod(Thread thread) {

		for (int i = 0; i < 3; i++) {
			System.out.println(thread.getName());
			try {
				Thread.sleep(50);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	public static void asyncStaticMethod(Thread thread) {

			for (int i = 0; i < 3; i++) {
				System.out.println(thread.getName() + 22);
				try {
					Thread.sleep(50);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
	}
}

Thread-0
Thread-1
Thread-222
Thread-1
Thread-0
Thread-222
Thread-1
Thread-0
Thread-222

```

synchronized同步Class对象和静态方法锁的是Class对象，它的作用是防止多个线程同时访问添加了synchronized锁的代码块和方法。

## 总结

-	当一个线程正在访问一个对象的synchronized方法，那么其他线程不能访问该对象的其他synchronized方法，因为一个对象只有一把锁，当一个线程获取了该对象的锁之后，其他线程无法获取该对象的锁，所有无法访问该对象的其他synchronized方法。

-	当一个线程正在访问一个对象的synchronized方法，那么其他线程能访问该对象的非synchronized方法。因为非synchronized方法不需要获取该对象的锁。

-	如果一个线程A需要访问对象object1的synchronized方法fun1，另外一个线程B需要访问对象object2的synchronized方法fun1，即使object1和object2是同一类型，也不会产生线程安全问题，因为他们访问的是不同的对象，所以不存在互斥问题。

-	如果一个线程执行一个对象的非static synchronized方法，另一个线程执行这个对象所属类的static synchronized方法，此时不会发生互斥现象，因为访问static synchronized方法占用的是类锁，而访问非static synchronized方法占用的是对象锁，所以不存在互斥现象。

需要注意的是：对于synchronized方法或者synchronized代码块，当出现异常时，JVM会自动释放当前线程占用的锁，因此不会由于异常导致出现死锁现象。

# 显示锁-ReentrantLock与Condition

## ReentrantLock

在JDk 5.0之前，协调共享对象的访问时，只有synchronized和volatile。Java 6.0增加了一种新的机制：ReentrantLock。显示锁ReentrantLock和内置锁synchronized相比，实现了相同的语义，但是具有更高的灵活性。

内置锁synchronized的获取和释放都在同一个代码块中，而显示锁ReentrantLock则可以将锁的获得和释放分开。同时显示锁可以提供轮训锁和定时锁，同时可以提供公平锁或者非公平锁。

ReentrantLock的基本操作如下：

| 函   数 | 作   用 |
|--------|--------|
|   lock()     |    			获取锁    |
|   tryLock()     |   			尝试获取锁     |
|   tryLock(timeout,Timeunit unit)     |   在指定时间内尝试获取锁     |
|   unLock()     |     释放锁   |
|   newCondition     |   获取锁的Condition     |

使用ReentrantLock的一般是lock、tryLock与unLock成对出现，需要注意的是，千万不要忘记调用unLock来释放锁，否则会引发死锁等问题。

ReentrantLock的常用形式如下所示：

```Java
Lock lock = new ReentrantLock();

public void run() {

	lock.lock();

	try {

		//执行任务

	} finally {

		lock.unlock();
	}
}
```
>需要注意的是，lock必须在finally块中释放，否则，如果受保护的代码块抛出异常，锁就有可能永远得不到释放。而使用synchronized同步，JVM将确保锁会获得自动释放，这也是Lock没有完全替代掉synchronized的原因。

当JVM用synchronized管理锁定请求和释放行为时，JVM在生成线程转储时能够包括锁定信息，这些对调式有非常大的价值，因为它们能标识死锁和其他异常行为的来源。Lock类只是普通的类，JVM不知道具体哪个线程拥有Lock对象。

## Condition

在ReentrantLock类中有一个重要的函数newCondition()，该函数用于获取lock上的一个条件，也就是说Condition是和Lock绑定的。Condition用于实现线程间的通信，它是为了解决Object.wait()、notify()、notifyAll()难以使用的问题。

Condition的基本操作如下所示：

| 方  法 | 作  用 |
|--------|--------|
|   await()     |    线程等待    |
|   await(int time,TimeUnit unit)     |  线程等待特定的时间，超过时间则为超时      |
|   signal()     |   随机唤醒某个等待线程     |
|   signalAll()     |  唤醒所有等待中的线程      |

## 综合应用

下面通过ReentrantLock和Condition类实现一个简单的阻塞队列。如果调用take方法时集合中没有数据，那么调用线程阻塞；如果调用put方法时，集合数据已满则调用线程阻塞。但是这两个阻塞条件是不同的，分别为notFull和notEmpty。MyArrayBlockingQueue的实现代码如下：

```Java
public class MyArrayBlockingQueue<T> {

	// 数据数组
	private final T[] items;
	// 锁
	private final Lock mLock = new ReentrantLock();
	// 数组满的条件
	private Condition notFull = mLock.newCondition();
	// 数组空的条件
	private Condition notEmpty = mLock.newCondition();

	// 头部
	private int head;
	// 尾部
	private int tail;
	// 数据数量
	private int count;

	public MyArrayBlockingQueue(int maxSize) {
		// TODO Auto-generated constructor stub
		items = (T[]) new Object[maxSize];
	}

	public MyArrayBlockingQueue() {
		// TODO Auto-generated constructor stub
		this(10);
	}

	public void put(T t) {

		mLock.lock();

		try {

			// 如果数据已满,等待
			while (count == getCapacity()) {
				System.out.println("数据已满，请等待");
				notFull.await();
			}

			System.out.println("存入数据");

			items[tail] = t;
			if (++tail == getCapacity()) {
				tail = 0;
			}

			++count;
			// 唤醒等待数据的线程
			notEmpty.signalAll();

		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			mLock.unlock();
		}
	}

	public T take() {

		mLock.lock();

		try {

			// 如果数组数据为空，则阻塞
			while (count == 0) {
				System.out.println("还没有数据，等待");
				notEmpty.await();
			}

			System.out.println("取出数据");

			T t = items[head];
			items[head] = null;

			if (++head == getCapacity()) {
				head = 0;
			}

			--count;
			// 唤醒添加数据的线程
			notFull.signalAll();
			return t;

		} catch (InterruptedException e) {
			// TODO: handle exception
		} finally {
			mLock.unlock();
		}

		return null;
	}

	public int getCapacity() {
		return items.length;
	}

	public int size() {
		mLock.lock();
		try {
			return count;
		} finally {
			mLock.unlock();
		}
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		final MyArrayBlockingQueue<String> mQueue = new MyArrayBlockingQueue<>(
				5);

		new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				while (true) {

					for(int i  = 0;i < 3;i++)
					mQueue.put("just");
					try {
						Thread.sleep(50);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}

				}

			}
		}).start();

		new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				while (true) {
					
					mQueue.take();

				}
			}
		}).start();

	}

}

结果打印
存入数据
存入数据
存入数据
取出数据
取出数据
取出数据
还没有数据，等待
存入数据
存入数据
存入数据
取出数据
取出数据
取出数据
还没有数据，等待

```

# 信号量-Semaphore

Semaphore是一个计数信号量，它的本质是一个“共享锁”。信号量维护一个信号许可集合，线程可以通过调用acquire()来获取信号量的许可。当信号量有可用的许可时，线程能获取该许可；否则线程必须等到，直到有可用的许可为止。线程可以通过release()来释放它所持有的信号量许可。

Semaphore实现的功能类似食堂窗口。例如，食堂只有3个销售窗口，要吃饭的有5个人，那么同时只有3个人买饭菜，每个人占用一个窗口，另外2人只能等待。当前3个人有人离开之后，后续的人才可以占用窗口进行购买。这里的窗口就是我们所说的许可集，这里为3.一个人占用窗口时相当于他调用acquire()获取了许可，当他离开时也就等于调用release()释放了许可，这样后续的人才可以得到许可。下面看看具体的示例：

```Java
public class SemaphoreTest {
	

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ExecutorService service = Executors.newFixedThreadPool(3);
		final Semaphore semaphore = new Semaphore(3);
		
		for(int i = 0;i < 5;i++) {
			
			service.submit(new Runnable() {
				
				@Override
				public void run() {
					// TODO Auto-generated method stub
					try {
						semaphore.acquire();
						System.out.println("剩余许可: " + semaphore.availablePermits());
						Thread.sleep(2000);
						semaphore.release();
						
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					
				}
			});
		}

	}

}

结果打印：

剩余许可: 0
剩余许可: 0
剩余许可: 0

剩余许可: 2
剩余许可: 1

```
上述结果中：前三行是立刻输出的，后两行是等待2秒之后才输出。原因是，信号量的许可集是3个，而消费线程是5个。前3个线程获取了许可之后，信号量的许可就为0。此时后面的线程再调用acquire()就会阻塞，直到前3个线程执行完之后，释放了许可（不需要同时释放许可）后两个线程才能获取许可并且继续执行。

# 循环栅栏-CyclicBarrier

CyclicBarrier是一个同步辅助类，允许一组线程互相等待，直到达到某个公共屏障点。因为该barrier在释放等待线程后可以重用，所有称为循环的barrier。

下面看看示例：

```Java
public class CyclicBarrierTest {
	
	private static final int SIZE = 5;
	private static CyclicBarrier mCyclicBarrier;

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub

		mCyclicBarrier = new CyclicBarrier(SIZE, new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				System.out.println("--满足条件执行特定操作，参与者： "+ mCyclicBarrier.getParties());
			}
		});
		
		
		for(int i = 0;i < SIZE;i++) {
			new WorkerThread().start();
		}
		
		
	}
	
	static class WorkerThread extends Thread {
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			try {
				System.out.println(Thread.currentThread().getName() + "等待CyclicBarrier");
				//将mCyclicBarrier的参与者数量加1
				mCyclicBarrier.await();
				//mCyclicBarrier的参与者数量加5时，才继续往后执行
				System.out.println(Thread.currentThread().getName()+"继续执行");
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (BrokenBarrierException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

}

结果打印：

Thread-1等待CyclicBarrier
Thread-0等待CyclicBarrier
Thread-2等待CyclicBarrier
Thread-3等待CyclicBarrier
Thread-4等待CyclicBarrier
--满足条件执行特定操作，参与者： 5
Thread-4继续执行
Thread-3继续执行
Thread-2继续执行
Thread-0继续执行
Thread-1继续执行

```

从结果可以看出，只有当有5个线程调用了mCyclicBarrier.await()方法后，后续的任务才会继续执行。上述例子中的5个WorkThread就位之后首先会执行一个Runnable，也就是CyclicBarrier构造函数的第二个参数，该参数也可以省略。执行该Runnable之后才会继续执行下面的任务。CyclicBarrier实际上相当于可以用于多个线程等待，直到某个条件被满足后开始继续执行后续的任务。对于该示例来说，这里的条件也就是有指定个数的线程调用了mCyclicBarrier.await()方法。

# 闭锁-CountDownLatch

CountDownLatch是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待，直到条件被满足。

示例如下：

```Java
public class CountDownLatchTest {
	
	private static final int LATCH_SIZE = 5;

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			
			CountDownLatch countDownLatch = new CountDownLatch(LATCH_SIZE);
			
			for(int i = 0;i < LATCH_SIZE;i++) {
				new WorkerThread(countDownLatch).start();
			}
	
			System.out.println("主线程等待");
			
			countDownLatch.await();
			
			System.out.println("主线程继续执行");
			
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	static class WorkerThread extends Thread {
		
		private CountDownLatch latch;
		
		public WorkerThread(CountDownLatch latch) {
			this.latch = latch;
		}
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			try {
				Thread.sleep(1000);
				System.out.println(Thread.currentThread().getName() + "执行操作");
				//将latch的数量减1
				latch.countDown();
				
			} catch (Exception e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

}

结果打印：

主线程等待
Thread-3执行操作
Thread-1执行操作
Thread-0执行操作
Thread-4执行操作
Thread-2执行操作
主线程继续执行

```

5个WorkThread对象在执行完操作之后会调用CountDownLatch的countDown()函数，当5个WorkThread全都调用了countDown()之后主线程就会被唤醒继续执行任务。

# CountDownLatch与CyclicBarrier区别

-	CountDownLatch的作用是允许1或者多个线程等待其他线程完成执行，而CyclicBarrier则是允许N个线程相互等待。

-	CountDownLatch的计数器无法被重置，CyclicBarrier的计数器可以被重置后使用。

