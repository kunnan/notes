Android性能优化
=======

*	[1. 布局优化](#layout)
*	[2. 内存优化](#memory)
*	[3. 性能分析工具](#tools) 
*	[4. 电量优化](#battery)


<h2 id="layout">1. 布局优化</h2>

系统在渲染UI界面的时候将消耗大量的资源，一个合格的UI不仅应该具有良好的视觉效果，更应该具有良好的使用体验，因此布局优化显得非常重要。

###1.1  Android UI渲染机制

人眼所感觉的流畅画面，需要画面的帧数达到40帧每秒到60帧每秒。在Android中，系统通过VSYNC信号出发对UI的渲染、重绘，其间隔时间是16ms，即1000/16。**如果系统每次渲染的时间都保持在10ms之内，那么我们看见的UI界面将是非常流畅的，但这也需要将所有程序的逻辑都保证在10ms内。**如果不能在16ms之内完成绘制，那么就会造成丢帧现象，即当前该重绘的帧被未完成的逻辑阻塞，例如一次绘制任务需要耗时20ms，那么在10ms系统发出的VSYNC信号时就无法完成，该帧就会被丢弃，等待下次信号开始绘制，导致16*2ms内都显示同一帧画面，这就是画面卡顿的原因。

Androi系统提供了检测UI渲染时间的工具，打开开发者选项，选择“Profile GPU Rendering”，并选择“On screen as bars”的选项，这时候屏幕将显示一些图形。每一条柱线都包含三个部分，**蓝色表示测量绘制DisplayList的时间，红色表示OpenGL渲染DisplayList所需要的时间，黄色表示CPU等待GPU处理的时间，绿色横线表示VSYNC时间16ms**，需要尽量将所有条形图都控制在这条绿线之下。

###1.2 避免Overdraw

Overdraw，过度绘制会浪费很多的CPU、GPU资源，例如系统默认会绘制Activity背景，如果再给布局绘制重叠的背景，那么默认Activity的背景就属于无效的过度绘制--Overdraw。Android系统在开发者选项中提供这样一个检测工具---“Enable GPU Overdraw”。激活后，可以通过界面上的颜色来判断Overdraw的次数，如图所示。

![ALT TEXT](http://static.oschina.net/uploads/img/201503/04080418_o2JX.png)  

蓝色、淡绿、淡红、深红代表4中不同程度的过度绘制，我们的目标就是尽量减少红色区域，增大蓝色区域。

**绘制优化**是指View的onDraw方法要避免执行大量的操作，这主要体现在两个方面：  

首先，onDraw中不要创建新的局部对象，这是因为onDraw方法可能会被频繁使用，这样就会在一瞬间产生大量的临时对象，这不仅占用更多的内存而且还会导致系统更加频繁GC，降低程序的执行效率。  

另一方面，onDraw方法中不要做耗时的任务，也不能执行成千上万的循环操作，尽管每次循环都轻量级，但是大量的循环仍然会抢占CPU的时间片，这会造成View的绘制不流畅。按照Google官方给出的性能优化典范中的标准，View的绘制帧率保证60fps是最佳的，这就要求每帧的绘制时间不超过16ms，虽然程序很难保证10ms这个时间，但是尽量降低onDraw方法的复杂度总是切实有效的。  

###1.3 优化布局层级

在Android中，系统对View的测量、布局和绘制时，都是通过对View数的遍历来进行操作的。如果一个View树的高度太高，就会严重影响测量、布局和绘制的速度，因此优化布局的第一个方法就是降低View树的高度，Google在其API文档中建议View树的高度不宜超过10层。

在早期Android版本中，Google使用LinearLayout作为默认创建的XML文件根布局，而现在的版本中，Google以及使用RelativeLayout来替代LinearLayout最为默认的根布局，**其原因就是通过扁平的RelativeLayout来降低通过LinearLayout嵌套所产生布局树的高度，从而提高UI渲染的效率**。

###1.4 避免嵌套过多无用布局

####1.4.1 使用<include\>标签重用Layout

在一个开发同一风格的应用程序，很多界面都会存在一些共通的UI，比如一个应用的TopBar、Bottombar等。对于这些共用的UI，如果在每个界面都来复制这样一段代码，不仅不利于后期代码的维护，更增加程序的冗余度。因此可以使用<include\>标签来定义一个通用的UI。

共通的UI如下commom_ui.xml  

	<TextView 
		android:layout_width="0dp"
		android:layout_height="0dp"
	</TextView>

在代码中，将layout\_width和layout\_height设置为0dp，这样就迫使开发者在使用是对宽高进行赋值，否则将无法看见这个界面。  

引用这个共通UI

	<LinearLayout>
			android:layout_width="match_parent"
			android:layout_height="match_parent">
	
		<include layout="@layout/common_ui"
			android:layout_alignParentBottom="true"
			android:layout_width="match_parent"
			android:layout_height="wrap_content" />
	
	</LinearLayout> 

**注意：**如果需要在<include\>标签中覆盖类似原布局中android:layout_XXX的属性，就必须在<include\>标签中同时指定android:layout\_width和android:layout\_height属性。

####1.4.2 使用<merge\>标签减少布局的层级  

<merge\>标签一般和<include\>标签一起使用从而减少布局的层级。在上面的示例中，由于当前布局是一个竖直方向的LinearLayout，这个时候如果被包含的布局文件中也采用了竖直方向的LinearLayout，那么显然被包含的布局文件的LinearLayout是多余的，通过<merge\>标签就可以去掉多余的那一层LinearLayout，如下所示 

	<merge xmlns:android="http://schemas.android.com/apk/res/android" >
	
	    <TextView
			android:id="@+id/tv_info"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="@string/hello_world" />

	    <Button
			android:id="@+id/tv_info"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="@string/hello_world" />
	
	</LinearLayout>

####1.4.3 使用<ViewStub\>实现View的延迟加载  

除了把一个View作为共通UI，并通过<include>标签进行引用之外，还可以使用<ViewStub\>标签来实现对一个View的引用并实现延迟加载。<ViewStub\>是一个非常轻量级的组件，它不仅不可视，而且大小为0。下面通过实例来演示如何使用<ViewStub\>

首先创建一个布局，这个布局在初始化加载的时候不需要显示，只在某些情况下才显示出来。例如查看用户信息时，只有点击某个按钮时，用户详细信息才显示出来。not\_often_use.xml  

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    tools:context="com.example.demo.MainActivity" >
	
	    <TextView
			android:id="@+id/tv_info"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="@string/hello_world" />
	
	</LinearLayout>

其次，在主布局中的<ViewStub\>中的layout属性来引用上面的布局

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    tools:context="com.example.demo.MainActivity" >
	
	    <TextView
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="@string/hello_world" />
	    
	    <ViewStub 
	        android:id="@+id/not_often_use"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:layout="@layout/not_often_use"/>
	
	</RelativeLayout>

最后，在程序中提供普通的findViewById()方法找到<ViewStub\>组件  

	mViewStub = (ViewStub) findViewById(R.id.not_often_use);

接下来，有两种方式来重新显示这个View。

*	VISIBLE

		mViewStub.setVisibility(View.VISIBLE);

*	inflate 

		View mView = mViewStub.inflate(); //两次调用会报错

这两种方式都可以让ViewStub重新展开，显示引用的布局，而唯一的区别就是inflate()方法可以返回引用的布局，从而可以再通过View.findViewById()方法来找到对应的控件

	View mView = mViewStub.inflate();
	TextView textView = (TextView)mView.findViewById(R.id.tv_info);
	textView.setText("HaHa");

**而不管使用哪种方式，一旦<ViewStub\>被设置为可见或是被inflate了，<ViewStub\>就不存在了，取而代之的是被inflate的Layout，并将这个Layout的ID重新设置为<ViewStub\>中通过android:inflateID属性所指定的ID，这也是为什么两次调用inflate方法会报错的原因。**

<ViewStub\>标签与设置View.Gone这种方式来隐藏一个View有什么区别了？  
的确，它们的共同点都是初始化时不会显示，但<ViewStub\>标签只会在显示时才去渲染整个布局，而View.GONE，在初始化布局的时候就已经添加在布局树上了，相比之下<ViewStub\>标签的布局具有更高的效率。


<h2 id="memory">2. 内存优化</h2>

###2.1 什么是内存

由于Android应用的沙箱机制，每个应用分配的内存大小是有限度的，内存太低就会触发LMK-Lower Memory Killer机制。一个Android应用占用的内存可通过如下方式获取：

	//应用所占用的堆内存
	long totalMemory = Runtime.getRuntime().totalMemory();
	//剩余内存，在堆内存不扩展的情况下
	long freeMemory = Runtime.getRuntime().freeMemory();
	//最大可用堆内存
	long maxMemory = Runtime.getRuntime().maxMemory();

那么到底什么是内存呢？通常情况下我们所说的内存是指手机的RAM，它包含以下几个部分。  

*	寄存器(Registers)

速度最快的存储位置，因为寄存器位于处理器内部，在Android应用中无法控制。

*	栈（Stack）

存放基本类型的数据和对象的引用，但对象本身不存放在栈中，而是存放在堆中。

*	堆（Heap）

堆内存用来存放由new创建的对象和数组。在堆中分配的内存，由Java虚拟机的自动垃圾回收器（GC）来管理。

*	静态存储区域（Static Field）

静态存储区就是指在固定的位置存放应用程序运行时一直存在的数据，Java在内存中专门划分了一个静态存储区域来管理一些特殊的数据变量如静态的数据变量。

*	常量池（Constant Pool）

JVM虚拟机必须为每个被装载的类型维护一个常量池。常量池就是该类型所用到常量的一个有序集合，包括直接常量（基本类型，String）和对其他类型、字段和方法的引用。

	public class Test{  
	  
	Integer i1=new Integer(1);  
	   Integer i2=new Integer(1);  
	//i1,i2分别位于堆中不同的内存空间   
	  
	   System.out.println(i1==i2);//输出false   
	  
	  
	   Integer i3=1;  
	   Integer i4=1;  
	//i3,i4指向常量池中同一个内存空间   
	  
	   System.out.println(i3==i4);//输出true   
	  
	//很显然，i1,i3位于不同的内存空间   
	  
	System.out.println(i1==i3);//输出false   
	  
	}  
	
	public class Test{  
	  
		public static void main(String[] args){  
		  
		//s1,s2分别位于堆中不同空间   
		  
		String s1=new String("hello");  
		  
		String s2=new String("hello");  
		  
		System.out.println(s1==s2)//输出false   
		  
		//s3,s4位于池中同一空间   
		  
		String s3="hello";  
		  
		String s4="hello";  
		  
		System.out.println(s3==s4);//输出true   
		  
		}  
	}  

>在这些概念中最容易混淆的就是堆和栈的区别。当定义一个变量，Java虚拟机就会在栈中为该变量分配内存空间，当该变量作用域结束后，这部分内存空间会立即被用作新的空间进行分配。如果使用new的方式创建一个变量，那么就会在堆中为这个对象分配内存空间，即使该对象的作用域结束，这部分内存也不会立即被回收，而是等待系统GC进行回收。堆的大小随着手机硬件的升级而不断变大。所谓内存分析，正是分析Heap中的内存状态。 

###2.2 获取Android内存信息

*	Process Stat

Process Stat是KK上新增的一个系统内存监视服务，可以通过设置-》开发者-》进程统计信息 打开其该功能界面。同样也可以使用Dumpsys命令获取这些信息`adb shell dumpsys procstats`

*	Meminfo

Meminfo也是系统上一个非常重要的内存监视工具，可以通过设置-》应用-》正在运行 打开打开其该功能界面。同样也可以使用Dumpsys命令获取这些信息`adb shell dumpsys meminfo`

###2.3 内存泄露优化

**内存回收**，Java对于C\C++这类语言的最大优势就是不用手动管理系统资源，Java创建了垃圾收集器线程（Garbage Collection Thread）来自动进行资源的管理。这样做的好处是大大降低了程序开发人员对内存管理的繁琐工作。但也带来了诸多问题，例如Java的GC是系统自动进行的，但何时进行却是开发者无法控制的，即使调用System.gc()方法，也只是建议系统进行GC，至于系统是否采纳你的建议，那就不一定了。JVM虚拟机虽然能够自动控制GC，但是再强大的算法，也难免会存在部分对象忘记回收的现象发生，这就是造成内存泄露的原因。

**内存泄露**的优化可分为两个方面，（1）一方面是在开发过程中避免写出有内存泄露的代码（2）另一方面是通过一些分析工具比如MAT来找出潜在的内存泄露继而解决。
下面介绍一些常用的内存泄露示例：

*	**场景1：静态变量导致的内存泄露**

下面的代码将导致Activity无法正常销毁，因为静态变量sContext引用了它

	public class BaseActivity extends Activity {
		
		private static Context sContext;
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			sContext = this; //Activity被sContext持有无法销毁
		}
	}

上面的代码也可以改造一下，如下所示，sView是一个静态变量，它内部持有了当前Activity，所以Activity仍然无法释放  

	public class BaseActivity extends Activity {
		
		private static View sView;
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			sView = new View(this);//内部持有Activity无法销毁
		}
	}

*	**场景2：单例模式导致的内存泄露**

单例模式所带来的内心泄露是我们容易忽视的，如下所示，首先提供一个单例模式的TestManager，TestManager可以接收外部的注册并将外部的监听器存储起来。

	public class TestManager {
	
		private List<OnDataArrivedListener> mOnDataArrivedListeners = new ArrayList<TestManager.OnDataArrivedListener>();
		
		private static class SingletonHolder {
			
			public static final TestManager INSTANCE = new TestManager();
		}
		
		private TestManager() {
			
		}
		
		public static TestManager getInstance() {
			return SingletonHolder.INSTANCE;
		}
		
		public synchronized void registerListener(OnDataArrivedListener listener) {
			
			if (!mOnDataArrivedListeners.contains(listener)) {
				mOnDataArrivedListeners.add(listener);
			}
		}
		
		public synchronized void unregisterListener(OnDataArrivedListener listener) {
			
			mOnDataArrivedListeners.remove(listener);
		}
		
		public interface OnDataArrivedListener {
			
			public void onDataArrived(Object data);
		}
	}

接着再让Activity实现OnDataArrivedListener接口并向TestManager注册监听，如下所示。下面的代码由于缺少解注册所以会引起内存泄露，泄露的原因是Activity的对象被单例模式的TestManager所持有，而单例模式的特点是**其生命周期和Application保持一致，因此Activity对象无法被及时释放**

	public class BaseActivity extends Activity implements OnDataArrivedListener {
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			// TODO Auto-generated method stub
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			TestManager.getInstance().registerListener(this); //若没有解注册，Activity被持有无法释放
		}
	
		@Override
		public void onDataArrived(Object data) {
			// TODO Auto-generated method stub
			
		}
	}

*	**场景3：属性动画导致的内存泄露**

从Android3.0开始，Google提供了属性动画，属性动画中有一类无限循环的动画，如果在Activity中播放此类动画没有在OnDestroy中去停止动画，那么动画会一直播放下去，尽管已经无法在界面上看到动画效果了，并且这个时候Activity的View即mButton仍被动画持有，而View又持有了Activity，最终Activity无法释放。解决方法是在Activity的OnDestroy中调用animator.cancel()来停止动画。

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		mButton = (Button)findViewById(R.id.button);
		ObjectAnimator animator = ObjectAnimator.ofFloat(mButton, "rotation", 0,360).setDuration(2000);
		animator.setRepeatCount(ValueAnimator.INFINITE);//无线循环
		animator.start();
		//animator.cancel(); //必须在OnDestroy中停止动画
		
	}

###2.4 响应速度优化和ANR日志分析

响应速度优化的核心思想是避免在主线程中做耗时操作，但是有时候的确有很多耗时操作，怎么办呢？可以将这些耗时操作放到线程中执行，即采用异步的方式执行耗时操作。响应速度过慢更多体现在Activity的启动速度上面，如果在主线程中做太多事情，会导致Activity启动时出现黑屏现象，甚至出现ANR。**Android规定，Activity如果5s之内无法响应屏幕触摸事件或者键盘输入事件就会出现ANR，而BroadcastReceiver如果10s内还未执行完操作也会出现ANR**。

在实际开发中，ANR是很难从代码上发现的，如果开发过程中遇到了ANR，那么怎么定位问题呢？其实当一个进程发送了ANR以后，系统会在/data/anr目录下创建一个文件trace.txt，通过分析这个文件就能定位出ANR的原因。


###2.5 内存优化实例

####2.5.1 Bitmap优化

Bitmap是造成内存占用过高甚至是OOM的最大威胁。下面是一些使用Bitmap的小技巧。

*	**使用适当分辨率和大小的图片**

由于Android系统在做资源适配的时候会对不同分辨率文件夹的图片进行缩放来适配相应的分辨率，如果图片分辨率与资源文件夹分辨率不匹配或者图片分辨率太高，就会导致系统消耗更多的内存资源。同时，在适当的时候应该显示合适大小的图片，例如图片列表界面可以使用图片的缩略图，而在详细使用图片的时候再显示原图；或者在对图片要求不高的地方，尽量降低图片的精度。  

*	**及时回收内存**

一旦使用完Bitmap后，一定要及时使用bitmap.recycle()方法释放内存资源。自Android3.0之后，由于Bitmap被放置到了堆中，其内存由GC管理，就不需要手动进行释放了。

*	**使用图片缓存**

通过内存缓存（LruCache）和磁盘缓存（DiskLruCache）可以更好的使用Bitmap。 

###2.5.2 ListView优化

ListView优化主要分为三个方面：

*	首先，采用ViewHolder并避免在getView方法中执行耗时操作
*	其次，要根据列表的滑动状态来控制任务的执行频率，比如当列表快速滑动时不太开启大量的异步任务
*	最后，可以尝试开启硬件加速来使ListView的滑动更加流畅

注意ListView的优化策略完全适应于GridView

###2.6 线程优化

线程优化的思想是采用线程池，避免程序中存在大量的Thread，线程池可以重用内部的线程，从而避免了线程的创建和销毁所带来的性能开销，同时线程池还能有效地控制线程池的最大并发数，避免大量的线程因互相抢占系统资源从而导致阻塞现象的发生。因此在实际开发中，我们要尽量采用线程池，而不是每次都要创建一个Thread对象。

###2.7 代码优化

*	避免创建过多的对象，任何Java类，都将占用500字节的内存空间，创建一个类的实例会消耗大约15字节的内存
*	不要过多使用枚举，枚举占用的内存空间要比整型大，少用迭代器
*	常量请使用static final来修饰
*	使用一些Android特有的数据结构，比如SparseArray和Pair等，它们都具有更好的性能
*	适当使用软引用和弱引用
*	采用内存缓存和磁盘缓存
*	尽量采用静态内部类，这样可以避免潜在的由于内部类而导致的内存泄露
*	使用静态方法，静态方法会比普通方法提高15%左右的访问速度
*	对Cursor、Receiver、Sensor、File等对象，要非常注意对它们的创建、回收与注册、注销
*	避免使用IOC框架，IOC框架通常使用注解、反射来进行实现，虽然现在Java对反射的效率已经进行了很好的优化，但大量使用反射依然会带来性能的下降
*	使用RenderScript、OpenGL来进行非常复杂的绘图操作
*	使用SurfaceView来替代View进行大量、频繁的绘图操作
*	尽量使用视图缓存，而不是每次都执行inflate()方法解析视图

<h2 id="tools">3. 性能分析工具</h2>

###3.1 Lint工具


###3.2 Android studio的Memory Monitor工具


###3.3 TraceView工具


###3.4 MAT工具

###3.5 Dumpsys命令分析系统状态

<h2 id="battery">4. 电量优化</h2>

量其实是目前手持设备最宝贵的资源之一，大多数设备都需要不断的充电来维持继续使用。不幸的是，对于开发者来说，电量优化是他们最后才会考虑的的事情。但是可以确定的是，千万不能让你的应用成为消耗电量的大户。

Purdue University研究了最受欢迎的一些应用的电量消耗，平均只有30%左右的电量是被程序最核心的方法例如绘制图片，摆放布局等等所使用掉的，剩下的 70%左右的电量是被上报数据，检查位置信息，定时检索后台广告信息所使用掉的。如何平衡这两者的电量消耗，就显得非常重要了。

有下面一些措施能够显著减少电量的消耗：

*	我们应该尽量减少唤醒屏幕的次数与持续的时间，使用WakeLock来处理唤醒的问题，能够正确执行唤醒操作并根据设定及时关闭操作进入睡眠状态。
*	某些非必须马上执行的操作，例如上传歌曲，图片处理等，可以等到设备处于充电状态或者电量充足的时候才进行。
*	触发网络请求的操作，每次都会保持无线信号持续一段时间，我们可以把零散的网络请求打包进行一次操作，避免过多的无线信号引起的电量消耗

在09年Google IO大会Jeffrey Sharkey的演讲（Coding for Life — Battery Life, That Is）中就探讨了这个问题，指出android应用的耗电主要在以下三个方面：

*	大数据量的传输。
*	不停的在网络间切换。
*	解析大量的文本数据。

并提出了相关的优化建议： 
 
*	在需要网络连接的程序中，首先检查网络连接是否正常，如果没有网络连接，那么就不需要执行相应的程序。
*	使用效率高的数据格式和解析方法，推荐使用JSON。
*	在进行大数据量下载时，尽量使用GZIP方式下载。
*	其它：回收java对象，特别是较大的java对像，使用reset方法；对定位要求不是太高的话尽量不要使用GPS定位，可能使用wifi和移动网络cell定位即可,**GPS定位消耗的电量远远高于移动网络定位**；尽量不要使用浮点运算；获取屏幕尺寸等信息可以使用缓存技术，不需要进行多次请求；使用AlarmManager来定时启动服务替代使用sleep方式的定时任务。