**Java多线程及线程池**

[TOC]

线程允许在同一个进程中同时存在多个程序控制流，即通过线程可以实现同时处理多个任务的功能。线程会共享进程范围内的资源，例如内存句柄和文件句柄，但每个线程都有各自的程序计数器、栈以及局部变量。

# 多线程的实现

## 实现方式

对于Java的多线程来说，我们学习的一般都是Thread和Runnable，通过我们使用如下代码启动一个新的线程：

```Java
private void startewThread() {

	new Thread(){

		@Override
		public void run() {

			// 耗时任务
		}

	}.start();
}
或者
private void startewThread1(){

	new Thread(new Runnable() {

		@Override
		public void run() {
			// 耗时任务

		}
	}).start();
}

```
第一种是覆写了Thread类中的run方法执行任务；第二种是实现Runnable接口中的run方法执行任务。

那么Thread和Runnable是什么关系呢？

## Thread和Runnable的关系

实际上Thread也是一个Runnable，它实现了Runnable接口，在Thread类中有一个Runnable类型的target字段，代表要被执行在这个子线程的任务。相关代码如下：

```Java
public class Thread implements Runnable {

	//要执行的目标任务
    private Runnable target;
	//线程所属的线程组
    private ThreadGroup group;
	
	public Thread() {
		init(null, null, "Thread-" + nextThreadNum(), 0);
    }
	
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }

    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name.toCharArray();

        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /group为null则获取当前线程的线程组
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }

        g.addUnstarted();

        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
		//设置target
        this.target = target;
        setPriority(priority);
        if (parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        tid = nextThreadID();
    }
	
   public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
			//调用native函数启动线程
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
	
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
}

```
实际上最终被线程执行的任务是Runnable，而非Thread。Thread 只是对Runnable的包装，并且通过一些状态对Thread进行管理和调度。Runnable的声明如下：

```Java
public interface Runnable {

    public void run();

}
```
当启动一个线程时，如果Thread的target不为空，则会在子线程中执行这个target的run方法，否则虚拟机就会执行该线程自身的run方法。

# 线程的wait、sleep、join和yield

先通过下面的表格来了解他们的区别：

| 函数名 | 作用 |
|--------|--------|
|    wait    |当一个线程执行到wait()方法时，它就进入到一个和该对象相关的等待池中，同时释放了对象的锁，使得其他线程可以访问。用户可以使用notify、notifyAll或指定睡眠时间来唤醒当前等待池中的线程。  注意：wait、notify、notifyAll方法必须放在synchronized block中，否则则会抛出异常。        |
|    sleep   |该函数时Thread的静态函数，作用是使调用线程进入睡眠状态。因为sleep()Thread的静态函数，因此它不能改变对象的锁。所以当一个synchronized块中调用sleep方法时，线程虽然休眠了，但是对象的锁并没有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁）        |
|    join    |等待目标线程执行完成之后再继续执行        |
|    yield   |线程礼让。目标线程由运行状态转换为就绪状态，也就是让出执行权限，让其他线程得以优先执行，但其他线程能否优先执行时未知的。|

## wait()

下面来看看wait、notify、notifyAll的使用：

```Java

public class WaitDemo {

	private static Object lockObject = new Object();

	private static void waitAndNotifAll() {

		System.out.println("主线程运行");

		//创建并启动子线程
		Thread thread = new WaitThread();
		thread.start();

		long startTime = System.currentTimeMillis();

		try {
			//必须在synchronized块中
			synchronized (lockObject) {
				System.out.println("主线程等待");
				lockObject.wait();

			}
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		//被唤醒后继续执行
		long endTime = System.currentTimeMillis() - startTime;
		
		System.out.println("主线程继续--->等待耗时： " + endTime + "ms");

	}

	private static class WaitThread extends Thread {

		@Override
		public void run() {
			// TODO Auto-generated method stub
			synchronized (lockObject) {
				try {
					Thread.sleep(3000);
					//唤醒正在等待中的线程
					lockObject.notifyAll();
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}

			}
		}
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		waitAndNotifAll();

	}

}

运行结果：

主线程运行
主线程等待
...
...

主线程继续--->等待耗时： 3001ms

```
wait、notify机制通常用于等待机制的实现，当条件未满足时调用wait进入等待状态，一旦条件满足，调用notify或notifyAll唤醒等待的线程继续执行。

## join()

join函数的原始解释为“Block the cuurent thread(Thread.currentThread()) untile the receiver finishes its execution and dies。意思就是阻塞当前调用join函数的任务所在的线程，直到该任务执行完成后再继续执行所在线程的任务。下面我们来看看一个具体是实例：

```Java
public class JoinDemo {
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub

		joinDemo();
	}

	static void joinDemo() {

		System.out.println("主线程开始执行");
		
		Worker worker1 = new Worker("worker-1");
		Worker worker2 = new Worker("worker-2");

		worker1.start();

		System.out.println("启动线程1--执行完毕");

		try {
			//等待worker1任务执行完成
			worker1.join();
			
			System.out.println("启动线程2--执行完毕");
			
			worker2.start();
			
			//等待worker2任务执行完成
			worker2.join();
			
			
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		System.out.println("主线程继续执行");
		
		System.out.println("主线程执行完毕");
	}

	static class Worker extends Thread {

		public Worker(String name) {
			super(name);
		}

		@Override
		public void run() {
			// TODO Auto-generated method stub
			try {
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

			System.out.println("Work in " + getName());
		}
	}

}

结果打印：

主线程开始执行
启动线程1--执行完毕
Work in worker-1
启动线程2--执行完毕
Work in worker-2
主线程继续执行
主线程执行完毕

```
上述代码的逻辑是主线程开始执行、启动线程1、等待线程1执行完毕、启动线程2、等待线程2执行完毕、继续执行主线程任务。

## yield()

```
public static native void yield();
```

yield函数的官方解释是"Causes the calling Thread to yiled execution time to another Thread that is ready to run",意思是使调用该函数的线程让出执行时间给其他已就绪状态的线程。

线程的执行是有时间片的，每个线程轮流占用CPU固定的时间，执行周期到了之后就让出执行权给其他线程，而yield函数的功能就是主动让出线程的执行权给其他线程，其他线程能否得到优先权就得看各个线程的状态了。下面来看看一个具体的示例：

```Java
public class YieldDemo {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		YieldThread t1 = new YieldThread("thread-1");
		YieldThread t2 = new YieldThread("thread-2");
		t1.start();
		t2.start();

	}

	
	static class YieldThread extends Thread {
		
		public YieldThread(String name) {
			// TODO Auto-generated constructor stub
			super(name);
		}
		
		@Override
		public synchronized void run() {
			// TODO Auto-generated method stub
			for(int i = 0; i < 5;i++) {
				System.out.println(this.getName() + " ; " + "线程优先级为： " + this.getPriority()+ "--->" + i);
				
				//当i为2时 调用当前线程yield函数
				if (i== 2) {
					Thread.yield();
				}
			}
		}
	}
}

打印结果：

thread-1 ; 线程优先级为： 5--->0
thread-2 ; 线程优先级为： 5--->0
thread-2 ; 线程优先级为： 5--->1
thread-2 ; 线程优先级为： 5--->2
thread-1 ; 线程优先级为： 5--->1
thread-1 ; 线程优先级为： 5--->2
thread-2 ; 线程优先级为： 5--->3
thread-2 ; 线程优先级为： 5--->4
thread-1 ; 线程优先级为： 5--->3
thread-1 ; 线程优先级为： 5--->4

```
从结果可知，thread-2首先执行到i的值为2，此时让出执行权，thread-1得到执行权运行到i的值为2时让出执行权，thread-2得到执行权执行任务结束，然后thread-1再继续执行任务。

注意：yield仅在一个时间片内有效。

## Callable、Future和FutureTask

除了Runnable之外，Java还有Callable、Future和FutureTask这几个与多线程相关的概念，与Runnable不同的是这个类型都只能运用到线程池中，而Runnable既能运用在Thread中，还能运用在线程池中。

### Callable

Callable与Runnable的功能大致相似不同的是Callable是一个泛型接口，它有一个泛型参数V，该接口中有一个返回值（类型为V）的Call函数，而Runnable中的run方法不能将结果返回至调用者。Callable的声明如下：

```Java
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```

### Future

Future为线程池制定了一个可管理的任务标准。它提供了对Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果、设置结果操作，分别对应cancel、isDone、get、set函数。get方法会阻塞，直到任务返回结果。Future的声明如下：

```Java
public interface Future<V> {

	//取消任务
	boolean cancel(boolean mayInterruptIfRunning);

	//判断任务是否已经取消
	boolean isCancelled();

	//判断任务是否已经完成
	boolean isDone();

	//获取结果，如果任务未完成则等待，直到完成，因此该函数会阻塞
	V get() throws InterruptedException, ExecutionException;

	//获取结果，如果未完成则等待，直到返回结果或timeout，该函数会阻塞
    V get(long timeout, TimeUnit unit)
    throws InterruptedException, ExecutionException, TimeoutException;
}

```
### FutureTask

Future只是定义了一些规范的接口，而FutureTask则是它的实现类。FutureTask实现了`RunnableFuture<V>`，而RunnableFuture实现了Runnable又实现了Future<V>这两个接口，因此FutureTask同时具备他们的功能。FutureTask的代码如下：

```Java
public class FutureTask<V> implements RunnableFuture<V> {
	.....
}


```

RunnableFuture<V>类的定义

```Java

public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
```
FutureTask像Thread那样包装Runnable那样对Runnable和`Callable<V>`进行包装，Runnable与Callable由构造函数注入

```Java

public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}

public FutureTask(Runnable runnable, V result) {
    this.callable = Executors.callable(runnable, result);
    this.state = NEW;       // ensure visibility of callable
}

```

从上述代码可以看出，如果注入的是Runnable则会被Executors.callable()函数转换为Callable类型，即FutureTask最终都是执行Callable类型的任务，该转换函数如下：

```Java
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

/**
* Runnable适配器，将Runnable转换为Callable
*/
static final class RunnableAdapter<T> implements Callable<T> {
    final Runnable task;
    final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}

```
由于FutureTask实现了Runnable，因此它既可以通过Thread包装来执行，也可以提交给ExecuteService来执行，并且还可以通过get()函数来获取执行结果，该函数会阻塞，直到结果返回。因此，FutureTask既是Future、Runnable，又是包装了Callable（Runnable最终也会被转换为Callable），它是这两者的合体。

下面示例演示Runnable、Callable、FutureTask的运用，代码如下：

```Java
public class FutureTaskDemo {

	//线程池
	static ExecutorService mExecutor = Executors.newSingleThreadExecutor();
	
	/**
	 * 向线程池提交Runnable对象
	 */
	private static void taskRunnable() {
		
		//无返回值
		Future<?> future = mExecutor.submit(new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				fibc(20);
				
			}
		});

		System.out.println("taskRunnable: " + future.get());
	}
	
	/**
	 * 向线程池提交Callable对象
	 * @throws ExecutionException 
	 * @throws InterruptedException 
	 */	
	private static void taskCallable() throws InterruptedException, ExecutionException {
		
		Future<Integer> future = mExecutor.submit(new Callable<Integer>() {

			@Override
			public Integer call() throws Exception {
				// TODO Auto-generated method stub
				return fibc(20);
			}
		});
		
		//返回值
		Integer result = future.get();
		
		if (result != null) {
			System.out.println("taskCallable: " + result);
		}
	}

	/**
	 * 向线程池提交FutureTask对象
	 * @throws ExecutionException 
	 * @throws InterruptedException 
	 */
	private static void taskFutureTask() throws InterruptedException, ExecutionException {
		
		FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {

			@Override
			public Integer call() throws Exception {
				// TODO Auto-generated method stub
				return fibc(20);
			}
		});
		
		mExecutor.submit(futureTask);
		
		Integer result = futureTask.get();
		
		if (result != null) {
			System.out.println("taskFutureTask: " + result);
		}
		
	}
	
	/**
	 * Thread包装FutureTask
	 * @throws InterruptedException
	 * @throws ExecutionException
	 */
	private static void taskThread() throws InterruptedException, ExecutionException {
		
		FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {

			@Override
			public Integer call() throws Exception {
				// TODO Auto-generated method stub
				return fibc(20);
			}
		});
		
		new Thread(futureTask).start();
		
		Integer result = futureTask.get();
		
		if (result != null) {
			System.out.println("taskThread: " + result);
		}
		
	}
	
	/**
	 * 斐波那契数列
	 * @param num
	 * @return
	 */
	private static int fibc(int num) {
		
		if (num == 0) {
			return 0;
		}
		
		if (num == 1) {
			return 1;
		}
		
		return fibc(num - 1) + fibc(num - 2);
	}
	
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			taskRunnable();
			taskCallable();
			taskFutureTask();
			taskThread();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 

	}

}

打印结果：

taskRunnable: null
taskCallable: 6765
taskFutureTask: 6765
taskThread: 6765

```
# 线程池

当我们需要频繁地创建多个线程进行耗时操作时，每次都通过new Thread实现并不是一种好的方式，每次new Thread新建销毁对象性能较差，线程缺乏统一的管理，可能会无限制地创建新的线程，线程之间相互竞争从而占用过多系统资源导致死锁，并且缺乏定期执行、定时执行、线程中断等功能。

Java提供了4中线程池，它能够有效地管理、调度线程，避免过多的资源消耗，它强大到几乎不需要开发人员自定义的程序。它的优点如下：

-	重用存在的线程，减少对象创建、销毁的开销；
-	可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞；
-	提供定时执行、定期执行、单线程、并发数控制等功能；

线程池的原理就是会创建创建多个线程并且对这些线程进行管理，提交给线程的任务 会被线程池指派给其中的线程执行，提供线程池的统一调度、管理。使得多线程的使用更简单、高效。

线程池都实现了ExecutorService接口，该接口定义了线程池需要实现的接口，如submit、execute、shutdown等。它的实现有ThreadPoolExecutor和ScheduledPoolExecutor，ThreadPoolExecutor是运行最多的线程池实现，ScheduledPoolExecutor则用于执行周期性任务。

## 启动指定数量的线程-ThreadPoolExecutor

ThreadPoolExecutor的功能是启动指定数量的线程以及将任务添加到一个队列中，并且将任务分发给空闲的线程。

ExecutorService的生命周期包括3中状态：运行、关闭、终止，创建后进入运行状态，调用shutdown()方法时便进入了关闭状态，此时ExecutorService不再接受新的任务，但它继续执行完已经提交的任务，当所有已经提交的任务都执行完后，就变成终止状态。

ThreadPoolExecutor的构造函数如下：

```Java
public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue,
						  ThreadFactory threadFactory,
						  RejectedExecutionHandler handler)
```
下面对参数进行详细说明:

| 参 数 名 | 作  用 |
|--------|--------|
|   corePoolSize     |       线程池中所保存的核心线程数。 |
|   maximumPoolSize    |     线程池所容纳的最大线程数，当活动线程达到这个数值后，后续的任务将会被阻塞   |
|   keepAliveTime     |      非核心线程闲置时的超时时间，超出这个时长，非核心线程就会被回收  |
|   unit     |     			 用于指定keepAliveTime参数的时间单位，有毫秒、秒、分钟等   |
|   workQueue     |    		 线程池中的任务队列，如果线程池的线程数量已经达到核心线程数并且当前所有线程都处于活动状态时，则将新任务放到此队列中等待执行    |
|   threadFactory     |      线程工厂，为线程池提供创建新线程的功能，通常不需要设置  |
|   handler     |        	 拒绝策略，当线程池与workQueue队列都满了的情况下，对新任务采取的处理策略|

[线程池参数也可以参考这篇文章http://liuguoquan727.github.io/2016/04/25/Android%E7%9A%84%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/:](http://liuguoquan727.github.io/2016/04/25/Android%E7%9A%84%E7%BA%BF%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E6%B1%A0/)

其中workQueue有下列几个常用的实现：

-	ArrayBlockingQueue

基于数组结构的有界队列，此队列按FIFO原则对任务进行排序。如果队列满了还有任务进来，则调用拒绝策略

-	LinkedBlockingQueue

基于链表结构的无界队列，此队列按FIFO原则对任务进行排序。因为它是无界的，所以才有此队列后线程池将忽略handler参数。

-	SynchronousQueue

直接将任务提交给线程而不是将它加入到队列，实际上该队列是空的。每个插入的操作必须等到另一个调用移除的操作，如果新任务来了线程池没有任何可用线程处理的话，则调用拒绝策略。

-	PriorityBlockingQueue

具有优先级的队列的有界队列，可用自定义优先级，默认是按自然排序的。

此外，当线程池与workQueue队列都满了的情况下，对新加任务采取的处理策略也有几个默认实现：

-	AbortPolicy

拒绝任务，抛出RejectedExecutionException异常，线程池默认策略

-	CallerRunsPolicy

拒绝新任务加入，如果该线程池还没有被关闭，那么将这个新任务执行在调用线程中

-	DiscardOldestPolicy

如果执行程序还没有关闭，则将位于工作队列头部的任务删除，然后重试执行程序（如果再次失败，则重复此过程）

-	DiscardPolicy

加不进的任务都被抛弃了，同时没有异常抛出

### newFixedThreadPool

对应Android平台来说，最常使用的就是通过Executors.newFixedThreadPool(int size)函数来启动固定数量的线程池，代码如下

```Java
public class ExectorsDemo {

	private static final int MAX = 10;
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		try {
			fixedThreadPool(MAX);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ExecutionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

	}
	
	private static void fixedThreadPool(int size) throws InterruptedException, ExecutionException {
		
		ExecutorService service = Executors.newFixedThreadPool(size);
		
		for(int i = 0;i < MAX;i++) {
			
			//提交任务
			Future<Integer> task = service.submit(new Callable<Integer>() {

				@Override
				public Integer call() throws Exception {
					// TODO Auto-generated method stub
					System.out.println("执行线程: " + Thread.currentThread().getName());
					return fibc(20);
				}
			});
			
			//获取结果
			System.out.println("第"+i+"次计算结果: " + task.get());
		}
	}
	
	/**
	 * 斐波那契数列
	 * @param num
	 * @return
	 */
	private static int fibc(int num) {
		
		if (num == 0) {
			return 0;
		}
		
		if (num == 1) {
			return 1;
		}
		
		return fibc(num - 1) + fibc(num - 2);
	}

}

结果打印：

执行线程: pool-1-thread-1
第0次计算结果: 6765
执行线程: pool-1-thread-2
第1次计算结果: 6765
执行线程: pool-1-thread-3
第2次计算结果: 6765
执行线程: pool-1-thread-1
第3次计算结果: 6765
执行线程: pool-1-thread-2
第4次计算结果: 6765
执行线程: pool-1-thread-3
第5次计算结果: 6765
执行线程: pool-1-thread-1
第6次计算结果: 6765
执行线程: pool-1-thread-2
第7次计算结果: 6765
执行线程: pool-1-thread-3
第8次计算结果: 6765
执行线程: pool-1-thread-1
第9次计算结果: 6765

```
在上述例子中，我们启动了含有3个线程的线程池，调用的是Executors的newFixedThreadPool函数，该函数的实现为

```Java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
可知它的corePoolSize和MaxnumPoolSize值都是nThreads，并且设置keepAliveTime为0毫秒，最后设置无界任务队列，这样该线程池中就含有固定个数的线程，并且能够容纳无数个任务。

### newCacheThreadPool

有时可能需要任务尽可能快地被执行，这就需要线程池中的线程足够多也就是说此时需要拿空间来换时间，线程越多占用的内存消耗就越大。因此，我们可能需要一种场景，如果来了一个新的任务，并且没有空闲线程可用，此时必须马上创建一个线程来立即执行任务。我们可以通过Executors的newCacheThreadPool函数来实现。

```Java
private static void newCacheThreadPool() throws InterruptedException, ExecutionException {

	ExecutorService service = Executors.newCachedThreadPool();

	for(int i = 0;i < MAX;i++) {

		//提交任务
		service.submit(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				System.out.println("执行线程: " + Thread.currentThread().getName() + ",结果:" + fibc(20));
			}
		});

	}
}
结果打印

执行线程: pool-1-thread-1,结果:6765
执行线程: pool-1-thread-2,结果:6765
执行线程: pool-1-thread-4,结果:6765
执行线程: pool-1-thread-6,结果:6765
执行线程: pool-1-thread-8,结果:6765
执行线程: pool-1-thread-5,结果:6765
执行线程: pool-1-thread-3,结果:6765
执行线程: pool-1-thread-7,结果:6765
执行线程: pool-1-thread-10,结果:6765
执行线程: pool-1-thread-9,结果:6765

```
从上述结果可以看出，为了保证吞吐量，该线程池为每个任务都创建了一个线程，当然这是在没有线程空闲的情况下创建的新的线程。假设执行前5个任务时都创建了一个线程，执行到底6个任务时刚好前面的第一个任务执行完毕，此时线程1空闲，那么第六个任务就会被执行在第一个线程中，而不是重新创建。


## 执行周期性任务的线程-ScheduledPoolExecutor

通过Executors的newScheduledThreadPool函数即可创建定时执行任务的线程池。

```Java
private static void newScheduledThreadPool() throws InterruptedException,
		ExecutionException {

	ScheduledExecutorService service = Executors.newScheduledThreadPool(4);

	// 参数2为第一次延迟的时间，参数2为执行周期
	service.scheduleAtFixedRate((new Runnable() {

		@Override
		public void run() {
			// TODO Auto-generated method stub
			System.out.println("执行线程: " + Thread.currentThread().getName()
					+ ",定时计算 1结果:" + fibc(20));
		}
	}), 1, 2, TimeUnit.SECONDS);

	// 参数2为第一次延迟的时间，参数2为执行周期
	service.scheduleAtFixedRate((new Runnable() {

		@Override
		public void run() {
			// TODO Auto-generated method stub
			System.out.println("执行线程: " + Thread.currentThread().getName()
					+ ",定时计算2结果:" + fibc(30));
		}
	}), 1, 2, TimeUnit.SECONDS);

}

打印结果：

执行线程: pool-1-thread-1,定时计算 1结果:6765
执行线程: pool-1-thread-2,定时计算2结果:832040
执行线程: pool-1-thread-1,定时计算 1结果:6765
执行线程: pool-1-thread-3,定时计算2结果:832040
执行线程: pool-1-thread-1,定时计算 1结果:6765
执行线程: pool-1-thread-4,定时计算2结果:832040
执行线程: pool-1-thread-1,定时计算 1结果:6765
执行线程: pool-1-thread-3,定时计算2结果:832040
执行线程: pool-1-thread-2,定时计算 1结果:6765
执行线程: pool-1-thread-4,定时计算2结果:832040

```
该线程池有4个线程，我们指定了两个定时任务，因此该线程池中有两个线程来定时执行任务，哪个线程空闲就调度哪个线程来执行任务。

# 同步集合

## 程序中的优化策略-CopyOnWrite

Copy-On-Write是一种用于程序设计中的优化策略，其基本思路是，从多个线程共享同一个列表，当某个线程想要修改这个列表的元素时，会把列表中的元素复制一份，然后进行修改，修改完成之后再将新的元素设置给这个列表，这是一种延时懒惰策略。这样做的好处是我们可以对CopyOnWrite容器进行并发的读而不需要加锁，因为当前容器不会添加、移除任何元素。所有CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。从JDK1.5起Java并发包提供了两个使用CopyOnWrite机制实现的并发容器，它们是CopyOnWriteArrayList和CopyOnWriteSet。

通过这种写时拷贝的原理可以将读、写分离，使并发场景下对列表的操作效率得到提高，但它的缺点是，在添加、移除元素时占用的内存空间翻了一倍，因此，这是以空间换时间的策略。

## 提高并发效率-ConcurrentHasMap

HashTable使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用个图方法来获取元素，所以竞争越激烈效率越低。

HashTable在竞争激烈的并发环境下表现出效率低下的原因是因为所有访问HashTable的线程都必须竞争同一把锁。
假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHasMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。有些方法需要跨段，如size()和containsValue()，它们可能需要锁定整个表而不仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。

## 有效的方法-BlockingQueue

BlockingQueue的重要方法:

| 函 数 名 | 作   用 |
|--------|--------|
|   add(e)     |   把元素e添加到队列中，成功返回true，否则抛出异常|
|   offer(e)     |        	把元素e添加到队列中，成功返回true，否则返回false|
|   offer(e,time,unit)     |把元素e添加到队列中，成功返回true，否则在等待指定的时间之后继续尝试添加，如果失败则返回false        |
|   put(e)     |     把元素e添加到队列中，如果队列不能容纳，则调用此方法的线程被阻塞直到队列里面有空间再继续添加|
|   take()     |      取出队列中的首个元素，若队列为空，则线程进入等待直到队列中新的元素加入为止|
|   poll(time,unit)     |    取出并移除队列中的首个元素，如果在指定的时间内没有获取元素，则返回null    |
|   element()     |        	获取队首元素，如果队列为null，那么抛出NoSuchElementException异常
|   peek()     |        	获取队首元素，如果队列为空，那么返回null
|   remove()     |        	获取并移除队首元素，如果队列为空，那么抛出NoSuchElementException异常


BlockingQueue常用的实现有：

-	ArrayBlockingQueue
-	LinkedBlockingQueue
-	LinkedBlockingDequeue
-	ConcurrentLinkedQueue


