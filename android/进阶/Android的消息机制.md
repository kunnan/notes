**Android的消息机制**
[TOC]

# Android的消息机制概述

Android的消息机制主要是指Handler的运行机制，Handler的运行机制需要底层的MessageQueue和Looper的支撑。

MessageQueue的中文为消息队列，顾名思义，它的内部存储了一组消息，以队列的形式对外提供插入和删除的方法。虽然称为消息队列，但是它的**内部存储结构并不是真正的队列，而是采用单链表的数据结构来存储消息列表**。

Looper的中文翻译是循环，这里可以理解为消息循环。由于MessageQueue只是一个消息的存储单元，它不能去处理消息，而Looper会以无限循环的形式其查询是否有新的消息，如果有的话就处理消息，否则就一直等待。

Looper中还有一个特殊的概念，那就是ThreadLocal，ThreadLocal并不是线程，它的作用是可以在每个线程中存储数据。我们知道，Handler创建的时候会采用当前线程的Looper来构造消息循环系统，那么Handler内部如何获取到当前线程的Looper呢？这就要使用ThreadLocal了，ThreadLocal可以在不同的线程中互不干扰地存储并提供数据，通过ThreadLocal就可以轻松地获取每个线程的Looper。

需要注意的是，线程是默认没有Looper的，如果需要使用Handler就必须为线程创建Looper。而主线程，即UI线程，它就是ActivityThread，ActivityThread被创建时就会初始化Looper，这也是在主线程中默认可以使用Handler的原因。

# Android消息机制分析

Android消息机制主要是指Handler的运行机制以及Handler所附带的MessageQueue和Looper的工作过程，这三者实际上是一个整体。**Handler的作用**主要是将一个任务切换到某个指定的线程中去执行，那么Android为什么要提供这个功能呢？**这是因为Android规定UI只能在主线程中进行，如果在子线程中访问UI，那么程序就会抛出运行时异常。**ViewRootImp对UI操作做了验证，这个验证工作是由ViewRootImpl的checkThread方法来完成的，如下所示。

```Java
void checkThread() {

	if(mThread != Thread.currentThread()) {
		throw new CalledFromWrongThreadException("Only the original thread that created a view hiearachy can touch its view");
	}
}
```

Android建议我们不要在主线程中进行耗时的操作，否则会导致程序无法响应ANR。考虑一种情况，假如我们需要从服务器拉取一些信息并将其显示在UI上，这个时候必须在子线程中进行拉取工作，拉取完毕之后又不能在子线程中直接访问UI，如果没有Handler，那么我们确实没有办法将访问UI的工作切换到主线程中执行，因此，系统提供Handler的主要原因及时为了解决在子线程中无法访问UI的矛盾。

**系统为什么不允许在子线程中访问UI呢？**这是 因为Android的UI控件不是线程安全的，如果在多线程中并发访问可能会导致UI控件处于不可预期的状态，而为什么系统不对UI控件的访问加上锁机制呢？缺点有两个：首先加上锁机制会让UI访问的逻辑变得复杂，其次锁机制会降低UI访问的效率，因为锁机制会阻塞某些线程的执行。因此最简单最高效的方法就是采用单线程模型来处理UI操作，对于开发者来说也不是很麻烦，只是需要通过Handler切换到UI访问的执行线程即可。

Handler的工作过程：Handler创建时会采用当前线程的Looper来构建内部的消息循环系统，如果当前线程没有Looper，那么就会报错；Handler创建完毕后，这个时候其内部的MessageQueue和Looper就可以协同工作了，然后通过Handler的post方法将一个Runnable传递到Handler内部的Looper中去处理，也可以通过Handler的send方法来发送一个消息，这个消息同样会在Looper中去处理。其实post方法最终也是在send方法中完成的。当Handler的send方法被调用时，它会调用MessageQueue的enqueueMessage方法将这个消息放入消息队列中，然后Looper发现有新消息到来时，就会处理这个消息，最终消息中的Runnable或者Handler的handleMessage方法就会被调用。

注意：Looper是运行在创建Handler所在的线程中的，这样一来Handler中的业务逻辑就被切换到创建Handler所在的线程中去执行了。

## ThreadLocal的工作原理

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据。

下面通过实际的例子来演示ThreadLocal的作用。首先定义一个ThreadLocal对象，这里选择Boolean类型的，如下所示：
```Java
private ThreadLocal<Boolean> mThreadLocal = new ThreadLocal<Boolean>();
```
然后在主线程、子线程1和子线程2中设置和访问它的值，代码如下所示：

```Java
    mThreadLocal.set(true);
    System.out.println("main thread: " + mThreadLocal.get()); //true
    
    new Thread(new Runnable() {
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			mThreadLocal.set(false);
			System.out.println("thread 1: " + mThreadLocal.get()); //false
			
		}
	}).start();
    
    new Thread(new Runnable() {
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			
			System.out.println("thread 2: " + mThreadLocal.get()); //null，没有设置值
			
		}
	}).start();
```

上述代码中，根据对ThreadLocal的描述，主线中mThreadLocal.get()为true，子线程1中mThreadLocal.get()为false，子线程2中mThreadLocal.get()为null。代码运行打印如下：

```
15:54:55.478: I/System.out(11711): main thread: true
01-09 15:54:55.479: I/System.out(11711): thread 1: false
01-09 15:54:55.480: I/System.out(11711): thread 2: null
```

**从上面的打印可以看出**，虽然不同的线程访问的是同一个ThreadLocal对象，但是它们通过ThreadLocal获取的值却是不一样的，这就是ThreadLocal的特点。ThreadLocal之所以有如此特点，是因为不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取出一个数组，然后再从数组中根据当前的ThreadLocal索引去查找对应的value值。很显然，不同线程中的数组是不同的，这就是为什么通过ThreadLocal可以在不同的线程中维护一套数据的副本并且互不干扰。

上面描述的ThreadLocal的使用方法和工作过程，下面分析ThreadLocal的内部实现，ThreadLocal是一个泛型类，它的定义为`public class ThreadLocal<T>`，只要弄清楚ThreadLocal的get和set方法就可以明白它的工作原理。

首先看get和set方法，如下所示

```Java
    public void set(T value) {
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values == null) {
            values = initializeValues(currentThread);
        }
        values.put(this, value);
    }

    Values initializeValues(Thread current) {
        return current.localValues = new Values();
    }

    /**
     * Gets Values instance for this thread and variable type.
     */
    Values values(Thread current) {
        return current.localValues;
    }
```

在上面的set方法中，首先会通过values方法来获取当前线程中的ThreadLocal数据，**如何获取呢？**在Thread内部有一个成员变量专门用于存储线程的ThreadLocal数据：ThreadLocal.Values localValues;因此获取当前线程的ThreadLocal数据就变得异常简单。如果localValues的值为null，那么久需要对其进行初始化化，初始化后再将ThreadLocal的值进行存储。

下面看一下ThreadLocal的值到底是如何在localValues中进行存储的。在localValues内部有一个数组：private Object[] table;ThreadLocal的值就存储在这个table数组中。下面看一下localValues是如何使用put方法将ThreadLocal的值存储到table数组中的，如下所示：

```Java
    /**
     * Sets entry for given ThreadLocal to given value, creating an
     * entry if necessary.
     */
    void put(ThreadLocal<?> key, Object value) {
        cleanUp();

        // Keep track of first tombstone. That's where we want to go back
        // and add an entry if necessary.
        int firstTombstone = -1;

        for (int index = key.hash & mask;; index = next(index)) {
            Object k = table[index];

            if (k == key.reference) {
                // Replace existing entry.
                table[index + 1] = value;
                return;
            }

            if (k == null) {
                if (firstTombstone == -1) {
                    // Fill in null slot.
                    table[index] = key.reference;
                    table[index + 1] = value;
                    size++;
                    return;
                }

                // Go back and replace first tombstone.
                table[firstTombstone] = key.reference;
                table[firstTombstone + 1] = value;
                tombstones--;
                size++;
                return;
            }

            // Remember first tombstone.
            if (firstTombstone == -1 && k == TOMBSTONE) {
                firstTombstone = index;
            }
        }
    }
```

上面的代码实现了数据的存储过程，我们可以由上可以得出一个存储规则，那就是ThreadLocal的值在table数组中的存储位置总是为ThreadLocal的reference字段所标识的对象的下一个位置，比如ThreadLocal的reference对在table数组中的索引为index，那么ThreadLocal的值在table数组中的索引就是index+1.最终ThreadLocal的值会被存储在table数组中：table[index + 1] = value;

接下来，分析get方法，如下所示

```Java
    public T get() {
        // Optimized for the fast path.
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values != null) {
            Object[] table = values.table;
            int index = hash & values.mask;
            if (this.reference == table[index]) {
                return (T) table[index + 1];
            }
        } else {
            values = initializeValues(currentThread);
        }

        return (T) values.getAfterMiss(this);
    }
```

可以发现，ThreadLocal的get方法同样是取出当前线程的localValues对象，如果这个对象不为null，那就取出它的table数组并找出ThreadLocal的reference对象在table数组中的位置，然后table数组中下一个位置所存储的数据就是ThreadLocal的值。如果这个对象为null，则返回初始值，初始值由ThreadLocal的initialValue方法来描述，默认情况下为null，当然也可以重写这个方法，它的默认实现如下

```Java
    Object getAfterMiss(ThreadLocal<?> key) {
        Object[] table = this.table;
        int index = key.hash & mask;

        // If the first slot is empty, the search is over.
        if (table[index] == null) {
            Object value = key.initialValue();

            // If the table is still the same and the slot is still empty...
            if (this.table == table && table[index] == null) {
                table[index] = key.reference;
                table[index + 1] = value;
                size++;

                cleanUp();
                return value;
            }

            // The table changed during initialValue().
            put(key, value);
            return value;
        }

    protected T initialValue() {
        return null;
    }
```

>从ThreadLocal的set和get方法可以看出，它们所操作的对象都是当前线程的Values对象的table数组，因此在不同的线程中访问同一个ThreadLocal的set和get方法，它们对ThreadLocal所在的读/写权限仅限各自线程的内部，这就是ThreadLocal可以在不同线程中互不干扰的储存和修改数据的原因，理解ThreadLocal的工作方式有助于理解Looper的工作原理。


## 消息队列的工作原理

消息队列在Android中指的是MessageQueue，MessageQueue主要包括两个操作：插入和读取，读取操作会伴随着删除操作，插入和读取的方法分别是enqueueMessage和next，其中enqueueMessage的作用往往是往消息队列中插入一条消息，而next的作用是往消息队列中取出一条消息并将其从消息队列中移除。尽管MessageQueue叫消息队列，但是它的内部实现不是用的队列，而是通过一个单链表的数据结构来维护消息列表，单链表在插入和删除上效率较高。enqueueMessage的源码如下：

```Java
    boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w("MessageQueue", e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don not have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

从enqueueMessage的实现来看，它的主要操作就是单链表的插入和删除，下面看一下next方法的实现：

```Java
   Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
		//死循环
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (false) Log.v("MessageQueue", "Returning message: " + msg);
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf("MessageQueue", "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```

可以发现next方法是一个无限循环的过程，如果消息队列中没有消息，那么next方法会一直阻塞在这里。当有新的消息到来时，next方法会返回这条消息并将其从单链表中移除。

## Looper的工作原理

Looper在Android消息机制里面扮演着消息循环的角色，具体来说它会不停地从MessageQueue中查看是否有新消息，如有有新消息就会立刻处理，否则就会一直阻塞在那里。
首先看一下Looper的构造函数，在构造方法中它会创建一个MessageQueue即消息队列，然后将当前线程的对象存储起来，如下所示：

```Java
private static void prepare(boolean quitAllowed) {
	if (sThreadLocal.get() != null) {
		throw new RuntimeException("Only one Looper may be created per thread");
	}
	sThreadLocal.set(new Looper(quitAllowed));
}

private Looper(boolean quitAllowed) {
	mQueue = new MessageQueue(quitAllowed);
	mThread = Thread.currentThread();
}
```

Handler的工作需要Looper，没有Looper的线程就会报错，那么如何为一个线程创建Looper呢？通过Looper.prepare()即可为当前线程创建一个Looper，接着通过Looper.loop()来开启消息循环，如下所示：

```Java
    new Thread(new Runnable() {
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			Looper.prepare(); //创建Looper
			mHandler = new Handler(){
				
				@Override
				public void handleMessage(Message msg) {
					// TODO Auto-generated method stub
					super.handleMessage(msg);
					
					if (msg.what == 0) {
						
						System.out.println("msg: " + "123456"); 
					}
				}
			};
			Looper.loop(); //开启Looper循环
			
		}
	}).start();

    /**
     * 发送消息
     * @param view
     */
    public void click(View view) {
    	
    	mHandler.sendEmptyMessage(0);
    }

    /**
     * 退出Looper循环
     * @param view
     */
    public void quit(View view) {
    	mHandler.getLooper().quit();
		mHandler.getLooper().quitSafely(); //API18
    }
```

Looper除了prepare方法外，还提供了prepareMainLooper()方法，这个方法主要是给主线程也就是ActivityThread创建Looper使用的，其本质也是通过prepare方法创建的。由于主线程的Looper比较特殊，所以Looper提供了一个getMainLooper方法，通过它可以在任何位置获取主线程Looper。

Looper也是可以退出的，Looper提供了quit和quitSafely来退出一个Looper，二者的区别在于：quit会直接退出Looper，而quitSafely只是设定一个退出标志，然后把消息队列中的已有消息处理完毕后才安全地退出。Looper退出后，提供Handler发送的消息会失败，这时Handler的send方法返回false。在子线程中，如果手动为其创建了Looper，那么在所有的事情完成以后应该调用quit方法来终止消息循环，否则这个子线程会一直处于等待状态，而如果推出Looper以后，这个线程就会立刻终止，因此建议不需要的时候终止Looper。

Looper最重要的一个方法是loop方法，只有调用了loop后，消息循环系统才会真正的运行，实现如下：

```Java
   /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might bloc  退出时返回null
            if (msg == null) {
                // No message indicates that the message queue is quitting. 
                return; //位移跳出循环的方式
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg); //分发消息 msg.target = Handler

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```

Looper的loop方法工作过程，loop方法是一个死循环，位移跳出循环的方式是MessageQueue的next方法返回null。当Looper的quit方法被调用时，Looper就会调用MessageQueue的quit或者quitSafely方法来通知消息队列退出，当消息队列表计为退出状态时，它的next方法就返回null。

Looper必须退出，否则loop方法会无线循环下去。loop方法会调用MessageQueue的next方法来获取新的消息，而next是一个阻塞操作，当没有消息时，next方法会一直阻塞在那里，这也导致loop方法一直阻塞在那里。如果MessageQueue的next方法返回了新的消息，Looper就会处理这条消息：msg.target.dispatchMessage(msg);这里的msg.target是发送这条消息的Handler对象，这样Handler的发送的消息最终在它的dispatchMessage中处理了。

## Handler的工作原理

Handler的主要工作包含消息的发送和接收过程。消息的发送可以通过post的一系列方法以及send的一系列方法实现，post的一系列方法最终是通过send的一系列方法来实现的。发送一条消息典型过程如下所示：

```Java
public final boolean sendMessage(Message msg)
	{
		return sendMessageDelayed(msg, 0);
	}

public final boolean sendMessageDelayed(Message msg, long delayMillis)
	{
		if (delayMillis < 0) {
			delayMillis = 0;
		}
		return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
	}

public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
	MessageQueue queue = mQueue;
	if (queue == null) {
		RuntimeException e = new RuntimeException(
				this + " sendMessageAtTime() called with no mQueue");
		Log.w("Looper", e.getMessage(), e);
		return false;
	}
	return enqueueMessage(queue, msg, uptimeMillis);
}

//送入消息队列
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
	msg.target = this;
	if (mAsynchronous) {
		msg.setAsynchronous(true);
	}
	return queue.enqueueMessage(msg, uptimeMillis);
}
```

可以发现，Handler发送消息的过程仅仅是向消息队列插入了一条消息，MessageQueue的next方法就会返回这条消息给Looper，Looper收到消息后就开始处理了，最终消息由Looper交由Handler处理，即Handler的dispatchMessage方法会被调用，这是Handler就进入了消息处理阶段。dispatchMessage实现如下：

```Java
public void dispatchMessage(Message msg) {
	if (msg.callback != null) {
		handleCallback(msg);
	} else {
		if (mCallback != null) {
			if (mCallback.handleMessage(msg)) {
				return;
			}
		}
		handleMessage(msg);
	}
}
```

Handler处理消息的过程如下：

首先，检查Message的Callback是否为null，不为null则通过handleCallback来处理消息，Message的callback是一个Runnable对象，实际上就是Handler的post方法所传递的Runnable参数。

```Java
private static void handleCallback(Message message) {
	message.callback.run();  //messge.callback = Runnable对象
}
```

其次，若Message的Callback是为null，则检查mCallback是否为null，不为null就调用mCallback的handleMessage方法来处理消息，Callback是个接口，定义如下：

```Java
public interface Callback {
	public boolean handleMessage(Message msg);
}
```
通过Callback框图用如下方式创建Handler对象：Handler mHandler = new Handler(mCallback)。那么Callback的意义是什么了？可以用来创建一个Handler的实例但并不需要派生Handler的子类。

最后，若都为null，则直接调用Handler中的handlerMessage方法来处理消息。

Handler还有一种特使的构造函数，那就是通过一个特定的Looper来构造Handler，通过这个构造方法可以实现一些特殊的功能如IntentService，它的实现如下所示

```Java
public Handler(Looper looper) {
	this(looper, null, false);
}
```

Handler的一个默认构造方法public Handler()，这个构造方法会调用下面的的构造方法。很明显，如果当前线程没有Looper的话，就会抛出异常，这也解释了在没有Looper的子线程创建Handler会引发程序异常的原因。

```Java
public Handler() {
	this(null, false);
}

public Handler(Callback callback, boolean async) {

	...

	mLooper = Looper.myLooper();
	if (mLooper == null) {
		throw new RuntimeException(
			"Can't create handler inside thread that has not called Looper.prepare()");
	}
	mQueue = mLooper.mQueue;
	mCallback = callback;
	mAsynchronous = async;
}
```

# 主线程的消息循环

Android的主线程就是ActivityThread，主线程的入口方法为main，在main方法中系统会通过Looper.prepareMainLooper()来创建主线程的Looper以及MessageQueue，并通过Looper.loop()开启主线程的消息循环，如下所示：

```Java
public static void main(String[] args) {
	SamplingProfilerIntegration.start();

	// CloseGuard defaults to true and can be quite spammy.  We
	// disable it here, but selectively enable it later (via
	// StrictMode) on debug builds, but using DropBox, not logs.
	CloseGuard.setEnabled(false);

	Environment.initForCurrentUser();

	// Set the reporter for event logging in libcore
	EventLogger.setReporter(new EventLoggingReporter());

	Security.addProvider(new AndroidKeyStoreProvider());

	Process.setArgV0("<pre-initialized>");

	Looper.prepareMainLooper(); //

	ActivityThread thread = new ActivityThread();
	thread.attach(false);

	if (sMainThreadHandler == null) {
		sMainThreadHandler = thread.getHandler();
	}

	AsyncTask.init();

	if (false) {
		Looper.myLooper().setMessageLogging(new
				LogPrinter(Log.DEBUG, "ActivityThread"));
	}

	Looper.loop(); //无限循环

	throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

主线程的消息循环开始以后，ActivityThread需要一个Handler来和消息队列进行交互，这个Handler就是ActivityThread.H，它内部定义了一组消息类型，主要包含了四大组件的启动和停止等过程，如下所示

ActivityThread通过ApplicationThread和AMS进行进程间通信，AMS以进程间通信的方式完成ActivityThread的请求后回调ApplicationThread中的Binder方法，然后ApplicationThread会向H发送消息，H收到消息后会将ApplicationThread中的逻辑切换到ActivityThread中去执行，即切换到主线程中执行，这个过程就是主线程的消息循环模型。

# 参考文献

Android开发艺术探索

