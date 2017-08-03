**Android中IPC的方式**

[TOC]

# Bundle

四大组件中的三大组件（Activity、Service、BroadcastReceiver）都是支持在Intent中传递Bundle数据的，由于Bundle实现了Parcelable接口，所以它可以方便地在不同的进程间传输，基于这一点，当我们在一个进程中启动了另一个进程的Activity、Service和BroadcastReceiver，我们就可以在Bundle中附加我们需要传输给远程进程的信息并通过Intent发送出去。当然，我们传输的数据必须能够被序列化，比如基本类型、实现了Parcelable接口的对象、实现了Serializable接口的对象以及一些Android所支持的特殊对象（如Bundle、Size、SizeF、IBinder）。

# 文件共享

共享文件也是一种不错的进程间通信方式，两个进程提供读/写同一个文件来交换数据，比如A进程把数据写入文件，B进程提供读取这个文件来获取数据。通过文件交换数据很好使用，除了可以交换一些文本信息外，我们还可以序列化一个对象到文件中，从另一进程中恢复这个对象。

通过文件共享方式来共享数据对文件格式是没有具体要求的，比如可以是文本文件，也可以是XML文件，只要读写双方约定数据格式即可。通过文件共享的方式是有局限性的，比如并发读/写的问题，因此我们要尽量避免并发写这种情况的发生或者考虑使用线程同步来限制多个线程的写操作。通过上面的分析可以知道，**文件共享方式适合在对数据同步要求不高的进程之间进行通信**，并且要妥善处理并发读/写的问题。

SharedPreference是个特例，SharedPreference是Android中提供的轻量级存储方案，它通过键值对的方式来存储数据，在底层实现上采用XML文件来存储键值对，每个应用的SharedPreference文件都可以在当前包所在的data目录下查到，一般来说，它的目录位于/data/data/package name/shared_prefs目录下。**从本质上来说，**SharedPreference属于文件的一种，但是由于系统对它的读/写有一定的缓存策略2，即在内存中会有一份SharedPreference文件的缓存。因此在多进程模式下，系统对它的读/写变得不可靠，当面对搞并发的读/写访问时，SharedPreference有很大几率会丢失数据，因此不建议在进程间通信中使用SharedPreference。


# Messenger

Messenger可以翻译为信使，通过它可以在不同进程中传递Message对象，在Message中放入我们需要传递的数据，就可以轻松实现数据的进程间传递,也可以在同一个进程中使用。Messenger是一种轻量级的IPC方案，它的底层实现是AIDL。Messenger类的构造方法如下

```Java
public Messenger(Handler target) {
	mTarget = target.getIMessenger();
}

public Messenger(IBinder target) {
	mTarget = IMessenger.Stub.asInterface(target);
}
```

Messenger使用简单，它对AIDL做了封装。同时，由于它一次处理一个请求，因此在服务端我们不用考虑线程同步的问题，这是因为服务端不存在并发执行的情形。实现一个Messenger由如下几个步骤，分为服务端和客户端。

*	**1. 服务端进程**

首先，我们需要在服务端创建一个Service来处理客户端的连接请求，同时创建一个Handler，并通过它来创建一个Messenger对象，然后在Service的onBind中返回这个Messenger对象底层的Binder即可。


*	**2. 客户端进程**

客户端进程中，首先要绑定服务端的Servcie，绑定成功后用服务端返回的IBinder对象创建一个Messenger，通过这个Messenger就可以向服务器发送消息了，发送消息类型为Message对象。如果需要服务端回应客户端蛮久和服务端一样，我们还需要创建一个Handler并创建一个新的Messenger，并把这个Messenger对象通过Messge的replyTo参数传递给服务端，服务端通过这个replyTo参数就可以回应客户端。

首先是服务端的代码

```Java
	public class MessengerService extends Service {
		
		//1.创建Handler对象处理Message
		private static class MessengerHandler extends Handler {
			
			@Override
			public void handleMessage(Message msg) {
				// TODO Auto-generated method stub
				switch (msg.what) {
				
				case Constants.MSG_FROM_CLIENT:
					
					System.out.println("receiver msg from client: " + msg.getData().get("msg"));
					
					//返回信息到服务端
					//获取客户端接收消息的Messenger
					Messenger client = msg.replyTo;
					Message replyMessage = Message.obtain();
					replyMessage.what = Constants.MSG_FROM_SERVER;
					Bundle data = new Bundle();
					data.putString("reply", "嗯，你的消息我已经收到!");
					replyMessage.setData(data);
					
					try {
						client.send(replyMessage);
					} catch (RemoteException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					
					break;
	
				}
				
				
				super.handleMessage(msg);
			}
		}
		
		//2.创建一个Messenger,将客户端发送的消息传递给MessengerHandler处理
		private final Messenger mMessenger = new Messenger(new MessengerHandler());
	
		//3.返回Messenger对象底层Binder
		@Override
		public IBinder onBind(Intent intent) {
			// TODO Auto-generated method stub
			return mMessenger.getBinder();
		}
	
	}
```

然后，注册Service

```Java
    <service
        android:name="com.ryg.chapter_2.messenger.MessengerService"
        android:process=":remote" >
    </service>
```

最后是客户端代码

```Java
	public class MainActivity extends Activity {
		
		private Messenger mService;
		
		//将服务端返回的消息传递MessengerHandler处理
		private Messenger mGetReplyMessenger = new Messenger(new MessengerHandler());
		
		private static class MessengerHandler extends Handler {
			
			@Override
			public void handleMessage(Message msg) {
				// TODO Auto-generated method stub
				
				switch (msg.what) {
				
				case Constants.MSG_FROM_SERVER:
					
					System.out.println("receiv msg from server: " + msg.getData().getString("reply"));
					
					break;
	
				}
				
				super.handleMessage(msg);
			}
		}
		
		private ServiceConnection mConnection = new ServiceConnection() {
			
			@Override
			public void onServiceDisconnected(ComponentName name) {
				// TODO Auto-generated method stub
				
			}
			
			@Override
			public void onServiceConnected(ComponentName name, IBinder service) {
				// TODO Auto-generated method stub
				//2.创建一个Messenger
				mService = new Messenger(service);
				
		    	//3.通过Messenger发送Message消息到服务端
		    	if (mService != null) {
					Message msg = Message.obtain();
					msg.what = Constants.MSG_FROM_CLIENT;
					Bundle data = new Bundle();
					data.putString("msg", "hello,this is client");
					msg.setData(data);
					
					//将接收服务端回复的Messenger传递给服务端，必须要传递过去，否则收不到回复
					msg.replyTo = mGetReplyMessenger;
					
					try {
						mService.send(msg);
					} catch (RemoteException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
	
				
			}
		};
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        
	        //1.绑定服务
	        Intent intent = new Intent(this, MessengerService.class);
	        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
	    }
	    
	    @Override
	    protected void onDestroy() {
	    	// TODO Auto-generated method stub
	    	if (mConnection != null) {
				
	    		unbindService(mConnection);
			}
	    	super.onDestroy();
	    }
	}
```

运行结果:

```
	 13:20:43.218: I/System.out(3280): receiver msg from client: hello,this is client
	01-11 13:20:43.234: I/System.out(3222): receiv msg from server: 嗯，你的消息我已经收到!
```

通过上面的例子可以看出，在Messenger中进行数据传递必须将数据放入Message中，而Messenger和Message都实现了Parcelable接口，因此可以跨进程传输。简单来说，Message中所支持的数据类型就是Messenger中所支持的传输类型。实际上，通过Messenger来传输Message，Message中能使用的载体只有what、arg1、arg2、Bundle以及reply。Message中的另一个字段object在同一进程中的很实用的，但是再进程间通信的时候，在Android2.2以前object字段不支持跨进程传输，即便是android2.2以后，也仅仅是系统提供的实现了Parcelable接口的对象才能通过它来传输，这就意味着我们自定义的Parcelable对象无法通过object字段来传输。因此使用Bundle可以支持大量的数据类型。

# AIDL

Messenger是以串行的方式处理客户端发来的消息，如果大量的消息同时发送到服务器，服务端仍然只能一个一个处理,（1）如果有大量的并发请求，那么用Messenger就不太合适了。同时，Messenger的作用仅仅是为了传递消息，（2）很多时候我们可能需要跨进程调用服务端的方法，这种情形用Messenger就无法做到了，但是我们可以使用AIDL来实现跨进程的方法调用。

下面介绍使用AIDL来进行进程间通信的流畅，分为服务端和客户端两个方面

## 服务端

服务端要首先创建一个Service用来监听客户端的连接，然后创建一个AIDL文件，将暴露给客户端的接口再这个AIDL文件中声明，最后在Service中实现这个AIDL接口。

（1）AIDL接口的创建  
创建一个后缀为aidl的文件，在里面声明了一个接口和两个接口方法

```Java
	//IBookManager.aidl
	
	package com.ryg.chapter_2.aidl;
	
	import com.ryg.chapter_2.aidl.Book;
	
	interface IBookManager {
	
		List<Book> getBookList();
		void addBook(in Book book);
		
	}
```

在AIDL文件中，并不是所有的数据类型都是可以使用的，AIDL到底支持哪些数据类型呢？如下所示

*	基本数据类型
*	String和CharSequence
*	List：只支持ArrayList，并且里面的每个元素必须能够被AIDL支持
*	Map： 只支持HashMap，并且里面的每个元素都必须能够被AIDL文件支持，包括key和value
*	Parcelable：所有实现了Parcelable接口的对象
*	AIDL：所有的AIDL接口本身也可以在AIDL文件中使用

以上6中就是AIDL支持的数据类型，其中自定义的Parcelable对象和AIDL对象必须要显示的import进来，不管它们是否和当前的AIDL文件位于同一个文件夹内。

IBookManager.aidl文件中引用了Book这个类，Book类是一个自定义的Parcelable对象，所以必须新建一个与它同名的AIDL文件，并在其中声明它为Parcelable，如下所示：

```
	//Book.aidl
	
	package com.ryg.chapter_2.aidl;
	
	parcelable Book;
```

**注意：**

除此之外，AIDL中除了基本数据类型，其他类型的参数必须标上方向：in、out或者inout，in表示输入型参数，out表示输出型参数，inout表示输入输出型参数。

AIDL的包结构在客户端工程和服务端工程中要保持一致，否则会运行出错，这是因为客户端需要反序列化服务端中和AIDL接口相关的所有类，如果类的完整路径不一致，就无法成功反序列化，程序也就无法正常运行。

（2）远程服务端service的实现  

```Java

	public class BookManagerService extends Service {
	
		private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<Book>();
		
		private Binder mBinder = new IBookManager.Stub() {
			
			@Override
			public List<Book> getBookList() throws RemoteException {
				// TODO Auto-generated method stub
				return mBookList;
			}
			
			@Override
			public void addBook(Book book) throws RemoteException {
				// TODO Auto-generated method stub
				if (!mBookList.contains(book)) {
					mBookList.add(book);
				}
			}
		};

		@Override
		public void onCreate() {
			// TODO Auto-generated method stub
			super.onCreate();
			
			mBookList.add(new Book(1, "Android"));
			mBookList.add(new Book(2, "IOS"));
		}
		
		@Override
		public IBinder onBind(Intent intent) {
			// TODO Auto-generated method stub
			return mBinder;
		}
	
	}
```

上面是一个服务端Service的典型实现，首先在onCreate中初始化添加两本书的信息，然后创建一个Binder对象并在Binder中返回次对象，这个对象继承自IBookManager.Stub并实现了内部的AIDL方法。这里采用了CopyOnWriteArrayList，这个CopyOnWriteArrayList支持并发读/写。AIDL方法是在服务端的Binder线程池中执行的，因此当多个客户端同时连接的时候，会存在多个线程同时访问的情形，所以我们要在AISL方法中处理线程的同步，这里使用CopyOnWriteArrayList来进行自动的线程同步。

AIDL中能够使用的List只有ArrayList，但是我们这里使用的CopyOnWriteArrayList不是继承自ArrayList，为什么能够正常工作呢？这是因为AIDL所支持的是抽象的List，而List只是一个接口，因此虽然服务端返回的是CopyOnWriteArrayList，但在Binder中会按照List的规范去访问数据并最终形成一个ArrayList传递给客户端。所以，在服务端采用CopyOnWriteArrayList是完全可行的，与此类似的类还有ConCureentHashMap。

注册Service

```
    <service
        android:name="com.ryg.chapter_2.aidl.BookManagerService"
        android:process=":remote1" >
    </service>
```

## 客户端

客户端首先要绑定远程服务，绑定成功后将服务端返回的Binder对象转换成AIDL接口，然后就可以通过这个接口去调用服务端的远程方法了，如下

```Java
	private ServiceConnection mConnection = new ServiceConnection() {
		
		@Override
		public void onServiceDisconnected(ComponentName name) {
			// TODO Auto-generated method stub
			
		}
		
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			// TODO Auto-generated method stub
			IBookManager bookManager = IBookManager.Stub.asInterface(service);
			
			try {
				List<Book> list = bookManager.getBookList();
				System.out.println("query book list type: " + list.getClass().getCanonicalName());
				System.out.println("query book list: " + list.toString());
				
				bookManager.addBook(new Book(3, "Windows Phone"));
				list = bookManager.getBookList();
				System.out.println("query book list: " + list.toString());
				
			} catch (RemoteException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			
		}
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		Intent intent = new Intent(this, BookManagerService.class);
		bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
	}
```

运行结果:

```
	 14:51:36.937: I/System.out(24841): query book list type: java.util.ArrayList  //CopyOnWriteArrayList转为Arraylist
	01-11 14:51:36.937: I/System.out(24841): query book list: [Book [bookId=1, bookName=Android], Book [bookId=2, bookName=IOS]]
	01-11 14:51:36.939: I/System.out(24841): query book list: [Book [bookId=1, bookName=Android], Book [bookId=2, bookName=IOS], Book [bookId=3, bookName=Windows Phone]]
```

## Binder意外死亡的处理办法

（1）给Binder设置DeathRecipinent监听，当Binder死亡时，会收到binderDied的回调，在此回调中重新连接远程服务，次方法在客户端的Binder线程池中调用，不能直接访问UI  

```Java
	private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
		
		//Binder死亡时的回调方法
		@Override
		public void binderDied() {
			// TODO Auto-generated method stub
			if (bookManager == null) {
				return;
			}
			
			bookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
			bookManager = null;
			
			//重新绑定远程服务
			Intent intent = new Intent(BookManagerActivity.this, BookManagerService.class);
			bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
			
		}
	};

	
	private ServiceConnection mConnection = new ServiceConnection() {
		
		@Override
		public void onServiceDisconnected(ComponentName name) {
			// TODO Auto-generated method stub
			
		}
		
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			// TODO Auto-generated method stub
			bookManager = IBookManager.Stub.asInterface(service);
			//给binder设置死亡代理
			try {
				service.linkToDeath(mDeathRecipient, 0);
			} catch (RemoteException e1) {
				// TODO Auto-generated catch block
				e1.printStackTrace();
			}
		}
```

（2）在onServiceDisconnected中重连远程服务，此方法在客户端的UI线程中调用  

```Java
	private ServiceConnection mConnection = new ServiceConnection() {
		
		@Override
		public void onServiceDisconnected(ComponentName name) {
			// TODO Auto-generated method stub
			//重新绑定远程服务
			Intent intent = new Intent(BookManagerActivity.this, BookManagerService.class);
			bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
			
		}
		
		@Override
		public void onServiceConnected(ComponentName name, IBinder service) {
			// TODO Auto-generated method stub
			bookManager = IBookManager.Stub.asInterface(service);
			//给binder设置死亡代理
			try {
				service.linkToDeath(mDeathRecipient, 0);
			} catch (RemoteException e1) {
				// TODO Auto-generated catch block
				e1.printStackTrace();
			}
	}
```

## AIDL中使用权限验证功能

默认情况下，我们的远程服务任何人都可以连接，所以我们必须给服务加入权限验证功能，权限验证失败则无法调用服务的方法，这里介绍两种常用的方法。

*	**onBind中验证**

早onBind中进行验证，验证不通过就直接返回null，这样验证失败的客户端无法绑定服务，比如使用permission验证。首先，在Manifest.xml中声明所需要的权限，比如：

```
    <permission 
        android:name="com.ryg.chapter_2.permission.ACCESS_BOOK_SERVICE"
        android:protectionLevel="normal">
```

然后在BookManagerService的onBind做权限验证，如下所示。

```Java
	@Override
	public IBinder onBind(Intent intent) {
		// TODO Auto-generated method stub
		int check = checkCallingOrSelfPermission("com.ryg.chapter_2.permission.ACCESS_BOOK_SERVICE");
		
		if (check == PackageManager.PERMISSION_DENIED) {
			return null; //客户端就无法绑定到此服务
		}
		
		return mBinder;
	}
```

这种方法同样适用于Messenger中。如果我们自己内部的应用想绑定到我们的服务中，只需要在它的AndroidManifest文件中使用permission即可

<uses-permission android:name="com.ryg.chapter_2.permission.ACCESS_BOOK_SERVICE"/>

*	**服务端的onTransact方法中验证**

在服务端的onTransact方法中进行权限验证，验证失败就返回false，这样服务端就不会终止执行AIDL中的方法从而达到保护服务端的效果。至于验证的方式有很多，如下

```Java
	private Binder mBinder = new IBookManager.Stub() {
		
		@Override
		public List<Book> getBookList() throws RemoteException {
			// TODO Auto-generated method stub
			return mBookList;
		}
		
		@Override
		public void addBook(Book book) throws RemoteException {
			// TODO Auto-generated method stub
			if (!mBookList.contains(book)) {
				mBookList.add(book);
			}
		}

		@Override
		public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
				throws RemoteException {
			// TODO Auto-generated method stub
			
			//1.通过permission验证
			int check = checkCallingOrSelfPermission("com.ryg.chapter_2.permission.ACCESS_BOOK_SERVICE");
			
			if (check == PackageManager.PERMISSION_DENIED) {
				return false;
			}
			
			//2.验证包名
			String packageName = null;
			String[] packages = getPackageManager().getPackagesForUid(getCallingUid());
			
			if (packages != null && packages.length > 0) {
				packageName = packages[0];
			}
			
			if (!packageName.startsWith("com.ryg")) {
				return false;
			}
			
			return super.onTransact(code, data, reply, flags);
		}
		
		
	};
```

上面介绍了常用的两种权限验证方式，但是还有其他方式做权限验证，比如为Service指定android:permission属性等。

# ContentProvider

ContentProvider是Android中专门用于不同应用间进行数据共享的方式，从这一点来看，它天生适合进程间通信。和Messenger一样，ContentProvider的底层实现同样是Binder。

系统预置了许多ContentProvider，比如通讯录信息
日程表信息等，要跨进程访问这些信息，只需要通过ContentResolver的query、update、insert和delete方法。下面我们演示实现一个自定义的ContentProvider，并演示如何在其他应用中获取ContentProvider中的数据从而实现进行间通信的目的。首先，创建一个ContentProvider的类，叫BookProvider，并实现6个抽象方法即可onCreate、query、delete、update、insert和getType。onCreate代表ContentProvider的创建，需要做一些初始化的工作；getType用来返回一个Uri请求所对应的MIME类型，比如视频、图片、等，如果应用不关心这个选项，可以直接在方法中返回null或者“\*/\*”剩下的四个方法对应于CRUD操作，对数据表的增删改查功能。

根据Binder的工作原理，这留个方法均运行在ContentProvider的进程中，除了onCreate有系统回调运行在主线程里，其他无非方法由外界回调并运行在Binder线程池中。

示例如下：

```Java
	//BookProvider.java
	public class BookProvider extends ContentProvider {
	
		@Override
		public boolean onCreate() {
			// TODO Auto-generated method stub
			System.out.println("onCreate current thread:" + Thread.currentThread().getName());
			return false;
		}
	
		@Override
		public Cursor query(Uri uri, String[] projection, String selection,
				String[] selectionArgs, String sortOrder) {
			// TODO Auto-generated method stub
			System.out.println("query current thread:" + Thread.currentThread().getName());
			return null;
		}
	
		@Override
		public String getType(Uri uri) {
			// TODO Auto-generated method stub
			return null;
		}
	
		@Override
		public Uri insert(Uri uri, ContentValues values) {
			// TODO Auto-generated method stub
			return null;
		}
	
		@Override
		public int delete(Uri uri, String selection, String[] selectionArgs) {
			// TODO Auto-generated method stub
			return 0;
		}
	
		@Override
		public int update(Uri uri, ContentValues values, String selection,
				String[] selectionArgs) {
			// TODO Auto-generated method stub
			return 0;
		}
	
	}
```

接下来我们需要注册这个BookProvider，如下所示。其中android:anthorities是ContentProvider的唯一标识，通过这个属性外部应用就可以访问我们的BookProvide。

```
    <provider
        android:name="com.ryg.chapter_2.provider.BookProvider"
        android:authorities="com.ryg.chapter_2.provider.book.provider" //标识
        android:permission="com.ryg.PROVIDER" //权限
        android:process=":provider"
        //ndroid:readPermission="com.ryg.PROVIDER.READ" //读权限
        //android:writePermission="com.ryg.PROVIDER.WRITE" >  //写权限
    </provider>
```

然后声明权限和加入权限

```
 	<uses-permission android:name="com.ryg.PROVIDER" />

    <permission
        android:name="com.ryg.PROVIDER"
        android:protectionLevel="normal" />
```

创建BookActivity.java访问这个ContentProvider，代码如下：

```Java
	public class BookActivity extends Activity {
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			
			Uri uri = Uri.parse("content://com.ryg.chapter_2.provider.book.provider");
			getContentResolver().query(uri, null, null, null, null);
			getContentResolver().query(uri, null, null, null, null);
			getContentResolver().query(uri, null, null, null, null);
			getContentResolver().query(uri, null, null, null, null);
		}
	}
```

上面的代码中，我们提供ContentResolver对象的query方法查询BookProvider中的数据，其中"content://com.ryg.chapter_2.provider.book.provider"位移标识了BookProvider，这个标识正式为BookProvider的android:authorities属性所指定的值。

运行结果如下

```
	 16:50:31.678: I/System.out(22482): onCreate current thread:main        //主线程
	01-11 16:50:31.680: I/System.out(22482): query current thread:Binder_2  //Binder线程池中
	01-11 16:50:31.681: I/System.out(22482): query current thread:Binder_1
	01-11 16:50:31.682: I/System.out(22482): query current thread:Binder_2
	01-11 16:50:31.682: I/System.out(22482): query current thread:Binder_1
```

从结果可以看出，onCreate运行于主线程中，所以不能在onCreate中做耗时操作，query方法的四次调用不在同一个线程中，但是在同一个Binder线程池中。

接下来，我们继续完善BookProvider，从而使其对外界的应用提供数据。为了完成上述功能，我们需要一个数据库来管理图示和用户信息，如下所示。

```Java
	public class BookOpenHelper extends SQLiteOpenHelper {
		
		private static final String DB_NAME = "book_provider.db";
		public static final String BOOK_TABLE_NAME = "book";
		public static final String USER_TABLE_NAME = "user";
		
		private static final int DB_VERSION = 1;
		
		//图书列表
		private String CREATE_BOOK_TABLE = "CREATE TABLE IF NOT EXISTS " + BOOK_TABLE_NAME +"(_id INTEGER PRIMARY KEY, name TEXT)";
		//用户列表
		private String CREATE_USER_TABLE = "CREATE TABLE IF NOT EXISTS " + USER_TABLE_NAME +"(_id INTEGER PRIMARY KEY, name TEXT, sex INT)";
	
	
		public BookOpenHelper(Context context) {
			super(context, DB_NAME, null, DB_VERSION);
			// TODO Auto-generated constructor stub
		}
	
		@Override
		public void onCreate(SQLiteDatabase db) {
			// TODO Auto-generated method stub
			db.execSQL(CREATE_BOOK_TABLE);
			db.execSQL(CREATE_USER_TABLE);
	
		}
	
		@Override
		public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
			// TODO Auto-generated method stub
	
		}
	
	}
```

接下来，分别为book表和user表指定Uri，并关联对应的uri——code

```Java
	public class BookOpenHelper extends SQLiteOpenHelper {
		
		private static final String DB_NAME = "book_provider.db";
		public static final String BOOK_TABLE_NAME = "book";
		public static final String USER_TABLE_NAME = "user";
		
		private static final int DB_VERSION = 1;
		
		//图书列表
		private String CREATE_BOOK_TABLE = "CREATE TABLE IF NOT EXISTS " + BOOK_TABLE_NAME +"(_id INTEGER PRIMARY KEY, name TEXT)";
		//用户列表
		private String CREATE_USER_TABLE = "CREATE TABLE IF NOT EXISTS " + USER_TABLE_NAME +"(_id INTEGER PRIMARY KEY, name TEXT, sex INT)";
	
	
		public BookOpenHelper(Context context) {
			super(context, DB_NAME, null, DB_VERSION);
			// TODO Auto-generated constructor stub
		}

		.......
	}
```

接下来我们就可以通过如下方式获取外界要访问点饿数据源，根据Uri取出Uri_code，根据Uri_code得到数据表的名称

```Java
	private String getTableName(Uri uri) {

		String tableName = null;

		switch (sUriMatcher.match(uri)) {

		case BOOK_URI_CODE:

			tableName = BookOpenHelper.BOOK_TABLE_NAME;
			
			break;

		case USER_URI_CODE:
			
			tableName = BookOpenHelper.USER_TABLE_NAME;
			break;

		default:
			break;
		}

		return tableName;
	}
```

接下来，我们就实现query、update、insert、delete方法了。

```Java
	public class BookProvider extends ContentProvider {
	
		public static final String AUTHORITY = "com.ryg.chapter_2.provider.book.provider";
	
		public static final Uri BOOK_CONTENT_URI = Uri.parse("content://"
				+ AUTHORITY + "/book");
		public static final int BOOK_URI_CODE = 0;
	
		public static final Uri USER_CONTENT_URI = Uri.parse("content://"
				+ AUTHORITY + "/user");
		public static final int USER_URI_CODE = 1;
	
		private static final UriMatcher sUriMatcher = new UriMatcher(
				UriMatcher.NO_MATCH);
	
		static {
			// 将Uri和Uri_Code关联起来
			sUriMatcher.addURI(AUTHORITY, "book", 0);
			sUriMatcher.addURI(AUTHORITY, "book", 1);
		}
		
		private Context mContext;
		private SQLiteDatabase db;
	
		@Override
		public boolean onCreate() {
			// TODO Auto-generated method stub
	
			mContext = getContext();
			initData();
			
			return true;
		}
	
		private void initData() {
			// TODO Auto-generated method stub
			db = new BookOpenHelper(mContext).getWritableDatabase();
			db.execSQL("delete from " + BookOpenHelper.BOOK_TABLE_NAME);
			db.execSQL("delete from " + BookOpenHelper.USER_TABLE_NAME);
			db.execSQL("insert into book values(2,'Android');");
			db.execSQL("insert into book values(3,'IOS');");
			db.execSQL("insert into book values(4,'Window Phone');");
			db.execSQL("insert into user values(6,'lee',1);");
			db.execSQL("insert into book values(7,'lau',0);");
			
		}
	
		@Override
		public Cursor query(Uri uri, String[] projection, String selection,
				String[] selectionArgs, String sortOrder) {
			
			String table = getTableName(uri);
			
			if (table == null) {
				throw new IllegalArgumentException("Unsupported URI: " + uri);
			}
			
			return db.query(table, projection, selection, selectionArgs, null, null, sortOrder, null);
		}
	
		@Override
		public String getType(Uri uri) {
			// TODO Auto-generated method stub
			return null;
		}
	
		@Override
		public Uri insert(Uri uri, ContentValues values) {
			// TODO Auto-generated method stub
			String table = getTableName(uri);
			
			if (table == null) {
				throw new IllegalArgumentException("Unsupported URI: " + uri);
			}
			
			db.insert(table, null, values);
			mContext.getContentResolver().notifyChange(uri, null); //通过数据源变化
			
			return uri;
		}
	
		@Override
		public int delete(Uri uri, String selection, String[] selectionArgs) {
			// TODO Auto-generated method stub
			String table = getTableName(uri);
			
			if (table == null) {
				throw new IllegalArgumentException("Unsupported URI: " + uri);
			}
			
			int count = db.delete(table, selection, selectionArgs);
			
			if (count >0) {
				mContext.getContentResolver().notifyChange(uri, null);
			}
			
			return count;
		}
	
		@Override
		public int update(Uri uri, ContentValues values, String selection,
				String[] selectionArgs) {
			// TODO Auto-generated method stub
			
			String table = getTableName(uri);
			
			if (table == null) {
				throw new IllegalArgumentException("Unsupported URI: " + uri);
			}
			
			int row = db.update(table, values, selection, selectionArgs);
			
			if (row > 0) {
				mContext.getContentResolver().notifyChange(uri, null);
			}
			
			return row;
		}
	
		private String getTableName(Uri uri) {
	
			String tableName = null;
	
			switch (sUriMatcher.match(uri)) {
	
			case BOOK_URI_CODE:
	
				tableName = BookOpenHelper.BOOK_TABLE_NAME;
				
				break;
	
			case USER_URI_CODE:
				
				tableName = BookOpenHelper.USER_TABLE_NAME;
				break;
	
			default:
				break;
			}
	
			return tableName;
		}
	
	}
```

访问BookProvider

```Java
	public class BookActivity extends Activity {
		
		private ContentObserver mObserver = new ContentObserver(new Handler()) {
			
			@Override
			public void onChange(boolean selfChange, Uri uri) {
				// TODO Auto-generated method stub
				System.out.println(uri);
				super.onChange(selfChange, uri);
			}
		};
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			
			
			Uri uri = Uri.parse("content://com.ryg.chapter_2.provider.book.provider/book");
			getContentResolver().registerContentObserver(uri, false, mObserver);
			ContentValues values = new ContentValues();
			values.put("_id", 7);
			values.put("name", "Html");
			getContentResolver().insert(uri, values);
			
			
		}
	}
```

需要注意的是，query、update、insert、delete四大方法是存在多线程并发访问的，因此方法内部要做好线程同步本例中，由于采用的是SQLite并且只有一个SQLiteDataBase的连接，所以可以正确应对多线程的情况。具体原因是SQLiteDatabase内部对数据库的操作是有同步处理的，但是如果通过多个SQLiteDatabase对象来操作数据库就无法保证线程同步，因为SQLiteDatabase对象之间无法进行线程同步。如果ContentProvider的底层数据是一块内存的话，比如List，在这种情况下同List的遍历、插入、删除操作就需要进行线程同步，否则就会引起并发错误。

# Socket

Socket又称为套接字，是网络通信的概念，它分为流式套接字和用户数据报套接字，分别对应于网络传输层的TCP和UDP协议。TCP是面向连接的协议，提供稳定的双向通信功能，TCP连接的建立需要经过“三次握手”才能完成，为了提供稳定的数据传输功能，其本身提供了超时重传机制，因此具有很高的稳定性；而UDP是面向无连接的协议，提供不稳定的单向通信功能，当然UDP也可以实现双向通信功能。在性能上，UDP具有更高的效率，其缺点是不能保证数据一定能够正确传输，尤其是在网络拥塞的情况下。

下面示例一个聊天室程序，首先是服务端的设计，当Service启动时，会在线程中建立TCP服务，这里监听的是8688端口，然后就可以等待客户端的连接请求。当有客户端连接时，就会生成一个新的Socket，通过每次新创建的Socket就可以分别和不同的客户端通信了。当客户端断开连接时，服务端这边也会关闭对应Socket并结束通话线程。这点是如何做到的呢？这里是通过判断服务端输入流的返回值来确定的，当客户端断开连接后，服务端这边的输入流会返回null，这个时候我们就知道客户端退出了。服务端代码如下：



# 选用合适的IPC方式

| 名称 | 优点 | 缺点 |适用场景
|----  |---   |---- |------
|Bundle|简单易用|只能传输Bundle支持的数据类型|四大组件间的进程间通信
|文件共享|简单易用|不适合高并发场景，并且无法做到进程间的即时通信|无并发访问情形，交换简单的数据实时性不高的场景
|AIDL|功能强大，支持一对多并发通信，支持实时通信|使用稍复杂，需要处理好现场同步|一对多通信且有RPC需求
|Messenger|功能一般，支持一对多串行通信，支持实时通信|不能很好地处理搞并发情形，不支持RPC，数据通过Messenger进行传输，因此只能传输Bundle支持的数据类型|低并发的一对多即时通信，无RPC需求，或者无需要返回结果的RPC需求
|ContentProvider|在数据源访问方面功能强大，支持一对多并发数据共享，可通过Call方法扩展其他操作|可以理解为受约束的AIDL，主要提供数据源的CRUD操作|一对多的进程间的数据共享
|Socket|功能强大，可通过网络传输字节流，支持一对多并发实时通信|实现细节稍微有些烦琐，不支持直接的RPC|网络数据交换

# 参考文献

Android开发艺术探索

