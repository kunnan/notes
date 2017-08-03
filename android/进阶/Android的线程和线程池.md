[TOC]

# 前言

线程在Android中是一个很重要的概念，从用途上来说，线程分为主线程和子线程，主线程主要处理和界面相关的事情，而子线程则往往用于执行耗时操作。由于Android的特性，如果在主线程中执行耗时操作那么就会导致程序无法及时地响应，因此耗时操作必须放在子线程中去执行。

在操作系统中，线程是操作系统调度的最小单元，同时线程又是一种受限的系统资源，即线程不可能无限制的产生，并且线程的创建和销毁都会有相应的开销。档系统中存在大量的线程时，系统会通过时间片轮转的方式调度每个线程，因此线程不可能做到绝对的并行，除非线程数量小于等于CPU核心数，一般来说这是不可能的。正确的做法是采用线程池，一个线程池会缓存一定数量的线程，通过线程池就可以避免因为频繁创建和销毁线程所带来的系统开销。

# 主线程和子线程

Android沿用了Java的线程模型，其中的线程也分为主线程和子线程，其中主线程也叫UI线程。主线程的作用是运行四大组件以及处理它们和用户的交互，而子线程的作用则是执行耗时任务，比如网络请求、I/O操作等。从Android3.0开始系统要求网络访问必须在子线程中进行，否则网络访问将会失败并抛出NetworkOnMainThreadException这个异常，这样做事为了避免主线程由于耗时操作所阻塞而出现ANR异常。


# Android中的线程形态


除了传统的Thread线程外，Android还提供了AsyncTask、HandlerTask以及IntentService，这三者的底层实现也是线程，但它们具有特殊的表现形式，同时在使用上也各有优缺点。

## AsyncTask

AsyncTask是一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程上更新UI。是实现上来说，**AsyncTask封装了Thread和Handler**，通过AsyncTask可以更加方便地执行后台任务以及在主线程中访问UI，但是AsyncTask并不适合进行特别耗时的任务，对应特别耗时的任务来说，建议使用线程池。

### AsyncTask使用

AsyncTask是一个抽象的泛型类，它提供了Params、Progress和Result这三个泛型参数，其中Params表示输入参数的类型，Progress表示后台任务的执行进度的类型，而Result则表示后台任务返回结果的类型，如果AsyncTask确实不需要传递具体的参数，那么这三个泛型可以用Void来代替。声明如下：

```Java
	public abstract class AsyncTask<Params,Progress,Result>
```

AsyncTask提供了4个核心方法，它们的含义如下图所示

1. onPreExecute()，在主线程中执行，在异步任务执行之前会调用此方法，一般可以用于做一些准备工作。

2. doInBackground(Params...params)，在线程池中执行，用于执行异步任务，params表示异步任务的输入参数。在该方法中可以通过调用publishProgress方法来更新任务的进度，因为publishProgress会调用onProgressUpdate方法。

3. onProgressUpdate(Progress...values)，在主线程中执行，当后台任务的执行进度发生改变时此方法会被调用。

4. onPostExecute(Result result)，在主线程中执行，在异步任务执行之后，次方法会被调用，其中result参数是后台任务的返回值，即doInBackground的返回值。  

上述方法中，onPreExecute先执行，然后是doInBackground，最后才是onPostExecute。此外AsyncTask还提供了onCancelled()方法，它同样在主线程中执行，当异步任务被取消时，onCancelled()方法会被调用，这个时候onPostExecute则不会被调用。

下面代码为AsyncTask的一个应用实例： 

```Java
	private class LoadRecordTask extends
			AsyncTask<Object, VideoInfo, List<VideoInfo>> {

		@Override
		protected void onPreExecute() {
			// TODO Auto-generated method stub
			super.onPreExecute();
			mDescLoad.setVisibility(View.VISIBLE);
			mDescLoad.setText(R.string.refreshing);
			mVideoRecords.setEnabled(false);
		}

		@Override
		protected List<VideoInfo> doInBackground(Object... params) {
			// TODO Auto-generated method stub
			videoInfos = (ArrayList<VideoInfo>) MediaContentResolverUtils
					.getVideoInfoList(RecordVideoActivity.this);

			mVideoThumbnailMap = (HashMap<String, String>) mVideoThumbnailDao
					.findAllToMap();

			if (videoInfos == null || videoInfos.size() == 0) {
				return videoInfos;
			}

			// 没有缩略图 获取缩略图
			for (VideoInfo info : videoInfos) {

				String md5Name = Md5Utils.encode(info.getFileTitle());

				if (!mVideoThumbnailMap.containsKey(md5Name)) {
					//数据处理
				}

				publishProgress(info);

				if (isCancelled()) { //异步任务取消时会调用
					break;
				}
			}

			return videoInfos;
		}

		@Override
		protected void onProgressUpdate(VideoInfo... values) {
			// TODO Auto-generated method stub
			super.onProgressUpdate(values);

			for (VideoInfo info : values) {
				//UI更新进度
			}
		}

		@Override
		protected void onPostExecute(List<VideoInfo> result) {

			//取得后台任务的结果，更新UI
		}

		/**
		 * 运行在UI线程，调用cancel()方法后触发，在doInBackground()方法结束后执行
		 */
		@Override
		protected void onCancelled(List<VideoInfo> result) {
			// TODO Auto-generated method stub
			super.onCancelled(result);
		}
	}
```

运行和取消该任务的代码如下:

```Java
	mLoadRecordTask = new LoadRecordTask();
	mLoadRecordTask.execute();
	mLoadRecordTask.cancel(true); //结束任务
```

#### AsyncTask条件限制

*	AsyncTask的类必须在主线程中加载
*	AsyncTask的对象必须在主线程中创建
*	execute方法必须在UI线程调用
*	不要在程序中直接调用onPreExecute()、onPostExecute()、doInBackgroud()和onProgressUpdate()
*	一个AsyncTask对象只能执行一次，即只能调用一次execute方法，否则会报运行时异常
*	在Android1.6之前，AsyncTask是串行执行任务的，Android1.6的时候AsyncTask开始采用线程池里处理并行任务，但从Android3.0开始，为了避免AsyncTask所带来的并发错误，AsyncTask又采用一个线程来串行执行任务。**尽管如此，在Android3.0及以后版本中，我们仍然可以通过AsyncTask的executeOnExecutor方法（不能向下兼容）来并行的执行任务**

### AsyncTask工作原理

我们从AsyncTask的execute方法开始分析，execute方法又会调用ecuteOnExecutor方法，它们的实现如下:

```Java
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }

    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:" //异步任务执行一次
                            + " the task is already running."); 
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:" //异步任务执行一次
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute(); //最先执行

        mWorker.mParams = params;
        exec.execute(mFuture); //线程池开始执行

        return this;
    }
```

上述代码中，sDefaultExecutor实际上是一个串行的线程池，**一个进程中所有的AsyncTask全部在这个串行的线程池中排队执行**。在executeOnExecutor方法中，AsyncTask的onPreExecute()最先执行，然后线程池开始执行。下面分析线程池的执行过程，如下所示：

```Java
    public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                return postResult(doInBackground(mParams));
            }
        };
		
		//将AsyncTask的Params参数封装到FutureTask对象中，FutureTask的run方法会调用mWorker的call方法
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occured while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }

    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors(); //CPU核心数
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1; //核心工作线程
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1; //最多工作线程
    private static final int KEEP_ALIVE = 1; //空闲线程的超时时间为1秒

    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);

	public static final Executor SERIAL_EXECUTOR = new SerialExecutor();  
	private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;

	//实现一个线程池
    private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

		//线程同步
        public synchronized void execute(final Runnable r) {
			//将任务r插入mTasks任务队列中
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run(); //执行任务
                    } finally {
                        scheduleNext(); //继续执行下一个任务
                    }
                }
            });
			
			//没有真正活动的AsyncTask时调用
            if (mActive == null) {
                scheduleNext(); 
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive); //真正执行任务
            }
        }
    }
```

从SerialExecutor的实现可以分析AsyncTask的排队执行情况。首先系统会将AsyncTask的Params参数封装到FutureTask对象中，FutureTask是一个并发类，在这里它充当了Runnable的作用(FutureTask实现了Runnable方法)。接着这个FutureTask即mFuture会交给SerialExecutor的execute方法去处理。SerialExecutor的execute方法首先会把FutureTask对象添加到任务队列mTasks中，如果当前没有正在活动的AsyncTask任务，那么就会调用SerialExecutor的scheduleNext方法来执行下一个AsyncTask任务，否则等待当前AsyncTask任务完成再继续执行新的AsyncTask任务，直到所有的AsyncTask任务执行完毕。**从这可以看出，AsyncTask是串行执行任务的**

AsyncTask中有两个线程池（SerialExecutor和THREAD_POOL_EXECUTOR）和一个Handler（InternalHandler），其中线程池SerialExecutor用于执行任务的排队，线程池THREAD_POOL_EXECUTOR用于真正地执行AsyncTask任务，InternalHandler用于将执行环境从线程池切换到主线程。在AsyncTask的构造方法中有如下这么一段代码，由于FutureTask的run方法调用mWorker的call方法，因此mWorker的call方法最终会在线程池中执行。  

```Java
	public AsyncTask() {
	    mWorker = new WorkerRunnable<Params, Result>() {
	        public Result call() throws Exception {
	            mTaskInvoked.set(true); //表示当前任务以及调用过了
	
	            Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
	            //noinspection unchecked
	            return postResult(doInBackground(mParams)); //执行doInBackground方法
	        }
	    };
		
		//将AsyncTask的Params参数封装到FutureTask对象中，FutureTask的run方法会调用mWorker的call方法
	    mFuture = new FutureTask<Result>(mWorker) {
	        @Override
	        protected void done() {
	            try {
	                postResultIfNotInvoked(get());
	            } catch (InterruptedException e) {
	                android.util.Log.w(LOG_TAG, e);
	            } catch (ExecutionException e) {
	                throw new RuntimeException("An error occured while executing doInBackground()",
	                        e.getCause());
	            } catch (CancellationException e) {
	                postResultIfNotInvoked(null);
	            }
	        }
	    };
	}
```

在mWorker的call方法中，首先将mTaskInvoked设为true，表示当前任务以及被调用了，然后执行AsyncTask的doInBackground方法，接着将其返回值传递给postResult方法，它的实现如下：

```Java
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```

在上面的代码中，postResult方法会通过sHandler发送一个MESSAGE_POST_RESULT的消息，这个sHandler的定义如下所示：

```Java
	private static InternalHandler sHandler;

    private static Handler getHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler();
            }
            return sHandler;
        }
    }

	private static class InternalHandler extends Handler {
	    public InternalHandler() {
	        super(Looper.getMainLooper());
	    }
	
	    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
	    @Override
	    public void handleMessage(Message msg) {
	        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
	        switch (msg.what) {
	            case MESSAGE_POST_RESULT:
	                // There is only one result
	                result.mTask.finish(result.mData[0]);
	                break;
	            case MESSAGE_POST_PROGRESS:
	                result.mTask.onProgressUpdate(result.mData);
	                break;
	        }
	    }
	}
```

可以发现，sHandler是一个静态的Handler类对象，为了能够将执行环境切换到主线程，这就sHandler这个对象必须在主线程中创建。**由于静态成员会在加载类的时候进行初始化，因此这就变相要求AsyncTask的类必须在主线程中加载，否则同一进程中的AsyncTask都将无法正常工作**。sHandler收到sHandlerMESSAGE_POST_PROGRESS会调用onProgressUpdate方法更新进度，收到MESSAGE_POST_RESULT这个消息后会调用AsyncTask的finish方法，如下

```Java
    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
```

AsyncTask的finish方法会判断AsyncTask是否取消执行了，是则调用onCancelled方法，否则调用onPostExecute(result)，此时doInBackground的返回结果会传递给onPostExecute方法，最后将任务状态mStatus置为完成。至此AsyncTask的整个过程就分析完成了。

**通过分析AsyncTask的源码，可以进一步确定，从Android3.0开始，默认情况下AsyncTask的确是串行执行。**我们仍然可以通过AsyncTask的executeOnExecutor方法（不能向下兼容）来并行的执行任务。


## HandlerThread


HandlerThread继承了Thread，它是一种可以使用Handler的Thread，它的实现也很简单，就是在run方法中通过Looper.prepare()来创建消息队列，并通过Looper.loop()来开启消息循环，这样在实际的使用中就允许HandlerThread中创建Handler。HandlerThread的run方法如下所示：

```Java
public void run() {
	mTid = Process.myTid();
	Looper.prepare();
	synchronized (this) {
		mLooper = Looper.myLooper();
		notifyAll();
	}
	Process.setThreadPriority(mPriority);
	onLooperPrepared();
	Looper.loop();
	mTid = -1;
}
```

从HandlerThread的实现来看，它和普通的Thread有显著的不同之处。普通Thread主要同于run方法中执行一个耗时任务，而HandlerThread在内部创建了消息队列，外界需要通过Handler的消息方式来通知HandlerThread执行一个具体任务。HandlerThread是个很有用的类，它在Android中的一个具体的使用场景是IntentService。由于HandlerThread的run方法是一个无限循环，因此当明确不需要再使用HandlerThread时，可以通过它的quit或者quitSafely方法来终止线程的执行，这是一个好的编程习惯。示例代码如下:

```Java
	public class HandlerThreadDemo extends Activity {
	
		private Looper mLooper;
		private MyHandlerThread mHandlerThread;
		private TextView mInfoTxt;
		
		private Handler mHandler;
		
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_thread);
			
			mInfoTxt  = (TextView) findViewById(R.id.tv_info);
			
			mHandlerThread = new MyHandlerThread("mHandlerThread");
			mHandlerThread.start(); //先start
			mLooper = mHandlerThread.getLooper();
			
			//注册到Handler，通过Handler发送消息
			mHandler = new Handler(mLooper,mHandlerThread);
			
		}
		
		public void click(View view) {
			
			mHandler.sendEmptyMessage(1);
			
		}
		
		private class MyHandlerThread extends HandlerThread implements Callback {
	
			public MyHandlerThread(String name) {
				super(name);
				// TODO Auto-generated constructor stub
			}
	
			@Override
			public boolean handleMessage(Message msg) {
				// TODO Auto-generated method stub
				
				if (msg.what == 1) {
					System.out.println("mHandlerThread");
					mInfoTxt.setText("mHandlerThread");
				}
				
				return true;
			}
		}
	}
```

## IntentService

IntentService是一种特殊的Service，它继承了Service并且它是一个抽象类，因此必须创建它的子类才能使用IntentService。IntentService可用于执行后台耗时的后台，当任务执行后它会自动停止，同时由于IntentService是服务的原因，这导致它的优先级比单纯的线程要高很多，所以IntentService比较适合执行一些高优先级的后台任务，因为它的优先级高不容易被系统杀死。在实现上，IntentService封装了HandlerThread和Handler，这一点可以在它的onCreate方法中看出来，如下所示。

```Java
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
```

当IntentService被第一次启动时，它的onCreate方法会被调用，onCreate方法会创建一个HandlerThread，然后使用它的Looper来构造一个Handler对象mServiceHandler，这样通过mServiceHandler发送的消息最终都会在HandlerThread中执行，从这个角度来看，IntentService也可以用于执行后台任务。**每次启动IntentService，它的onStartCommand方法就会调用一次**，IntentService在onStartCommand中处理每个后台任务的Intent。下面看一下onStartCommand方法是如何处理外界Intent的，onStartCommand调用了onStart，onStart方法的实现如下所示：

```Java
public void onStart(Intent intent, int startId) {
	Message msg = mServiceHandler.obtainMessage();
	msg.arg1 = startId;
	msg.obj = intent;
	mServiceHandler.sendMessage(msg);
}
```

可以看出，IntentService仅仅是通过mServiceHandler发送了一个消息，这个消息会在HandlerThread中去处理。mServiceHandler收到消息后，会将Intent对象对象传递给onHandleIntent方法去处理。注意这个Intent对象的内容和外界的startService(intent)中的intent的内容是完全一致的，通过这个Intent对象即可解析出外界启动IntentService时所传递的参数，通过这些参数就可以区分具体的后台任务，这样在onHandleIntent方法中就可以对不同的后台任务做处理了。当onHandleIntent方法执行结束后，IntentService会通过stopSelf（int startId）来尝试停止服务。**这里之所以采用stopSelf（int startId）而不是stopSelf（）来停止服务，是因为stopSelf（）会立刻停止服务，而这个时候还可能有其他消息未处理，stopSelf（int startId）则会等待所有的消息都处理完毕后才终止服务**。一般来说，stopSelf（int startId）在尝试停止服务之前会判断最近启动服务的次数是否和startId相等，如果相等就立刻停止服务，不相等则不停止服务，这个策略可以从AMS的stopServiceToken方法的实现中找到依据。ServiceHandler的实现如下所示：

```Java
private final class ServiceHandler extends Handler {
	public ServiceHandler(Looper looper) {
		super(looper);
	}

	@Override
	public void handleMessage(Message msg) {
		onHandleIntent((Intent)msg.obj);
		stopSelf(msg.arg1);
	}
}
```

IntentService的onHandleIntent方法是一个抽象方法，它需要我们在子类中实现，它的作用是从Intent参数中区分具体的任务并执行这些任务。如果目前只存在一个后台任务，那么onHandleIntent(Intent)方法执行完这个任务后，stopSelf（int startId）就会直接停止服务；如果目前存在多个后台任务，那么当onHandleIntent方法执行完最后一个任务时，stopSelf（int startId）才会直接停止服务。另外，由于没执行一个后台任务就必须启动一次IntentService，而IntentService内部则通过消息的方式向HandlerThread请求执行任务，Handler中的Looper是顺序处理消息的，这就意味着IntentService也是顺序执行后台任务，当有多个后台任务同时存在时，这些后台任务会按照外界发起的顺序排队执行。

下面通过一个示例来说明IntentService的工作方式，首先派生一个IntentService的子类，它的实现如下所示：

```Java
public class LocalIntentService extends IntentService {

	public LocalIntentService() {
		super("LocalIntentService");
		// TODO Auto-generated constructor stub
	}
	
	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
		// TODO Auto-generated method stub
		System.out.println("onStartCommand");
		return super.onStartCommand(intent, flags, startId);
	}

	@Override
	protected void onHandleIntent(Intent intent) {
		// TODO Auto-generated method stub
		String action = intent.getStringExtra("task");
		System.out.println("action: " + action);
		SystemClock.sleep(3000); //休眠模拟耗时的后台任务
		if (action.equals("task1")) {
			System.out.println("handle action: " + action);
		}

		if (action.equals("task2")) {
			System.out.println("handle action: " + action);
		}
		
		if (action.equals("task3")) {
			System.out.println("handle action: " + action);
		}
	}

	@Override
	public void onDestroy() {
		// TODO Auto-generated method stub
		System.out.println("onDestroy");
		super.onDestroy();
		
	}
}
```

LocalIntentService实现完成以后，就可以在外界请求执行后台任务了，下面在Activity中发起3个后台任务的请求，如下所示:

```
    Intent service = new Intent(this, LocalIntentService.class);
    service.putExtra("task", "task1");
    startService(service);
    service.putExtra("task", "task2");
    startService(service);
    service.putExtra("task", "task3");
    startService(service);
```

运行程序，观察日记如下

```
22:14:19.407: I/System.out(16384): onStartCommand
01-08 22:14:19.407: I/System.out(16384): action: task1
01-08 22:14:19.407: I/System.out(16384): onStartCommand
01-08 22:14:19.408: I/System.out(16384): onStartCommand
01-08 22:14:22.407: I/System.out(16384): handle action: task1
01-08 22:14:22.409: I/System.out(16384): action: task2
01-08 22:14:25.410: I/System.out(16384): handle action: task2
01-08 22:14:25.418: I/System.out(16384): action: task3
01-08 22:14:28.418: I/System.out(16384): handle action: task3
01-08 22:14:28.429: I/System.out(16384): onDestroy
```

从日志可以看出，三个后台任务是排队执行的，它们的执行顺序就是它们发起请求对的顺序。当task3执行完毕后，LocalIntentService才真正地停止，执行了onDestroy方法。

# Android中的线程池

线程池的有点主要有三点：

*	重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销

*	能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象。

*	能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能。

Android中的线程池概念来源于Java中的Executor，Executor是一个接口，真正的线程池的实现为ThreadPoolExecutor。ThreadPoolExecutor提供了一系列参数来配置线程池，通过不同的参数可以创建不同的线程池。由于Android中的线程池都是直接或者间接通过配置ThreadPoolExecutor来实现的，因此需要先介绍ThreadPoolExecutor。

## ThreadPoolExecutor

ThreadPoolExecutor是线程池的真正实现，它的构造方法提供了一系列参数来配置线程池，下面介绍ThreadPoolExecutor的构造方法中各个参数的含义，这些参数将会直接影响到线程池的功能特性，下面是ThreadPoolExecutor的一个比较常用的构造方法。

```Java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory) {
    }
```

*	**corePoolSize**  

线程池的核心线程数，默认情况下，核心线程会在线程池中一直存活，即使他们处于闲置状态。如果将ThreadPoolExecutor的allowCoreThreadTimeout属性设置为true，那么闲置的核心线程在等待新任务到来会有超时策略，这个时间间隔由keepAliveTime所指定，当等待时间超时keepAliveTime所指定的时长后，核心线程会被终止。

*	**maximumPoolSize**

线程池所能容纳的最大线程数，当活动线程达到这个数值后，后续的新任务将会被阻塞。

*	**keepAliveTime**

非核心线程闲置时的超时时长，超过这个时间，非核心线程就会被收回。当ThreadPoolExecutor的allowCoreThreadTimeout属性设置为true时，keepAliveTime同样会作用于核心线程。

*	**unit**

用于指定keepAliveTime参数的时间单位，这是一个枚举，常用的有TimeUnit.MILLISECONDS;TimeUnit.SECONDS;TimeUnit.MINUTES等

*	**workQueue**

线程池的任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中。

* **ThreadFactory**

线程工厂，为线程池提供创建新线程的功能。ThreadFactory是一个接口，它只有一个方法：Thread newThread(Runnable r);

除上面的这些主要的参数外，ThreadPoolExecutor还有一个不常用的参数RejectedExecutionHandler。当线程池无法执行新的任务时，这可能是由于任务队列已满或者是无法成功执行任务，这个时候ThreadPoolExecutor回调用RejectedExecutionHandler的rejectedExecution(Runnable r, ThreadPoolExecutor executor)方法来通知调用者，默认情况下rejectedExecution会直接抛出一个RejectedExecutionException的运行时异常。ThreadPoolExecutor为RejectedExectutionHandler提供了几个可选值：CallerRunsPolicy、AbortPolicy、DiscardPolicy、DiscardOldestPolicy，其中AbortPolicy是默认值，但是RejectedExecutionHandler这个参数不常用。

ThreadPoolExecutor执行任务时大致遵循如下规则：

*	如果线程池中的线程数量未达到核心线程的数量，那么会直接启动一个核心线程来执行任务

*	如果线程池中的线程数量已经达到或超过核心线程的数量，那么任务会被插入到任务队列中排队等待执行。

*	如果在步骤2中无法将任务插入到任务队列中，这往往是由于任务队列已满，这个时候如果线程数量未达到线程池规定的最大值，那么会立刻启动一个非核心线程来执行任务。

*	如果步骤3中线程数量已经达到线程池中规定的最大值，那么就拒绝执行此任务，ThreadPoolExecutor会调用RejectedExecutionHandler的rejectedExecution方法来通知调用者。

ThreadPoolExecutor的参数配置在AsyncTask中有明显的体现，下面是AsyncTask中的线程池配置情况

```Java
    private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE = 1;

    private static final ThreadFactory sThreadFactory = new ThreadFactory() {
        private final AtomicInteger mCount = new AtomicInteger(1);

        public Thread newThread(Runnable r) {
            return new Thread(r, "AsyncTask #" + mCount.getAndIncrement());
        }
    };

    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * An {@link Executor} that can be used to execute tasks in parallel.
     */
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
```

AsyncTask线程池配置后的规格如下：

* 核心线程数等于CPU核心数+1
* 线程池的最大线程数为CPU的核心数的2倍 + 1
* 核心线程无超时机制，非核心线程在闲置时的超时时间为1秒
* 任务队列的容量为128


## 线程池的分类


*	**FixedThreadPool**

通过Executors的newFixedThreadPool方法来创建。它是一种线程数量固定的线程池，当线程处于空闲状态时，它们并不会被收回，除非线程池关闭了。当所有的线程都处于活动状态时，新任务都会处于等待状态，直到有线程空闲出来。由于FixedThreadPool只有核心线程并且这些核心线程都不会被回收，**这意味着它能够更加快速的响应外界的请求**。实现如下，可以发现FixedThreadPool中只有核心线程并且这些核心线程没有超时机制，另外任务队列也是没有大小限制的

```Java
	/*
     * @param nThreads the number of threads in the pool
     * @return the newly created thread pool
     * 
     */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

*	**CachedThreadPool**

通过Executors的newCachedThreadPool方法来创建。它是一种线程数量不定的线程池，它只有非核心线程，并且最大线程数为Integer.MAX_VALUE。由于Integer.MAX_VALUE是一个很大的数，实际上就相当于最大线程数可以任意大。当线程池中的线程都处于活动状态时，线程池会创建新的线程来处理新任务，否则就会利用空闲的线程来处理新的任务。线程池中的空闲线程都有超时机制，这个超时时长为60秒，超过60秒闲置线程就会被回收。和FixedThreadPool不同的是，CachedThreadPool的任务队列其实相当于一个空集合，这将导致任何任务都会立即被执行，因为这种情况下SynchronousQueue是无法插入任务的。SynchronousQueue是一个非常特殊的队列，很多情况下可以理解为一个无法存储元素的队列（实际中很少使用）。从CachedThreadPool的特性来看**这类线程池比较适合执行大量的耗时较少的任务**。当整个线程池都处于闲置状态时，线程池中的线程都会超时而被停止，这个时候CachedThreadPool之中是没有任何线程的，它几乎不占用任何系统资源的。

```Java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

*	**ScheduledThreadPoll**

通过Executors的newScheduledThreadPool方法来创建。它的核心线程数量是固定的，而非核心线程数量是没有限制的，并且当核心线程闲置时会被立即收回。ScheduledThreadPoll这类线程**主要用于执行定时任务和具有固定周期的重复任务**

```Java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
	return new ScheduledThreadPoolExecutor(corePoolSize);
}

public ScheduledThreadPoolExecutor(int corePoolSize,
								   ThreadFactory threadFactory) {
	super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
		  new DelayedWorkQueue(), threadFactory);
}
```

*	**SingleThreadExecutor**

通过Executors的newSingleThreadExecutor方法来创建。这类线程池内部只有一个核心线程，它确保所有的任务都在同一个线程中顺序执行。SingleThreadExecutor的意义在于统一所有的外界任务到一个线程中，这使得在这些任务之间不需要处理线程同步的问题。

```Java
public static ExecutorService newSingleThreadExecutor() {
	return new FinalizableDelegatedExecutorService
		(new ThreadPoolExecutor(1, 1,
								0L, TimeUnit.MILLISECONDS,
								new LinkedBlockingQueue<Runnable>()));
}
```

# 参考文献

Android开发艺术探索

