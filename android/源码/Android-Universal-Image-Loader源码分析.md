Android-Universal-Image-Loader源码分析
==

# 功能介绍

## Android-Universal-Image-Loader

Android-Universal-Image-Loader是一个强大的、可高度定制的开源图片缓存框架，简称UIL。简单的说UIL就做了一件事--获取图片并显示在相应的控件上。

## 基本使用

### 初始化

添加完依赖后再Application或Activity中初始化ImageLoader，一般式在Application中初始化，如下：

```Java
public class UILApplication extends Application {


	@Override
	public void onCreate() {


		super.onCreate();

		initImageLoader(getApplicationContext());
	}

	public static void initImageLoader(Context context) {

		//缓存目录
		File cacheDir = StorageUtils.getCacheDirectory(context);
		//添加配置需求
		ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(context)
		  .memoryCacheExtraOptions(480, 800) // default = device screen dimensions
		  .diskCacheExtraOptions(480, 800, CompressFormat.JPEG, 75, null)
		  .taskExecutor(...)
		  .taskExecutorForCachedImages(...)
		  .threadPoolSize(3) // default 线程池大小
		  .threadPriority(Thread.NORM_PRIORITY - 1) // default 线程优先级
		  .tasksProcessingOrder(QueueProcessingType.FIFO) // default 任务队列模式
		  .denyCacheImageMultipleSizesInMemory()
		  .memoryCache(new LruMemoryCache(2 * 1024 * 1024)) //
		  .memoryCacheSize(2 * 1024 * 1024)
		  .memoryCacheSizePercentage(13) // default
		  .diskCache(new UnlimitedDiscCache(cacheDir)) // default
		  .diskCacheSize(50 * 1024 * 1024)
		  .diskCacheFileCount(100)
		  .diskCacheFileNameGenerator(new HashCodeFileNameGenerator()) // default
		  .imageDownloader(new BaseImageDownloader(context)) // default
		  .imageDecoder(new BaseImageDecoder()) // default
		  .defaultDisplayImageOptions(DisplayImageOptions.createSimple()) // default
		  .writeDebugLogs()
		  .build();

		// 初始化ImageLoader配置
		ImageLoader.getInstance().init(config);
	}
}
```
ImageLoaderConfiguration表示ImageLoader的配置信息，可包括图片最大尺寸、线程池、下载器、缓存等参数的配置。

### Manifest配置

```Java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

<application
	android:name=".UILApplication"
	``
	``
	``
/application>
```
添加网络权限和添加读写外部存储权限

### 下载显示图片

下载图片，解析为Bitmap并在ImageView中显示。

```Java
	private ImageLoadingListener animateFirstListener = new AnimateFirstDisplayListener();

	private DisplayImageOptions options;

	ImageAdapter(Context context) {
			inflater = LayoutInflater.from(context);
			
			//下载图片选项
	options = new DisplayImageOptions.Builder()
			.showImageOnLoading(R.drawable.ic_stub) //下载中
			.showImageForEmptyUri(R.drawable.ic_empty) //空URL
			.showImageOnFail(R.drawable.ic_error) //失败
			.cacheInMemory(true)
			.cacheOnDisk(true)
			.considerExifParams(true)
			.displayer(new CircleBitmapDisplayer(Color.WHITE, 5))
			.build();
	//显示图片
	ImageLoader.getInstance().displayImage(imageUrl, holder.image, options, animateFirstListener);	
	
	/**
	*监听下载图片，传递Bitmap给回调接口
	*/
	private static class AnimateFirstDisplayListener extends SimpleImageLoadingListener {

		static final List<String> displayedImages = Collections.synchronizedList(new LinkedList<String>());

		@Override
		public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
			if (loadedImage != null) {
				ImageView imageView = (ImageView) view;
				boolean firstDisplay = !displayedImages.contains(imageUri);
				if (firstDisplay) {
					FadeInBitmapDisplayer.animate(imageView, 500);
					displayedImages.add(imageUri);
				}
			}
		}
	}
```
## 特点

-	可配置度高。支持任务线程池、下载器、解码器、内存及磁盘缓存、显示选项等的配置。
-	包含内存缓存和磁盘缓存两级缓存。
-	支持多线程，支持异步和同步加载
-	支持多种缓存算法、下载进度监听、ListView图片错乱解决等。

# 总体设计

## 总体设计框图

![UIL框架图](http://7xs7a3.com1.z0.glb.clouddn.com/uil-overall-design.png)

上面是UIL的总体框架图。整个库主要分为ImageLoader、ImageLoaderEngine、Cache、ImageDownloader、ImageDecoder、BitmapDisplayer、BitmapProcessor七大模块，其中Cache分为Memory Cache和DiskCache两部分。

简单的讲就是ImageLoader收到加载及显示图片的任务，并将它交给ImageLoaderEngine，ImageLoaderEngine分发任务到具体线程池去执行，任务通过Cache及ImageDownloader获取图片，中间可能经过BitmapProcessor和ImageDecoder处理，最终转换为Bitmap交给BitmapDisplayer在ImageAware中显示。

## UIL中的概念

-	**ImageLoaderEngine**

任务分发器，负责分发LoadAndDisplayImageTask和ProcessAndDisplayImageTask给具体的线程去执行。

-	**LoadAndDisplayImageTask**

用于加载并显示图片的任务。

-	**ProcessAndDisplayImageTask**

用于处理并显示图片的任务。

-	**DisplayBitmapTask**

用于显示图片的任务

-	**ImageAware**

显示图片的对象，可以是ImageView等。

-	**BitmapDisplayer**

将Bitmap对象显示在相应的控件ImageAware上。

-	**ImageDownloader**

图片下载器，负责从图片的各个来源获取输入流

-	**MemoryCache**

内存图片缓存，可向内存缓存图片或从内存读取图片

-	**DiskCache**

本地图片缓存，可向本地磁盘缓存保存图片或从本地磁盘读取图片

-	**ImageDecoder**

图片解码器，负责将图片输入流InputStream转换为Bitmap对象。

-	**BitmapProcessor**

图片处理器，负责从缓存读取或写入前对图片进行处理

## 流程图

![图片加载及显示流程图](http://7xs7a3.com1.z0.glb.clouddn.com/uil-flow.png)

上图为图片加载及显示流程图,可知有三种情况:

- [ 1 ] - 图片没有缓存

先下载图片，然后显示图片，同时异步将图片缓存到磁盘和内存中。

- [ 2 ] - 图片缓存在磁盘上

若图片不在缓存中，则从磁盘缓存中查找图片，然后将图片解码为Bitmap对象并显示在控件上。

- [ 3 ] - Bitmap对象缓存在内存里

直接从内存缓存取出相应的Bitmap对象并显示在控件上。

# 详细设计

## 类关系图

![类关系图](http://7xs7a3.com1.z0.glb.clouddn.com/uil-relation-class.png)

## 核心类介绍

### ImageLoader.java

图片加载器，对外的主要API，采用了**单例设计模式**，用于图片的加载和显示。

主要函数：

-	getInstance()

得到ImageLoader单例，通过双层是否为null判断提高性能。

-	init(ImageLoaderConfiguration configuration)

初始化配置参数，参数为configuration为ImageLoader的配置信息，包括图片最大尺寸、任务线程池、磁盘缓存、下载器、解码器等等。实现中会初始化ImageLoaderEngine engine属性，该属性为任务分发器。

-	displayImage(String uri, ImageAware imageAware, DisplayImageOptions options, ImageLoadingListener listener, ImageLoadingProgressListener progressListener)

加载并显示图片或加载并执行回调接口。ImageLoader加载图片主要分为三类接口：

1. displayImage()表示异步加载并显示图片到对应的ImageAware上。
2. loadImage()表示异步加载图片并执行回调接口。
3. loadImageSync()表示同步加载图片。

以上三类接口最终都会调用到这个函数进行图片加载。函数参数解释如下：

Uri: 图片的uri，uri支持多种来源的图片，包括http、https、file、content、assets、drawable及自定义
ImageAware： 一个接口，表示需要加载图片的对象，可包装View。
Options： 图片显示的配置项。比如加载前、加载中、加载失败应该显示的占位图片，图片是否需要在磁盘缓存，是否需要在内存缓存等。
listener： 图片加载各种时刻的回调接口，包括开始加载、加载失败、加载成功、取消加载四个时刻的回调函数。
progressListener： 图片加载进度的回调接口。

函数流程图如下：

![](http://7xs7a3.com1.z0.glb.clouddn.com/uil-display-image-flow-chart.png)


###  ImageLoaderConfiguration.java

ImageLoader的配置信息，包括图片最大尺寸、线程池、缓存、下载器、解码器等等。

主要属性：

-	Resources resources 程序本地资源访问器，用于加载DisplayImageOptions中设置的一些App中图片资源。

-	int maxImageWidthForMemoryCache 内存缓存的图片最大宽度。

-	int maxImageHeightForMemoryCache 内存缓存的图片最大高度。

-	int maxImageWidthForDiskCache 磁盘缓存的图片最大宽度。

-	int maxImageHeightForDiskCache 磁盘缓存的图片最大高度。

-	BitmapProcessor processorForDiskCache 图片处理器，用于处理从磁盘缓存中读取到的图片。

-	Executor taskExecutor  ImageLoaderEngine中用于执行从源获取图片的任务

-	Executor taskExecutorForCachedImages ImageLoaderEngine中用于执行从缓存获取图片任务的Executor。

-	boolean customExecutor 用户是否自定义了上面的taskExecutor

-	boolean customExecutorForCachedImages 用户是否自定义了上面的taskExecutorForCachedImages

-	int threadPoolSize 上面两个默认线程池的核心池大小，即最大并发数

-	int threadPriority 上面默认线程池的线程优先级。

-	QueueProcessingType tasksProcessingType 上面两个默认线程池的线程队列类型，目前只有FIFO，LIFO两种选择

-	MemoryCache MemoryCache 图片内存缓存

-	DiskCache diskCache 图片磁盘缓存，一般放在SD卡

-	ImageDownloader downloader 图片下载器

-	ImageDecoder decoder 图片解码器，内部可使用我们常见的BitmapFactory.decode()将图片资源解码成Bitmap对象

-	DisplayImageOptions defaultDisplayImageOptions 图片显示的配置项。比如加载前、加载中、加载失败应该显示的占位图片，图片是否需要在磁盘缓存，是否需要在内存缓存等。

-	ImageDownloader networkDeniedDownloader 不允许访问网络的图片下载器

-	ImageDownloader slowNetworkDownloader 慢网络情况下的图片下载器

### ImageLoaderConfiguration.Builder.java 静态内部类

**Builder模式**，用于构造参数繁多的ImageLoaderConfiguration。其属性与ImageLoaderConfiguration类似，函数多是属性设置函数。

主要函数及含义：

-	builder()

按照配置，生成ImageLoaderConfiguration。代码如下：

```Java
public ImageLoaderConfiguration build() {
	initEmptyFieldsWithDefaultValues();
	return new ImageLoaderConfiguration(this);
}
```
-	initEmptyFieldsWithDefaultValues()

初始化值为null的属性。若用户没有配置相关项，UIL会通过调用DefaultConfigurationFactory中的函数返回一个默认值当配置。

taskExecutorForCacheImages、taskExecutor及ImageLoaderEngine的taskDistributor的默认值如下：

| parameters | taskDistributor | taskExecutorForCacheImages/taskExecutor
|--------|--------|
|   corePoolSize     |   0     | 3
|   maximumPoolSize     |   Integer.MAX_VALUE     | 3
|   keepAliveTime     |   60     | 3
|   unit     |   SECONDS     | MILLISECONDS
|   workQueue     |   SynchronousQueue     | LIFOLinkedBlockingDequeu/LinkedBlockingQueue
|   priority     |   5     | 3

diskCacheFileNameGenerator默认值为HashCodeFileNameGenerator

memoryCache默认值为LruMemoryCache。如果内存缓存不允许缓存一张图片的多个尺寸，则用FuzzyKeyMemoryCache做封装，同一个图片新的尺寸会覆盖缓存中该图片老的尺寸。

diskCache默认值与diskCacheSize和diskCacheFileCount值有关，如果他们有一个大于0，则默认为LruDiskCache，否则使用无大小限制的UnlimitedDiskCache。

downloader默认值为BaseImageDownloader。

decoder默认值为BaseImageDecoder

defaultDisplayImageOptions为Builder().build()

-	denyCacheImageMultipleSizeInMemory()

设置内存缓存不允许缓存一张图片的多个尺寸，默认允许。后面会讲到的View的getWidth()在初始化前后的不同值与这个设置的关系。

-	diskCacheSize(int maxCacheSize)

设置磁盘缓存的最大字节数，如果大于0或者下面的maxFileCount大于0，默认的DiskCache会用LruDiskCache，否则使用无大小限制的UnlimitedDiskCache。

-	diskCacheFileCount(int maxFileCount)

设置磁盘缓存的最大文件数，如果大于0或者上面的maxCacheSize大于0，默认的DiskCache会用LruDiskCache，否则使用无大小限制的UnlimitedDiskCache。

### ImageLoaderConfiguration.NetworkDeniedDownloader.java 静态内部类

不允许访问网络的图片下载器，实现了ImageDownloader接口。实现也比较简单，包装了一个ImageDownloader对象，通过getStream()函数中禁止Http和Https Scheme禁止网络访问，如下：

```Java
private static class NetworkDeniedImageDownloader implements ImageDownloader {

	private final ImageDownloader wrappedDownloader;

	public NetworkDeniedImageDownloader(ImageDownloader wrappedDownloader) {
		this.wrappedDownloader = wrappedDownloader;
	}

	@Override
	public InputStream getStream(String imageUri, Object extra) throws IOException {
		switch (Scheme.ofUri(imageUri)) {
			case HTTP:
			case HTTPS:
				throw new IllegalStateException();
			default:
				return wrappedDownloader.getStream(imageUri, extra);
		}
	}
}
```

### ImageLoaderConfiguration.SlowNetworkingImageDownloader.java 静态内部类

慢网络情况下的图片下载器，实现了ImageDownloader接口。
通过包装一个ImageDownloader对象实现，在getStream()函数中当Scheme为Http和Https时，用FlushedInputStream代替InputStream处理慢网络情况。

```Java
private static class SlowNetworkImageDownloader implements ImageDownloader {

	private final ImageDownloader wrappedDownloader;

	public SlowNetworkImageDownloader(ImageDownloader wrappedDownloader) {
		this.wrappedDownloader = wrappedDownloader;
	}

	@Override
	public InputStream getStream(String imageUri, Object extra) throws IOException {
		InputStream imageStream = wrappedDownloader.getStream(imageUri, extra);
		switch (Scheme.ofUri(imageUri)) {
			case HTTP:
			case HTTPS:
				return new FlushedInputStream(imageStream);
			default:
				return imageStream;
		}
	}
}
```

### ImageLoaderEngine.java

LoadAndDisplayImageTask和ProcessAndDisplayImageTask任务分发器，负责分发任务给具体的线程池。

主要属性：

-	ImageLoaderConfiguration configuration

ImageLoader的配置信息，可包括图片最大尺寸、线程池、缓存、下载器、解码器等等。

-	Executor taskExecutor

用于执行从源执行获取图片任务的Executor，为configuration中的taskExecutor，如果为null，则会调用DefaultConfigurationFactory.createExecutor()根据配置返回一个默认的线程池。

-	Executor taskExecutorForCachedImages

用于执行从缓存获取图片任务的Executor，为configuration中的taskExecutorForCachedImages，如果为null，则会调用DefaultConfigurationFactory.createExecutor()根据配置返回一个默认的线程池。

-	Executor taskDistributor

任务分发线程池，任务指LoadAndDisplayImageTask和ProcessAndDisplayImageTask，因为需要分发给上面的两个Executor去执行任务，不存在较耗时或阻塞操作，所以无并发数（Int最大值）限制的线程池即可。

-	Map cacheKeysForImageAwares

ImageAware与内存缓存key对应的map，key为ImageAware的id，value为内存缓存的key。

-	Map uriLocks

图片正在加锁的重入锁map，key为图片的uri，value为标识其正在加载的重入锁。

-	AtomicBoolean pause

是否被暂停。如果为true，则所有新的加载或显示任务都会等待直到取消暂停（为false）

-	AtomicBooleannetWorkDenied

是否不允许访问网络，如果为true，通过回调ImageLoadingListener.onLoadingFailed()获取图片，则所有不在缓存中需要网络访问的请求都会失败，返回失败的原因为：网络访问被禁止。

-	AtomicBoolean slowNetwork

是否是慢网络情况，如果未true，则自动调用SlowNetworkImageDownloader下载图片

-	Object pauseLock

暂停的等待锁，可在engine被暂停后调用这个锁等待

主要函数：

-	void submit(final LoadAndDisplayImageTask task)

传入一个LoadAndDisplayImageTask，直接用taskDistributor执行一个Runnable，在Runnable内部根据图片是否被磁盘缓存过确定使用taskExecutorForCachedImages还是taskExecutor执行该task。

-	void submit(ProcessAndDisplayImageTask task)

传入一个ProcessAndDisplayImageTask，直接用taskExecutorForCachedImages执行该task，从缓存中去图片。

-	void pause()

暂停图片加载任务，所有新的加载或显示任务都会等待直到取消暂停为止。

-	void resume()

继续图片加载任务

-	stop()

暂停所有加载和显示图片任务并清除这里的内部属性值。

-	fireCallBack(Runnable r)

taskDistributor立即执行某个任务

-	getLockForUri(String uri)

得到某个uri的重入锁，如果不存在则新建一个

-	private Executor createTaskExecutor()

调用DefaultConfigurationFactor.createExecutor()创建一个线程池

-	String getLoadingUriForView(ImageAware imageAware)

得到某个ImageAware正在加载的图片uri

-	prepareDisplayTaskFor(ImageAware imageAware, String memoryCacheKey)

准备开始一个Task。向cacheKeysForImageAwares中插入ImageAware的id和图片在内存缓存中的key

-	void cancelDisplayTaskFor(ImageAware imageAware)

取消一个显示任务。从cacheKeysForImageAwares中删除ImageAware对应元素

-	void denyNetworkDownloads(boolean denyNetworkDownloads)

设置是否不允许网络访问

-	void handleSlowNetwork(boolean handleSlowNetwork)

设置是否慢网络情况

### DefaultConfigurationFactory.java

为ImageLoaderConfiguration及ImageLoaderEngine提供一些默认配置

主要函数：

-	Executor createExecutor(int threadPoolSize, int threadPriority,QueueProcessingType tasksProcessingType)

创建线程池：
threadPoolSize表示核心线程池大小（最大并发数）
threadPriority表示线程优先级
tasksProcessingType表示线程队列类型，目前只有FIFO，LIFO两种可选择

内部实现会调用createThreadFactory(…)返回一个支持线程优先级设置，并且以固定规则命名新建的线程的线程工厂类DefaultConfigurationFactory.DefaultThreadFactory。

-	Executor createTaskDistributor()

为ImageLoaderEngine中的任务分发器taskDistributor提供线程池，该线程池为normal优先级的无并发大小限制的线程池。

-	FileNameGenerator createFileNameGenerator()

返回一个HashCodeFileNameGenerator对象，即以uri HashCode为文件名的文件名生成器。

-	DiskCache createDiskCache(Context context, FileNameGenerator diskCacheFileNameGenerator,long diskCacheSize, int diskCacheFileCount)

创建一个 Disk Cache。如果 diskCacheSize 或者 diskCacheFileCount 大于 0，返回一个LruDiskCache，否则返回无大小限制的UnlimitedDiskCache。

-	MemoryCache createMemoryCache(Context context, int memoryCacheSize)

创建一个 Memory Cache。返回一个LruMemoryCache，若 memoryCacheSize 为 0，则设置该内存缓存的最大字节数为App最大可用内存的1/8。这里的App的最大可用内存也支持系统在Honeycomb之后（Api Level >= 11)application中android:largeHeap="true"的设置。

-	ImageDownloader createImageDownloader(Context context)

创建图片下载器，返回一个BaseImageDownloader。

-	ImageDecoder createImageDecoder(boolean loggingEnabled)

创建图片解码器，返回一个BaseImageDecoder。

-	BitmapDisplayer createBitmapDisplayer()

创建图片显示器，返回一个SimpleBitmapDisplayer。

### DefaultConfigurationFactory.DefaultThreadFactory

默认的线程工厂类，为

DefaultConfigurationFactory.createExecutor(…)
和
DefaultConfigurationFactory.createTaskDistributor(…)
提供线程工厂。支持线程优先级设置，并且以固定规则命名新建的线程。

### ImageAware

需要显示图片的对象的接口，可包装View表示某个需要显示图片的View。

主要函数：

-	View getWrappedView()

得到被包装的View，图片显示在该View上

-	int getWidth()和int getHeight()

得到宽度高度，在计算图片缩放比例时会用到

-	int getId()

得到唯一标识id。ImageLoaderEngine中用这个id标识正在加载图片的ImageAware和图片内存缓存key的对应关系，图片请求前会将内存缓存key与新的内存缓存key进行比较，如果不相等，则之前的图片请求会被取消。这样当ImageAware被复用时就不会因异步加载（前面任务未取消）而造成错乱了。

### ViewAware.java

封装Android View来显示图片的抽象类，实现类ImageAware接口，利用Reference来Wrap View防止内存泄露。

主要函数：

-	ViewAware(View view, boolean checkActualViewSize)

构造函数：
view表示需要显示图片的对象
checkActualViewSize表示通过getWidth()和getHeight()获取图片宽高时返回真实的宽和高，还是LayoutParams的宽高，true表示返回真实宽和高。

如果为true会导致一个问题，view在还没有初始化完成时加载图片，这是它的真实宽高为0，会取它LayoutParams的宽高，而图片缓存的key与这个宽高有关，所以当view的初始化完成再次需要加载该图片时，getWidth()和getHeight()返回的宽高都已经变化了，缓存key不一样，从而导致缓存读取失败会再次从网络下载一次图片。可通过ImageLoaderConfiguration.Builder.denyCacheImageMultipleSizesInMemory()设置不允许内存缓存缓存一张图片的多个尺寸。

-	boolean setImageDrawable(Drawable drawable)

如果当前操作在主线程并且View没有被回收，则调用抽象函数setImageDrawableInto(Drawable drawable, View view)去向view设置图片。

-	boolean setImageBitmap(Bitmap bitmap)

如果当前操作在主线程并且View没有被回收，则调用抽象函数setImageBitmapInto(Bitmap bitmap, View view)去向view设置图片。

### ImageViewAware.java

封装Android ImageView来显示图片的ImageAware，继承了viewAware，利用Reference来Wrap View防止内存泄露。

如果getWidth()函数小于等于0，会利用反射获取mMaxWidth的值作为宽。
如果getHeight()函数小于等于0，会利用反射获取mMaxHeight的值作为高。

### NonViewAware

仅包含处理图片相关信息却没有需要显示图片的View的ImageAware，实现了ImageAware接口。常用于加载图片后调用回调接口而不是显示的情况。

### DisplayImagesOptions.java

图片显示的配置项。比如加载前、加载中、加载失败应该显示的占位图片，图片是否需要在磁盘缓存，是否需要在memory缓存等。

主要属性及含义：

-	int imageResOnLoading

图片正在加载中的占位图片的resource id，优先级比下面的imageOnLoading高，当存在时，imageOnLoading不起作用。

-	int imageResForEmptyUri

空uri时的占位图片的resource id，优先级比下面的imageForEmptyUri高，当存在时，imageForEmptyUri不起作用。

-	int imageResOnFail

加载失败时的占位图片的resource id，优先级比下面的imageOnFail高，当存在时，imageOnFail不起作用。

-	Drawable imageOnLoading

加载中的占位图片的Drawable对象，默认为null

-	Drawable imageForEmptyUri

空uri时的占位图片drawable对象，默认为null

- boolean resetViewBeforeLoading

在加载前是否重置view，通过Builder构建的对象默认为false

-	boolean cacheInMemory

是否缓存在内存中，通过Builder构建的对象默认为false。

-	boolean cacheOnDisk

是否缓存在磁盘中，通过Builder构建的对象默认为false。

-	ImageScaleType imageScaleType

图片的缩放类型，通过Builder构建的对象默认为IN_SAMPLE_POWER_OF_2

-	Options decodingOptions

为BitmapFactory.Options，用于BitmapFactory.decodeStream(imageStream, null, decodingOptions)得到图片尺寸等信息。

-	int delayBeforeLoading

设置在开始加载前的延迟时间，单位为毫秒，通过Builder构建的对象默认为0。

-	boolean considerExitParams

是否考虑图片的EXIF信息，通过Builder构建的对象默认为false。

-	Object extraForDownloader

下载器需要的辅助信息。下载时传入ImageDownloader.getStream(String,Object)的对象，方面用户自己扩展，默认为null

-	BitmapProcessor preProcessor;

缓存在内存之前的处理程序，默认为null

-	BitmapProcessor postProcessor

缓存在内存之后的处理程序，默认为null

-	BitmapDisplayer displayer;

图片的显示方式，通过Builder构建的对象默认为SimpleBitmapDisplayer

-	Handler handler;

handler对象，默认为null

-	boolean isSyncLoading;

是否同步加载，通过Builder构建的对象默认为false。

### DisplayImageOptions.Builder.java 静态内部类

Builder模式，用于构造参数繁多的DisplayImageOptions。

### ImageLoadingListener

图片加载各种时刻的回调接口，可在图片加载的某些点做监听。
包括开始加载（onLoadingStarted）、加载失败（onLoadingFailed）、加载成功（onLoadingComplete）、取消加载（onLoadingCancelled）四个回调函数。

### SimpleImageLoadingListener

实现ImageLoadingListener接口，不过各个函数都是空实现，表示不在Image加载过程中做任何回调监听实现。ImageLoader.displayImage()函数中当listener传入值为null时的默认值。

### ImageLoadingProgressListener.java

Image加载进度的回调接口

```Java
public interface ImageLoadingProgressListener {

	/**
	 * Is called when image loading progress changed.
	 *
	 * @param imageUri Image URI
	 * @param view     View for image. Can be <b>null</b>.
	 * @param current  Downloaded size in bytes
	 * @param total    Total size in bytes
	 */
	void onProgressUpdate(String imageUri, View view, int current, int total);
}
```
会在获取图片存储到文件系统时被调用，其中total表示图片总大小，为网络请求结果Response Header中content-length字段，如果不存在则为-1。

### DisplayBitmapTask.java

显示图片的Task，实现类Runnable接口，必须在主线程调用。

主要函数：

-	run()

首先判断ImageAware是否被GC回收，如果是直接调用取消加载回调接口listener.onLoadingCancelled()；
否则判断ImageAware是否被复用，如果是直接调用取消加载回调接口listener.onLoadingCancelled()；
否则调用diaplay显示图片，并将ImageAware从正在加载的map中移除。调用加载成功回调接口listener.onLoadingComplete()；

对于ListView或是GridView这里缓存item的View来说，单个Item中如果含有ImageView，在滑动过程中可能因为异步加载及View复用导致图片错乱，这里对ImageAware是否被复用的判断就能很好的解决这个问题。原因类似：[
Android ListView滑动过程中图片显示重复错位闪烁问题解决](http://www.trinea.cn/android/android-listview-display-error-image-when-scroll/)

### ProcessAndDisplayImageTask.java

处理并显示图片的Task，实现了Runnable接口。

主要函数：

-	run()

主要通过imageLoadingInfo得到BitmapProcessor处理图片，并且处理后的图片和配置新建一个DisplayBitmapTask在ImageAware中显示图片。

### LoadAndDisplayImageTask.java

加载并显示图片的Task，实现了Runnable接口，用于从网络、文件系统或内存获取图片并解析，然后调用DisplayBitmapTask在ImageAware中显示图片。

主要函数：

-	run()

获取图片并显示，核心代码如下：

```Java
bmp = configuration.memoryCache.get(memoryCacheKey);
if (bmp == null || bmp.isRecycled()) {
	bmp = tryLoadBitmap();
	if (bmp == null) return; // listener callback already was fired


	if (bmp != null && options.isCacheInMemory()) {
		L.d(LOG_CACHE_IMAGE_IN_MEMORY, memoryCacheKey);
		configuration.memoryCache.put(memoryCacheKey, bmp);
	}

	if (bmp != null && options.shouldPostProcess()) {
		L.d(LOG_POSTPROCESS_IMAGE, memoryCacheKey);
		bmp = options.getPostProcessor().process(bmp);
		if (bmp == null) {
			L.e(ERROR_POST_PROCESSOR_NULL, memoryCacheKey);
		}
	}

}

DisplayBitmapTask displayBitmapTask = new DisplayBitmapTask(bmp, imageLoadingInfo, engine, loadedFrom);
runTask(displayBitmapTask, syncLoading, handler, engine);
```

从上面代码可以看到显示从内存缓存中去读bitmap对象，若bitmap对象不存在，则调用tryLoadBitma()函数获取bitmap对象，获取成功后若在DisplayImageOptions.Builder中设置类cacheInMemory(true),同时将Bitmap对象缓存到内存中。最后新建DisplayBitmapTask对象显示图片。

函数流程图如下：

![](http://7xs7a3.com1.z0.glb.clouddn.com/uil-load-display-flow-chart.png)

1. 判断图片的内存缓存是否存在，若存在直接执行步骤8；
2. 判断图片的内存缓存是否存在，若存在直接执行步骤5；
3. 从网络上下载图片
4. 将图片缓存在磁盘上
5. 将图片decode成bitmap对象；
6. 根据DisplayImageOptions配置对图片进行预处理；
7. 将Bitmap对象缓存到内存中；
8. 根据DisplayImageOptions配置对图片进行后处理；
9. 执行DisplayBitmapTask将图片显示在相应的控件上；

-	tryLoadBitmap()

从磁盘缓存或网络获取图片，核心代码如下：

```Java
	private Bitmap tryLoadBitmap() throws TaskCancelledException {
		Bitmap bitmap = null;
		try {
			File imageFile = configuration.diskCache.get(uri);
			if (imageFile != null && imageFile.exists() && imageFile.length() > 0) {
				
				...
				
				bitmap = decodeImage(Scheme.FILE.wrap(imageFile.getAbsolutePath()));
			}
			if (bitmap == null || bitmap.getWidth() <= 0 || bitmap.getHeight() <= 0) {

				...

				String imageUriForDecoding = uri;
				if (options.isCacheOnDisk() && tryCacheImageOnDisk()) {
					imageFile = configuration.diskCache.get(uri);
					if (imageFile != null) {
						imageUriForDecoding = Scheme.FILE.wrap(imageFile.getAbsolutePath());
					}
				}

				checkTaskNotActual();
				bitmap = decodeImage(imageUriForDecoding);

			}
		}
		return bitmap;
	}
```

首先根据uri看磁盘中是不是已经缓存了这个文件，如果已经缓存，调用decodeImage函数，将图片文件decode成Bitmap对象；如果Bitmap对象不合法或缓存文件不存在，判断是否需要缓存在磁盘，需要则调用tryCacheImageOnDisk()函数去下载并缓存图片到本地磁盘，再通过decodeImage(imageUri)函数将图片文件decode成bitmap对象，否则直接通过decodeImage(imageUriForDecoding)下载图片并解析。

### tryCacheImageOnDisk()

下载图片并存储在磁盘内，根据磁盘缓存图片最长宽高的配置处理图片

```Java
	private boolean tryCacheImageOnDisk() throws TaskCancelledException {
		L.d(LOG_CACHE_IMAGE_ON_DISK, memoryCacheKey);

		boolean loaded;
		try {
			loaded = downloadImage(); //调用下载器并保存图片
			if (loaded) {
				int width = configuration.maxImageWidthForDiskCache;
				int height = configuration.maxImageHeightForDiskCache;
				if (width > 0 || height > 0) {
					L.d(LOG_RESIZE_CACHED_IMAGE_FILE, memoryCacheKey);
					resizeAndSaveImage(width, height); // TODO : process boolean result
				}
			}
		} catch (IOException e) {
			L.e(e);
			loaded = false;
		}
		return loaded;
	}
```
如果你在ImageLoaderConfiguration中配置了maxImageWidthForDiskCache或者maxImageHeightForDiskCache，还会调用resizeAndSaveImage()函数，调整图片尺寸，并保存新的图片文件。

### downloadImage()

下载图片并存储在磁盘内。调用getDownloader()得到ImageDownloader其下载图片。

### resizeAndSaveImage(int maxWidth,int maxHeight)

从磁盘缓存中得到图片，重新设置大小及进行一些处理后保存。

### geDownloader()

根据ImageLoaderEngine配置得到下载器。
如果不允许访问网络，则使用不允许访问网络的图片下载器NetWorkDeniedImageDownloader；如果是慢网络情况，则使用慢网络情况下的图片下载器SlowNetworkImageDownloader；否则直接使用ImageLoaderConfiguration中的downloader。

### ImageLoadingInfo.java

加载和显示图片任务需要的信息。成员变量如下：

String uri  图片url
String memoryCacheKey  图片缓存key
ImageAware imageAware  需要加载图片的对象
ImageSize targetSize  图片的显示尺寸
DisplayImageOptions options; 图片显示的配置项
ImageLoadingListener listener; 图片加载时状态的回调接口
ImageLoadingProgressListener progressListener; 图片加载进度的回调接口
ReentrantLock loadFromUriLock; 图片加载中的重入锁

### ImageDownloader.java

图片下载接口，待实现函数

```Java
getString(String imageUri, Object extra)
```
表示通过uri得到InputStream
通过内部定义的枚举Scheme，可以看出UIL支持哪些图片来源。

```
HTTP("http"), HTTPS("https"), FILE("file"), CONTENT("content"), ASSETS("assets"), DRAWABLE("drawable"), UNKNOWN("");
```

### BaseImageDownloader.java

ImageDownloader的具体实现类。得到上面各种Scheme对应的图片InputStream。

主要函数：

-	InputStream getStream(String imageUri, Object extra)

函数内根据不同Scheme类型获取图片输入流

```Java
	public InputStream getStream(String imageUri, Object extra) throws IOException {
		switch (Scheme.ofUri(imageUri)) {
			case HTTP:
			case HTTPS:
				return getStreamFromNetwork(imageUri, extra);
			case FILE:
				return getStreamFromFile(imageUri, extra);
			case CONTENT:
				return getStreamFromContent(imageUri, extra);
			case ASSETS:
				return getStreamFromAssets(imageUri, extra);
			case DRAWABLE:
				return getStreamFromDrawable(imageUri, extra);
			case UNKNOWN:
			default:
				return getStreamFromOtherSource(imageUri, extra);
		}
	}
```
-	InputStream getStreamFromNetwork(String imageUri, Object extra)

通过HttpURLConnection从网络获取图片的InputStream，支持response code为3xx的重定向。这里有个小细节代码如下：

```Java
InputStream imageStream;
try {
	imageStream = conn.getInputStream();
} catch (IOException e) {
	// Read all data to allow reuse connection (http://bit.ly/1ad35PY)
	IoUtils.readAndCloseStream(conn.getErrorStream());
	throw e;
}
if (!shouldBeProcessed(conn)) {
	IoUtils.closeSilently(imageStream);
	throw new IOException("Image request failed with response code " + conn.getResponseCode());
}
```
在发生异常时会调用`conn.getErrorStream()`继续读取Error Stream，这是为了利用网络连接回收及复用，但有意思的是在2.2之前，HttpURLConnection有个重大Bug，调用close()函数会影响连接池，导致连接复用失效，不过2.3以后已经解决了此went。

-	InputStream getStreamFromFile(String imageUri, Object extra)

从文件系统获取图片的InputStream。如果uri的类型是Video，则得到video的缩略图返回，否则按照一般文件操作返回。

```Java
	protected InputStream getStreamFromFile(String imageUri, Object extra) throws IOException {
		String filePath = Scheme.FILE.crop(imageUri);
		if (isVideoFileUri(imageUri)) {
			return getVideoThumbnailStream(filePath); //缩略图
		} else {
			BufferedInputStream imageStream = new BufferedInputStream(new FileInputStream(filePath), BUFFER_SIZE);
			return new ContentLengthInputStream(imageStream, (int) new File(filePath).length());
		}
	}
```

-	InputStream getStreamFromContent(String imageUri, Object extra)

从ContentProvider获取图片的InputStream。
如果是video类型，则先从MediaStore得到video的缩略图返回；
如果是联系人类型，则通过`ContactsContract.Contacts.openContactPhotoInputStream(res, uri, true)`读取内容返回；
否则通过`ContentResolver..openInputStream(uri)`读取内容返回

-	InputStream getStreamFromAssets(String imageUri, Object extra)

从Assets文件夹中获取图片的InputStream

```Java
	protected InputStream getStreamFromAssets(String imageUri, Object extra) throws IOException {
		String filePath = Scheme.ASSETS.crop(imageUri);
		return context.getAssets().open(filePath);
	}
```
-	InputStream getStreamFromDrawable(String imageUri, Object extra)

从Drawable资源中获取图片的InputStream。

-	InputStream getStreamFromOtherSource(String imageUri, Object extra)

UNKNOWN类型的处理，目前直接抛出不支持的异常

### MemoryCache.java

Bitmap内存缓存接口，需要实现的接口包括get()、put()、remove()、clear()、keys()

```Java
public interface MemoryCache {
	/**
	 * Puts value into cache by key
	 */
	boolean put(String key, Bitmap value);

	/** Returns value by key. If there is no value for key then null will be returned. */
	Bitmap get(String key);

	/** Removes item by key */
	Bitmap remove(String key);

	/** Returns all keys of cache */
	Collection<String> keys();

	/** Remove all items from cache */
	void clear();
}
```

### BaseMemoryCache.java

实现了MemoryCache主要函数的抽象类，以`Map<String, Reference<Bitmap>> softMap`作为缓存池，利于虚拟机在内存不足是回收缓存对象。提供抽象函数：

```Java
protected abstract Reference<Bitmap> createReference(Bitmap value);
```
表示根据Bitmap创建一个Reference作为缓存对象。Reference可以是WeakReference、SoftReference等。

### WeakMemoryCache.java

以`WeakReference<Bitmap>`作为缓存value的内存缓存，实现了BaseMemoryCache的`createReference(Bitmap value)`函数，直接返回一个`new WeakReference<Bitmap>(value)`作为缓存value。

### LimitedMemoryCache.java

限制总字节大小的内存缓存，继承自BaseMemoryCache抽象类。
会在put(...)函数中判断总体大小是否超出上限，超出则循环删除缓存对象直到小于上限。删除顺序由抽象函数`protected abstract Bitmap removeNext()`决定。抽象函数`protected abstract int getSize(Bitmap value)`表示每个元素大小。

### LargestLimitedMemoryCache.java

限制总字节大小的内存缓存，会在缓存满时优先删除size最大的元素，继承自LimitedMemoryCache。实现了LimitedMemoryCache的removeNext()函数，总是返回当前缓存中size最大的元素。

### UsingFreqLimitedMemoryCache.java

限制总字节大小的内存缓存，会在缓存满时优先删除使用次数最少的元素，继承自LimitedMemoryCache。实现了LimitedMemoryCache的removeNext()函数，总是返回当前缓存中使用次数最少的元素。

### LRULimitedMemoryCache.java

限制总字节大小的内存缓存，会在缓存满时优先删除最近最少使用的元素，继承自LimitedMemoryCache。通过`new LinkedHashMap<String, Bitmap>(10, 1.1f, true)`作为缓存池。LinkedHashMap第三个参数表示是否需要根据访问顺序(accessOrder)排序，true表示根据accessOrder排序，最近访问的跟最新加入的一样放到最后面，false表示根据插入顺序排序。这里为true且缓存满时始终删除第一个元素，即始终删除最近最少访问的元素。实现了LimitedMemoryCache的removeNext()函数，总是返回当前缓存中最近最少使用的元素。

### FIFOLimitedMemoryCache.java

限制总字节大小的内存缓存，会在缓存满的时优先删除进入缓存的元素，继承自LimitedMemoryCache。实现了LimitedMemoryCache的removeNext()函数，总是返回当前缓存中最先进入缓存的元素。

>**以上所有LimitedMemoryCache子类都有个问题，就是Bitmap虽然通过WeakReference<Bitmap>包装，但实际根本不会被虚拟机回收，因为他们子类中同时都保留了Bitmap的强引用。这些大都是UIL早期实现的版本，不推荐使用**

### LruMemoryCache.java

限制总字节大小的内存缓存，会在缓存满时优先删除最近最少使用的元素，实现了MemoryCache。LRU(Least Recently Used)为最少使用算法。

通过`new LinkedHashMap<String, Bitmap>(0, 0.75f, true)`作为缓存池。LinkedHashMap第三个参数表示是否需要根据访问顺序(accessOrder)排序，true表示根据accessOrder排序，最近访问的跟最新加入的一样放到最后面，false表示根据插入顺序排序。这里为true且缓存满时始终删除第一个元素，即始终删除最近最少访问的元素。

在put(..)函数中通过trimToSize(int maxtSize)函数判断总体大小是否超出了上限，是则删除缓存池中第一个元素，即最近最少使用的元素，指导总体大小小于上限。

LruMemory功能上谕LRULimitedMemoryCache类似，不过在实现上更加优雅，用简单的实现接口方式，而不是不断继承的方式。

### LimitedAgeMemoryCache.java

限制类对象最长存活周期的内存缓存。
MemoryCache的装饰者，相当于为MemoryCache添加一个特性，以一个MemoryCache内存缓存和一个maxAge作为构造函数参数。在get()中判断如果对象存活时间已经超过设置的最长时间，则删除。

### FuzzyKeyMemoryCache.java

可以将某些原本不同的key看做相等，在put时删除这些相等的key。
MemoryCache的装饰者，相当于为MemoryCache添加一个特性，以一个MemoryCache内存缓存和一个 keyComparator作为构造函数参数。在put()函数中判断如果key与缓存中已有key经过Comparator比较后相等，则删除之前的元素。

### FileNameGenerator.java

根据uri得到文件名的接口

### HashCodeFileNameGenerator.java

以uri的hashCode值作为文件名

### Md5FileNameGenerator.java

以uri的MD5值作为文件名

```Java
public class Md5FileNameGenerator implements FileNameGenerator {

	private static final String HASH_ALGORITHM = "MD5";
	private static final int RADIX = 10 + 26; // 10 digits + 26 letters

	@Override
	public String generate(String imageUri) {
		byte[] md5 = getMD5(imageUri.getBytes());
		BigInteger bi = new BigInteger(md5).abs();
		return bi.toString(RADIX);
	}

	private byte[] getMD5(byte[] data) {
		byte[] hash = null;
		try {
			MessageDigest digest = MessageDigest.getInstance(HASH_ALGORITHM);
			digest.update(data);
			hash = digest.digest();
		} catch (NoSuchAlgorithmException e) {
			L.e(e);
		}
		return hash;
	}
}
```

### DiskCache.java

图片的磁盘缓存接口。

主要函数：

-	File getDirectory()

得到磁盘缓存的根目录

-	File get(String imageUri)

根据原始图片uri去获取缓存图片的文件

-	boolean save(String imageUri, InputStream imageStream, IoUtils.CopyListener listener)

保存imageStream到磁盘中，listener表示保存进度且可在其中取消某些段的保存。

-	boolean save(String imageUri, Bitmap bitmap)

保存图片到磁盘

-	boolean remove(String imageUri)

根据图片uri删除缓存图片

-	void close()

关闭磁盘缓存，并释放资源

-	void clear()

清空磁盘缓存

### BaseDiskCache.java

一个无大小限制的本地图片缓存，实现了DiskCache主要函数的抽象类。
图片缓存在cacheDir文件夹内，当cacheDir不可用时，则使用备用库reserveCacheDir。

主要函数：

-	boolean save(String imageUri, InputStream imageStream, IoUtils.CopyListener listener)

先根据imageUri得到目标文件，将imageStream先写入与目标文件同一文件夹的.tmp结尾的临时文件内，若未被listener取消且写入成功则将临时文件重命名为目标文件并返回true，否则删除临时文件并返回false。

-	boolean save(String imageUri, Bitmap bitmap)

先根据imageUri得到目标文件，通过Bitmap.compress(..)函数将bitmap先写入与目标文件同一文件夹的.tmp结尾的临时文件内，若写入成功则将临时文件名重命名为目标文件并返回true，否则删除临时文件并返回false。

-	File getFile(String imageUri)

根据imageUri和fileNameGenerator得到文件名，返回cacheDir文件夹内该文件，若cacheDir不可用，则使用备用库reserveCacheDir。

### LimitedAgeDiskCache.java

限制缓存对象最长存活周期的磁盘缓存，继承自BaseDiskCache。
在get()函数判断如果缓存对象存活时间已经超过设置的最长时间，则删除。在save()中保存当前时间作为对象的创建时间。

### UnLimitedDiskCache.java

一个无大小限制的本地图片缓存。与BaseDiskCache无异，只是用了个意思明确的类名。

### DiskLruCache.java

限制总字节大小的磁盘缓存，会在缓存满时优先删除最近最少使用的元素。

通过缓存目录下名为journal的文件记录缓存的所有操作，并在缓存open时读取journal的文件内容存储到`LinkedHashMap<String, Bitmap> lruEntries`，后面`get(String key)`获取缓存内容时，会先从lruEntries中得到图片文件名返回文件。

通过`new LinkedHashMap<String, Entry>(0, 0.75f, true)`作为缓存池。LinkedHashMap第三个参数表示是否需要根据访问顺序(accessOrder)排序，true表示根据accessOrder排序，最近访问的跟最新加入的一样放到最后面，false表示根据插入顺序排序。这里为true且缓存满时trimToSize()函数始终删除第一个元素，即始终删除最近最少访问的元素。

### LruDiskCache.java

限制总字节大小的本地缓存，会在缓存满时优先删除最近最少使用的元素，实现了DiskCache。内部有个DiskLruCache cache、属性，缓存的存、取操作基本都是由该属性代理完成。

### StrictLineReader.java

通过readLine()函数从InputStream中读取一行，目前仅用于磁盘缓存操作记录文件journal的解析。

### Util.java

工具类：

String readFully(Reader reader)读取 reader 中内容。
deleteContents(File dir)递归删除文件夹内容。

### ContentLengthInputStream.java

InputStream的装饰者，可通过available()函数得到 InputStream 对应数据源的长度(总字节数)。主要用于计算文件存储进度即图片下载进度时的总进度。

### FailReason.java

图片下载及显示时的错误原因，目前包括：
IO_ERROR 网络连接或是磁盘存储错误。
DECODING_ERROR decode image 为 Bitmap 时错误。
NETWORK_DENIED 当图片不在缓存中，且设置不允许访问网络时的错误。
OUT_OF_MEMORY 内存溢出错误。
UNKNOWN 未知错误。

### FlushedInputStream.java

为了解决早期 Android 版本BitmapFactory.decodeStream(…)在慢网络情况下 decode image 异常的 Bug。
主要通过重写FilterInputStream的 skip(long n) 函数解决，确保 skip(long n) 始终跳过了 n 个字节。如果返回结果即跳过的字节数小于 n，则不断循环直到 skip(long n) 跳过 n 字节或到达文件尾。

### ImageScaleType.java

Image 的缩放类型，目前包括：
NONE不缩放。
NONE_SAFE根据需要以整数倍缩小图片，使得其尺寸不超过 Texture 可接受最大尺寸。
IN_SAMPLE_POWER_OF_2根据需要以 2 的 n 次幂缩小图片，使其尺寸不超过目标大小，比较快的缩小方式。
IN_SAMPLE_INT根据需要以整数倍缩小图片，使其尺寸不超过目标大小。
EXACTLY根据需要缩小图片到宽或高有一个与目标尺寸一致。
EXACTLY_STRETCHED根据需要缩放图片到宽或高有一个与目标尺寸一致。

### ViewScaleType.java

ImageAware的 ScaleType。
将 ImageView 的 ScaleType 简化为两种FIT_INSIDE和CROP两种。FIT_INSIDE表示将图片缩放到至少宽度和高度有一个小于等于 View 的对应尺寸，CROP表示将图片缩放到宽度和高度都大于等于 View 的对应尺寸。

### ImageSize.java

表示图片宽高的类。
scaleDown(…) 等比缩小宽高。
scale(…) 等比放大宽高。

### LoadedFrom.java

图片来源枚举类，包括网络、磁盘缓存、内存缓存。

### ImageDecoder.java

将图片转换为 Bitmap 的接口，抽象函数：
Bitmap decode(ImageDecodingInfo imageDecodingInfo) throws IOException;
表示根据ImageDecodingInfo信息得到图片并根据参数将其转换为 Bitmap。

### BaseImageDecoder.java

实现类ImageDecoder。调用ImageDownloader获取图片，然后根据ImageDecodingInfo或图片Exif信息处理图片转换为Bitmap。

主要函数：

-	decode(ImageDecodingInfo decodingInfo)

调用ImageDownloader获取图片，再调用defineImageSizeAndRotation()函数得到图片的相关信息，调用preparedDecodingOptions()得到图片缩放的比例，调用BitmapFactory.decodeStream()将InputStream转换为Bitmap，最后调用considerExactScaleAndOrientatiton()根据参数将图片放大、翻转、旋转为合适的样子返回。

-	ImageFileInfo defineImageSizeAndRotation(InputStream imageStream, ImageDecodingInfo decodingInfo)

得到图片真实大小以及Exif信息（设置考虑Exif的条件下）

-	ExifInfo defineExifOrientation(String imageUri)

得到图片Exif信息中的翻转以及旋转角度信息。

-	Options prepareDecodingOptions(ImageSize imageSize, ImageDecodingInfo decodingInfo)

得到图片缩放的比例：
1. 如果scaleType等于ImageScaleType.NONE，则缩放比例为1；
2. 如果scaleType等于ImageScaleType.NONE_SAFE，则缩放比例为ImageSizeUtils.computeImageSampleSize.computeMinImageSampleSize()的返回值。
3. 否则，调用ImageSizeUtils.computeImageSampleSize()计算返回值。

在computeImageSampleSize()中

1. 如果viewScaleType等于FIT_INSIDE：
	1.1 如果scaleType等于ImageScaleType.IN_SAMPLE_POWER_OF_2，则缩放比例从1开始不断*2直到宽或高小于最大尺寸。
	1.2 否则，取宽和高分别与最大尺寸比例中较大值，即Math.max(srcWidth / targetWidth, srcHeight / targetHeight)。

2. 如果viewScaleType等于CROP；
	2.1 如果scaleType等于ImageScaleType.IN_SAMPLE_POWER_OF_2，则缩放比例从1开始不断*2直到宽或高小于最大尺寸。
	2.2 否则，取宽和高分别与最大尺寸比例中较小值，即Math.min(srcWidth / targetWidth, srcHeight / targetHeight)

3. 最后，在considerMaxTextureSize()中判断宽和高是否超过最大值，如果是则*2或是+1缩放。

-	Bitmap considerExactScaleAndOrientatiton(Bitmap subsampledBitmap, ImageDecodingInfo decodingInfo,int rotation, boolean flipHorizontal)

根据参数将图片放大、翻转、旋转为合适的样子返回。

### ImageDownloadingInfo.java

Image Decode 需要的信息。
String imageKey 图片。
String imageUri 图片 uri，可能是缓存文件的 uri。
String originalImageUri 图片原 uri。
ImageSize targetSize 图片的显示尺寸。
imageScaleType 图片的 ScaleType。
ImageDownloader downloader 图片的下载器。
Object extraForDownloader 下载器需要的辅助信息。
boolean considerExifParams 是否需要考虑图片 Exif 信息。
Options decodingOptions 图片的解码信息，为 BitmapFactory.Options。

### BitmapDisplayer.java

在ImageAware中显示 bitmap 对象的接口。可在实现中对 bitmap 做一些额外处理，比如加圆角、动画效果。

### FadeInBitmapDisplayer.java

图片淡入方式显示在ImageAware中，实现了BitmapDisplayer接口。

### RoundedBitmapDisplayer.java

为图片添加圆角显示在ImageAware中，实现了BitmapDisplayer接口。主要通过BitmapShader实现。

### RoundedVignetteBitmapDisplayer.java

为图片添加渐变效果的圆角显示在ImageAware中，实现了BitmapDisplayer接口。主要通过RadialGradient实现。

### SimpleBitmapDisplayer.java

直接将图片显示在ImageAware中，实现了BitmapDisplayer接口。

### BitmapProcessor.java

图片处理接口。可用于对图片预处理(Pre-process Bitmap）和后处理(Post-process Bitmap)。抽象函数：

```Java
public interface BitmapProcessor {

	Bitmap process(Bitmap bitmap);
}
```
用户可以根据自己的需要去实现它。比如你想要为你的图片添加一个水印，那么可以自己去实现BitmapProcessor接口。在DisplayImageOptions中配置Pre-process阶段预处理图片，这样设置后存储在文件系统以及内存缓存中的图片都是加了水印的。如果只希望在显示时改变不动原图片，可以在BitmapDisplayer中处理。

### PauseOnScrollListener.java

可以在View滚动过程中暂停图片加载的Listener，实现了OnScrollListener接口。
它的好处是防止滚动中不必要的图片加载，在ListView或GridView中item加载图片最好使用它，简单的一行代码：

```Java
gridView.setOnScrollListener(new PauseOnScrollListener(ImageLoader.getInstance(), false, true));
```
主要成员变量：

pauseOnScroll; 触摸(手指依然在屏幕上)滑动过程中是否暂停图片加载
pauseOnFling;  甩指(手指已离开屏幕)过程中是否暂停图片加载
externalListener; 自定义的OnScrollListener接口，适用于View原来就有自定义OnScrollListener情况设置

实现原理：重写onScrollStateChanged(…)函数判断不同的状态下暂停或继续图片加载。

OnScrollListener.SCROLL_STATE_IDLE表示 View 处于空闲状态，没有在滚动，这时候会加载图片。

OnScrollListener.SCROLL_STATE_TOUCH_SCROLL表示 View 处于触摸滑动状态，手指依然在屏幕上，通过pauseOnScroll变量确定是否需要暂停图片加载。这种时候大都属于慢速滚动浏览状态，所以建议继续图片加载。

OnScrollListener.SCROLL_STATE_FLING表示 View 处于甩指滚动状态，手指已离开屏幕，通过pauseOnFling变量确定是否需要暂停图片加载。这种时候大都属于快速滚动状态，所以建议暂停图片加载以节省资源。

### QueueProcessingType.java

任务队列的处理类型，包括FIFO先进先出、LIFO后进先出。

### LIFOLinkedBlockingDeque.java

后进先出阻塞队列。重写LinkedBlockingDeque的offer()函数如下：

```Java
@Override
public boolean offer(T e) {
	return super.offerFirst(e);
}
```
让LinkedBlockingDeque插入总在最前，而remove()本身始终删除第一个元素，所以就变为了后进先出阻塞队列。实际一般情况只重写offer(…)函数是不够的，但因为ThreadPoolExecutor默认只用到了BlockingQueue的offer(…)函数，所以这种简单重写后做为ThreadPoolExecutor的任务队列没问题。

LIFOLinkedBlockingDeque.java包下的LinkedBlockingDeque.java、BlockingDeque.java、Deque.java都是 Java 1.6 源码中的，这里不做分析。

### DiskCacheUtils.java

磁盘缓存工具类，可用于查找或删除某个 uri 对应的磁盘缓存。

### MemoryCacheUtils.java

内存缓存工具类。可用于根据 uri 生成内存缓存 key，缓存 key 比较，根据 uri 得到所有相关的 key 或图片，删除某个 uri 的内存缓存。
generateKey(String imageUri, ImageSize targetSize)
根据 uri 生成内存缓存 key，key 规则为[imageUri]_[width]x[height]。

### StorageUtils.java

得到图片 SD 卡缓存目录路径。
缓存目录优先选择/Android/data/[app_package_name]/cache；若无权限或不可用，则选择 App 在文件系统的缓存目录context.getCacheDir()；若无权限或不可用，则选择/data/data/[app_package_name]/cache。
如果缓存目录选择了/Android/data/[app_package_name]/cache，则新建.nomedia文件表示不允许类似 Galley 这些应用显示此文件夹下图片。不过在 4.0 系统有 Bug 这种方式不生效。

### ImageSizeUtils.java

用于计算图片尺寸、缩放比例相关的工具类。

### IoUtils.java

IO 相关工具类，包括 stream 拷贝，关闭等。

### L.java

Log 工具类。

## 后记

UIL的内存缓存默认使用了LRU算法，即近期最少使用算法，选用了基于链表结构的LinkedHashMap作为存储结构。

假设情景：内存缓存设置的阈值只够存储两个bitmap对象，当put第三个bitmap对象时，将近期最少使用的bitmap对象移除。
1. 初始化LinkedHashMap，并按使用顺序来排序，accessOrder = true
2. 向缓存池中放入bitmap1和bitmap2两个对象
3. 继续放入第三个bitmap3，根据假设情景，将会超过设定缓存池阈值
4. 释放对bitmap1对象的引用
5. bitmap1对象被GC回收

UIL的磁盘缓存默认使用了UnlimitedDiskCache
