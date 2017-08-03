**Android IPC机制**

[TOC]

## Android IPC简介

IPC是Inter-Process Communication的缩写，含义为进程间通信，是指两个进程之间进行数据交换的过程。

Android中IPC的使用情况分为两种：

*	第一种情况是一个应用因为某些原因自身需要采用多进程模式来实现，至于原因，可能有很多，比如有些模块由于特殊的原因需要运行在单独的进程中，又或者为了加大一个应用可使用的内存所以需要通过多进行来获取多分内存空间。

*	第二种情况是当前应用需要向其他应用获取数据，由于是两个应用，所以必须采用跨进程的方式来获取所需要的数据，甚至我们系统提供的ContentProvider去查询数据的时候，其实也是一种进程间通信，只不过通信细节被系统内部屏蔽了。


## Android中的多进程模式

### 开启多进行模式

正常情况下，在Android中多进程是指一个应用中存在多个进程的情况，因此这里不讨论两个应用之间的多进程情况。首先，在Android中使用多进程只有一种方法，就是给四大组件在AndroidManifest中指定android:process属性，除此之外没有其他办法。其实还有另一种非常规的多进程方法，那就是通过JNI在native层其fork一个新的进程，这种方法属于特殊情况，也不是常用的创建多进程的方式。下面示例，描述如何在Android中创建多进程

```Java
    <activity
        android:name="com.example.demo.AActivity"
        android:process=":remote" />
    
    <activity
        android:name="com.example.demo.BActivity"
        android:process="com.example.demo.BaseActivity.remote" />
```

上面示例分为为AActivity和BActivity指定了process属性，并且他们的属性值不同，意味着当前应用又增加了两个进程。当AActivity启动时，系统会为它创建一个单独的进程，进程名为"com.example.demo.BaseActivity:remote"；当BActivity启动时，系统也会为它创建一个单独的进程，进程名为"com.example.demo.BaseActivity.remote"。

":remote"和"com.example.demo.BaseActivity.remote"这两种命名方式的区别？
*	首先“:”的含义是指要在当前的进程名前面附加当前的包名，而"com.example.demo.BaseActivity.remote"是完整的命名方式不会附加包名信息

*	其次，进程名以“:”开头的进程属于当前应用的私有进程，其他应用的组件不可以和它跑在同一个进程中，而进程名不易“：”开头的进程属于全局进程，其他应用通过ShareUID方式可以和它跑在同一个进程中。

### 多进程模式的运行机制

Android为你每个应用分配了一个独立的虚拟机，或者说为每个进程都分配一个独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这就会导致在不同的虚拟机中访问同一个类对象会产生多份副本。

所有运行在不同进程中的四大组件，只要它们之间需要通过内存来共享数据，都会共享失败，这也是多线程所带来的主要影响。

一般来说，使用多线程会造成如下四个方面的问题：

*	（1）静态成员和单例模式完全失效

Android为你每个应用分配了一个独立的虚拟机，或者说为每个进程都分配一个独立的虚拟机，不同的虚拟机在内存分配上有不同的地址空间，这就会导致在不同的虚拟机中访问同一个类对象会产生多份副本。

*	（2）线程同步机制完全失效

本质上和第一个问题是类似的，既然都不是一块内存了，那么不管锁对象还是锁全局类都无法保证线程同步，因为不同进程锁的不是同一个对象。

*	（3）SharedPreference的可靠性下降

是因为SharedPreference不支持两个进程同步去执行写操作，否则会导致一定几率的丢失，这是因为SharedPreference底层是通过读/写XML文件实现的，并发写文件显然是可能出问题的，甚至并发读/写都有可能出问题。

*	（4）Application会多次创建

当一个组件跑在一个新的进程中的时候，**由于系统要在创建新的进程同时分配独立的虚拟机，所以这个过程其实就是启动一个新应用的过程**。因此，相当于系统又把这个应用重新启动了一遍，既然重新启动了，那么自然会创建新的Application。

在多进程模式中，不同进程的组件的确会拥有独立的虚拟机、Application和内存空间。

为了解决这个问题，Android系统提供了很多跨进程通信方法实现数据交互。如Intent来传递数据，共享文件和SharedPreference，基于Binder的Messenger和AIDL，Socket等。

## IPC基础概念介绍

### Serializable接口

Serializable是Java所提供的一个序列化接口，它是一个空接口，为对象提供标准的序列化和反序列化操作。使用Serializable来实现序列化相当简单只需要在类的声明中指定一个类似下面的标志即可自动实现默认的序列化过程。

```Java
	private static final long serialVersionUID = 5123020951483359287L; //系统生成的hash值
	private static final long serialVersionUID = 1L; //指定为1L

	public class User implements Serializable {
	
		/**
		 * 
		 */
		private static final long serialVersionUID = 5123020951483359287L; //系统生成的hash值
		
	
		public int 		userId;
		public String 	userName;
		public boolean 	isMale;
		
		@Override
		public String toString() {
			return "User [userId=" + userId + ", userName=" + userName
					+ ", isMale=" + isMale + "]";
		}
	
	}
```

通过Serializable接口来实现对象的序列化过程非常简单，几乎所有的工作都被系统自动完成了。

```Java

	//序列化存储
	User user = new User(2, "liuguoquan", true);
		ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(Environment.getExternalStorageDirectory()+"/cache.txt"));
		out.writeObject(user);
		out.close();

	//反序列化
		ObjectInputStream in = new ObjectInputStream(new FileInputStream(Environment.getExternalStorageDirectory()+"/cache.txt"));
		User newUser = (User) in.readObject();
		in.close();
```

上述代码描述了采用Serializable方式序列化对象的典型过程，很简单，只需要把实现了Serializable接口的User对象写到文件中就可以快速恢复了，恢复后的对象newUser和user的内容完全一样，但是两者并不是同一个对象。

其实，不指定serialVersionUID也可以实现序列化，那到底要不要指定呢？系统既然提供了这个serialVersionUID，那么它必须是有用的，原则上序列化的数据中的serialVersionUID只有和当前类的serialVersionUID相同时才能够正常地被反序列化。

serialVersionUID的详细工作机制是这样的：序列化的时候系统会把当前类的serialVersionUID写入序列化的文件中，当反序列化的时候系统会去检查文件中的serialVersionUID，看它是否和当前类的serialVersionUID一致，如果一致说明序列化的类版本与当前类的版本是相同的则可以成功反序列化；否则就说明当前类和序列化的类相比发生了某些变化，比如成员变量的数量、类型可能发生了改变，这个时候是无法正常反序列化的。

给serialVersionUID指定为1L或者采用系统当前类结构去生成的hash值，这两者并没有什么区别，效果完全一样。以下两点需要注意：

*	静态成员变量属于类不属于对象，所以不会参与序列化过程
*	用transient关键字标记的成员变量不参与序列化过程


### Parcalable接口

Parcelable也是一个也是一个接口，只要实现这个接口，一个类的对象就要就可以实现序列化并可以通过Intent和Binder传递。下面的示例是一个典型的用法。

```Java
	public class Person implements Parcelable {
		
		private int id;
		private String name;
		private int sex;
		private User user;
		
		
	
	
		@Override
		public int describeContents() {
			// TODO Auto-generated method stub
			return 0;
		}
	
		//序列化
		@Override
		public void writeToParcel(Parcel dest, int flags) {
			// TODO Auto-generated method stub
			dest.writeInt(id);
			dest.writeString(name);
			dest.writeInt(sex);
			dest.writeSerializable(user);
			
		}
		
		//反序列化
		public static final Parcelable.Creator<Person> CREATOR = new Parcelable.Creator<Person>() {
	
			@Override
			public Person createFromParcel(Parcel source) {
				// TODO Auto-generated method stub
				
				Person person = new Person();
				//必须要按照成员变量的初始化顺序
				person.id = source.readInt();
				person.name = source.readString();
				person.sex = source.readInt();
				person.user = (User) source.readSerializable();
				return person;
			}
	
			@Override
			public Person[] newArray(int size) {
				// TODO Auto-generated method stub
				return new Person[size];
			}
		};
	
	}
```

系统已经为我们提供了许多实现了Parcelable接口的类，它们逗死可以直接序列化的，比如Intent、Bundle、Bitmap等，同时List和Map也可以序列化，前提是他们里面的每个元素都可以序列化。

既然Parcelable和Serializable都能实现序列化并且都可用于Intent间的数据传递，那么二者该如何选取呢？Serializable是Java中的序列化接口，其使用起来非常简单但是开销很大，序列化和反序列化过程需要大量的I/O操作。而Parcelable是Android中的序列化方式，因此更适合在Android平台上，它的缺点就是使用起来稍微麻烦点，但是它的效率很高，这是Android推荐的序列化方式。因此首选Parcelable。Parcelable主要用在内存序列化上，通过Parcelable将对象序列化到存储设备中或者将对象序列化后通过网络传输也都是可以的，但这个过程显得复杂，因此这两种情况下建议使用Serializable。

Binder是Android中的一个类，它实现了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信方式，Binder还可以理解为一种虚拟的物理设备，它的设备驱动是/dev/binder，该通信方式在Linux中没有；从Android Framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager，等待）和相应的ManagerService的桥梁；从Android应用层来说，Binder是客户端和服务端进行通信的媒介，当bindservice时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或数据，这里的服务包括普通的服务和基于AIDL的服务。

Android开发中，Binder主要用在service中，包括AIDL和Messenger，其中普通Service中的Binder不涉及进程间通信，而Messenger的底层其实是AIDL，所以这里选用AIDL来分析Binder的工作机制。

下面新建一个AIDL示例，新建三个文件Book.java、Book.aidl、IBookManager.aidl，代码如下所示：

Book.java

```Java
	package com.ryg.chapter_2.aidl;
	
	
	import android.os.Parcel;
	import android.os.Parcelable;
	
	public class Book implements Parcelable {
		
		public int bookId;
		public String bookName;
	
		
		
		public Book(int bookId, String bookName) {
			this.bookId = bookId;
			this.bookName = bookName;
		}
	
		@Override
		public int describeContents() {
			// TODO Auto-generated method stub
			return 0;
		}
	
		@Override
		public void writeToParcel(Parcel dest, int flags) {
			// TODO Auto-generated method stub
			dest.writeInt(bookId);
			dest.writeString(bookName);
	
		}
		
		public static final Parcelable.Creator<Book> CREATOR = new Parcelable.Creator<Book>() {
	
			@Override
			public Book createFromParcel(Parcel source) {
				// TODO Auto-generated method stub
				
				return new Book(source);
			}
	
			@Override
			public Book[] newArray(int size) {
				// TODO Auto-generated method stub
				return new Book[size];
			}
		};
		
		
		private Book(Parcel in) {
			
			bookId = in.readInt();
			bookName = in.readString();
		}
	
	}

Book.aidl

	package com.ryg.chapter_2.aidl;
	
	parcelable Book;

IBookManager.aidl

	package com.ryg.chapter_2.aidl;
	
	import com.ryg.chapter_2.aidl.Book;
	
	interface IBookManager {
	
		List<Book> getBookList();
		void addBook(in Book book);
		
	}
```

Book.java是一个图书信息的类，它实现了Parcelable接口。Book.aidl是Book类在AIDL的声明。IBookManager.aidl是我们定义的一个接口，里面有两个方法，其中getBookList用于从远程服务端获取图书列表，而addBook用于向图书列表中添加一本书。尽管Book类已经和IBookManager位于相同的包中，但是IBookManager中仍然要导入Book类，接下来系统会在gen目录自动生成一个IBookManager的类。如下

```java
	/*
	 * This file is auto-generated.  DO NOT MODIFY.
	 * Original file: D:\\liuguoquan\\workspace\\chapter_2\\src\\com\\ryg\\chapter_2\\aidl\\IBookManager.aidl
	 */
	package com.ryg.chapter_2.aidl;
	
	//在Binder传输的接口都要实现IInterface
	public interface IBookManager extends android.os.IInterface {
		/** Local-side IPC implementation stub class. */
		public static abstract class Stub extends android.os.Binder implements
				com.ryg.chapter_2.aidl.IBookManager {
			private static final java.lang.String DESCRIPTOR = "com.ryg.chapter_2.aidl.IBookManager";
	
			//内部类，这个Stub就是一个Binder类
			/** Construct the stub at attach it to the interface. */
			public Stub() {
				this.attachInterface(this, DESCRIPTOR);
			}
	
			/**
			 * Cast an IBinder object into an com.ryg.chapter_2.aidl.IBookManager
			 * interface, generating a proxy if needed.
			 */
			public static com.ryg.chapter_2.aidl.IBookManager asInterface(
					android.os.IBinder obj) {
				if ((obj == null)) {
					return null;
				}
				android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
				if (((iin != null) && (iin instanceof com.ryg.chapter_2.aidl.IBookManager))) {
					return ((com.ryg.chapter_2.aidl.IBookManager) iin);
				}
				return new com.ryg.chapter_2.aidl.IBookManager.Stub.Proxy(obj);
			}
	
			@Override
			public android.os.IBinder asBinder() {
				return this;
			}
	
			@Override
			public boolean onTransact(int code, android.os.Parcel data,
					android.os.Parcel reply, int flags)
					throws android.os.RemoteException {
				switch (code) {
				case INTERFACE_TRANSACTION: {
					reply.writeString(DESCRIPTOR);
					return true;
				}
				case TRANSACTION_getBookList: {  //用于标识方法
					data.enforceInterface(DESCRIPTOR);
					java.util.List<com.ryg.chapter_2.aidl.Book> _result = this
							.getBookList();
					reply.writeNoException();
					reply.writeTypedList(_result);
					return true;
				}
				case TRANSACTION_addBook: {
					data.enforceInterface(DESCRIPTOR);
					com.ryg.chapter_2.aidl.Book _arg0;
					if ((0 != data.readInt())) {
						_arg0 = com.ryg.chapter_2.aidl.Book.CREATOR
								.createFromParcel(data);
					} else {
						_arg0 = null;
					}
					this.addBook(_arg0);
					reply.writeNoException();
					return true;
				}
				}
				return super.onTransact(code, data, reply, flags);
			}
	
			private static class Proxy implements
					com.ryg.chapter_2.aidl.IBookManager {
				private android.os.IBinder mRemote;
	
				Proxy(android.os.IBinder remote) {
					mRemote = remote;
				}
	
				@Override
				public android.os.IBinder asBinder() {
					return mRemote;
				}
	
				public java.lang.String getInterfaceDescriptor() {
					return DESCRIPTOR;
				}
	
				@Override
				public java.util.List<com.ryg.chapter_2.aidl.Book> getBookList()
						throws android.os.RemoteException {
					android.os.Parcel _data = android.os.Parcel.obtain();
					android.os.Parcel _reply = android.os.Parcel.obtain();
					java.util.List<com.ryg.chapter_2.aidl.Book> _result;
					try {
						_data.writeInterfaceToken(DESCRIPTOR);
						mRemote.transact(Stub.TRANSACTION_getBookList, _data,
								_reply, 0);
						_reply.readException();
						_result = _reply
								.createTypedArrayList(com.ryg.chapter_2.aidl.Book.CREATOR);
					} finally {
						_reply.recycle();
						_data.recycle();
					}
					return _result;
				}
	
				@Override
				public void addBook(com.ryg.chapter_2.aidl.Book book)
						throws android.os.RemoteException {
					android.os.Parcel _data = android.os.Parcel.obtain();
					android.os.Parcel _reply = android.os.Parcel.obtain();
					try {
						_data.writeInterfaceToken(DESCRIPTOR);
						if ((book != null)) {
							_data.writeInt(1);
							book.writeToParcel(_data, 0);
						} else {
							_data.writeInt(0);
						}
						mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
						_reply.readException();
					} finally {
						_reply.recycle();
						_data.recycle();
					}
				}
			}
	
			static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0); //标识方法的id
			static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
		}
	
		public java.util.List<com.ryg.chapter_2.aidl.Book> getBookList()
				throws android.os.RemoteException;
	
		public void addBook(com.ryg.chapter_2.aidl.Book book)
				throws android.os.RemoteException;
	}
```

上述代码实现的功能：  
（1）IBookManager类继承IInterface接口，同时它自己也是个接口，所有可以在Binder中传输的接口都需要继承IInterface接口  
（2）首先，它声明了两个方法getBookList和addBook，就是我们自IBookManager.aidl中定义的方法，同时还声明了两个整形id分别用于标识这两个方法，这两个id用于标识在transact中客户端所请求的到底是哪个方法  
（3）接着，它声明了一个内部类Stub，这个Stub就是一个Binder类，当客户端和服务端都位于一个进程时，方法调用不会走跨进程的transact过程，而当两者位于不同进程时，方法调用需要走transact过程，这个逻辑由Stub的内部代理类Proxy来完成。这个接口的核心实现就是**内部了Stub和Stub的内部代理类Proxy**

**说明：**首先，当客户端发起远程请求时，由于当前线程会被挂起直至服务器进程返回数据，所以如果一个远程方法是很耗时的，那么不能在UI线程中发起次远程请求；其次，由于服务器的Binder方法运行在Binder线程池中，所以Binder方法不管是否耗时都应该采用同步的方式去实现，因为它已经运行在一个线程中了。

# 参考文献

Android开发艺术探索

