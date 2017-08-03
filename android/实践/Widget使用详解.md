**Widget使用详解**

[TOC]

# Widget是什么

Widget就是桌面控件，就是指能直接显示在Android系统桌面的小程序。


# AppWidget原理

AppWidget是通过Broadcast的形式来进行控制的，因此每个桌面控件都对应于一个BroadcastReceiver。Android系统提供了一个APPWidgetProvider类，它继承了BroadcastReceiver类，来实现APPWidget的控制和管理。

## 创建AppWidget

### 创建Widget布局

在res/layout文件夹中创建process_widget.xml布局，如下所示：

```Java
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical" >
	
	    <TextView
	        android:id="@+id/tv_show"
	        android:layout_width="match_parent"
	        android:layout_height="30dp"
	        android:gravity="center"
	        android:text="hello" />
	
	    <Button
	        android:id="@+id/btn_presss"
	        android:layout_width="match_parent"
	        android:layout_height="50dp"
	        android:gravity="center"
	        android:text="press" />
	
	</LinearLayout>
```

### 创建appwidget-provider布局

在res/xml中创建mywidget_info.xml文件，如下所示：

```Java
	<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
	    android:minWidth="100dp"
	    android:minHeight="80dp"
	    android:updatePeriodMillis="10"
	    android:initialLayout="@layout/process_widget"
	    >
	</appwidget-provider>
	<!-- 指定widget的布局 宽高 更新周期 
	
	    android:minWidth="100dp" 
	    android:minHeight="80dp"
	    android:updatePeriodMillis="86400000"  86400000是24小时 //已无效
	    android:initialLayout="@layout/process_widget"
	-->
```

### 创建AppWidgetProvider类对象

创建MyWidget对象，继承AppWidgetProvider类，然后重写AppWidgetProvider不同状态的生命周期方法。

```Java
	public class MyWidget extends AppWidgetProvider {
		
		private static final String TAG = "MyWidget";
		
		//widget创建时回调该方法
		@Override
		public void onEnabled(Context context) {
			// TODO Auto-generated method stub
			super.onEnabled(context);
			Log.i(TAG, "onEnabled");
		}
		
		//更新桌面控件的回调方法
		@Override
		public void onUpdate(Context context, AppWidgetManager appWidgetManager,
				int[] appWidgetIds) {
			
			Log.i(TAG, "onUpdate");
			
		}

		//	
		@Override
		public void onDisabled(Context context) {
			// TODO Auto-generated method stub
			super.onDisabled(context);
			Log.i(TAG, "onDisabled");
		}
		
		//widget删除时回调该方法
		@Override
		public void onDeleted(Context context, int[] appWidgetIds) {
			// TODO Auto-generated method stub
			super.onDeleted(context, appWidgetIds);
			Log.i(TAG, "onDeleted");
		}
		
		//广播接收者回调方法
		@Override
		public void onReceive(Context context, Intent intent) {
			// TODO Auto-generated method stub
			super.onReceive(context, intent);
			
			Log.i(TAG, "onReceive: " + intent.getAction());
		}
	}
Widget添加桌面时回调方法调用顺序： onEnable--> onUpdate  
Widget移除桌面时回调方法调用顺序： onDeleted ---> onDisable  
```


### 在AndroidManifest文件在注册AppWidgetProvider类对象

```Java
    <receiver android:name="com.lgq.widget.MyWidget" >
        <intent-filter>
            <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
        </intent-filter>

        <!-- 配置meta-data 必须-->
        <meta-data
            android:name="android.appwidget.provider"
            android:resource="@xml/mywidget_info" />
    </receiver>
```

### 重写onUpdate方法实现Widget

```Java
	@Override
	public void onUpdate(Context context, AppWidgetManager appWidgetManager,
			int[] appWidgetIds) {

		Log.i(TAG, "onUpdate");

		// 1.创建RemoteViews对象加载指定Widget布局
		RemoteViews remoteViews = new RemoteViews(context.getPackageName(),
				R.layout.process_widget);

		// 2.为widget布局空间显示内容
		remoteViews.setTextViewText(R.id.tv_show, "Just do it");

		// 执行布局中Button点击事件
		Intent intent = new Intent();
		intent.setAction("com.lgq.press");
		PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0,
				intent, 0);
		remoteViews.setOnClickPendingIntent(R.id.btn_presss, pendingIntent);

		// 3.将AppWidgetProvider子类对象包装成ComponentName对象
		ComponentName componentName = new ComponentName(context, MyWidget.class);
		// 4. 将remoteViews添加到ComponentName中
		appWidgetManager.updateAppWidget(componentName, remoteViews);

	}
```Java

### 响应Button事件

```Java
	public class ClickReceiver extends BroadcastReceiver {
	
		@Override
		public void onReceive(Context context, Intent intent) {
			// TODO Auto-generated method stub
			
			//处理Button点击事件
			Toast.makeText(context, "已按下press按钮", Toast.LENGTH_SHORT).show();
			
			//更新widget
			RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.process_widget);
			
			//2.为widget布局空间显示内容
			remoteViews.setTextViewText(R.id.tv_show, "已按下press按钮");
			
			ComponentName componentName = new ComponentName(context, MyWidget.class);
			
			AppWidgetManager awm = AppWidgetManager.getInstance(context);
			
			awm.updateAppWidget(componentName, remoteViews);
	
		}
	
	}
```

## 定期更新Widget

### Widget添加桌面时开启定时更新的服务

```Java
public class MyWidget extends AppWidgetProvider {
    @Override
    public void onEnabled(Context context) {
    	//开启服务 定期的更新widget
    	Intent i = new Intent(context,UpdateWidgetService.class);
    	context.startService(i);
    	super.onEnabled(context);
    }
    
    @Override
    public void onDisabled(Context context) {
    	//停止服务 不再去更新widget
    	Intent i = new Intent(context,UpdateWidgetService.class);
    	context.stopService(i);
    	super.onDisabled(context);
    }
}
```

### 在服务中定时更新widget

```Java
	public class UpdateWidgetService extends Service {
		protected static final String TAG = "UpdateWidgetService";
		private Timer timer;
		private TimerTask task;
		private AppWidgetManager awm;
	
		@Override
		public IBinder onBind(Intent intent) {
			return null;
		}
	
		@Override
		public void onCreate() {
			awm = AppWidgetManager.getInstance(this);
			timer = new Timer();
			task = new TimerTask() {
				@Override
				public void run() {
					Log.i(TAG,"更新widget");
					//让桌面更新widget  由另外一个进程 更新ui
					ComponentName provider = new ComponentName(getApplicationContext(), MyWidget.class);
					//远程view的描述信息， 并不是一个真实的view对象， 由远程的桌面应用根据描述信息把view对象创建出来。
					RemoteViews views = new RemoteViews(getPackageName(), R.layout.process_widget);
					views.setTextViewText(R.id.process_count, "运行中的进程："+SystemInfoUtils.getRunningPocessCount(getApplicationContext()));
					String availsize = Formatter.formatFileSize(getApplicationContext(), SystemInfoUtils.getAvailMem(getApplicationContext()));
					views.setTextViewText(R.id.process_memory, "可用内存："+availsize);
					//由另外一个进程执行的动作,由桌面发送一个广播
					Intent intent = new Intent();//自定义的广播
					intent.setAction("com.itheima.killall");
					PendingIntent pendingIntent= PendingIntent.getBroadcast(getApplicationContext(), 0, intent, 0);
					views.setOnClickPendingIntent(R.id.btn_clear, pendingIntent);
					awm.updateAppWidget(provider, views);
				}
			};
			timer.schedule(task, 0, 5000);
			super.onCreate();
		}
	
		@Override
		public void onDestroy() {
			super.onDestroy();
			if (timer != null && task != null) {
				timer.cancel();
				task.cancel();
				timer = null;
				task = null;
			}
		}
	
	}
```

## AppWidgetHost

AppWidgetHost是真正容纳AppWidget的地方，它的主要功能有两个：

* 监听来自AppWidgetService的事件，这是主要处理update和provider_changed两个事件,根据这两个事件更新widget。

```Java
	  class UpdateHandler extends Handler {
	        public UpdateHandler(Looper looper) {
	            super(looper);
	        }
	       
	        public void handleMessage(Message msg) {
	            switch (msg.what) {
	                case HANDLE_UPDATE: {//更新事件
	                    updateAppWidgetView(msg.arg1, (RemoteViews)msg.obj);
	                    break;
	                }
	                case HANDLE_PROVIDER_CHANGED: {//升级事件
	                    onProviderChanged(msg.arg1, (AppWidgetProviderInfo)msg.obj);
	                    break;
	                }
	            }
	        }
	    }
```

* 另外一个功能就是创建AppWidgetHostView。前面我们说过RemoteViews不是真正的View，只是View的描述，而AppWidgetHostView才是真正的View。这里先创建AppWidgetHostView，然后通过AppWidgetService查询appWidgetId对应的RemoteViews，最后把RemoteViews传递给AppWidgetHostView去updateAppWidget。

```Java
	    protected AppWidgetHostView onCreateView(Context context, int appWidgetId,
	            AppWidgetProviderInfo appWidget) {
	        return new AppWidgetHostView(context);
	    }
	
		//更新和升级AppWidget实现：通过appWidgetId,获得相应的AppWidgetHostView 。
	    protected void onProviderChanged(int appWidgetId, AppWidgetProviderInfo appWidget) {
	        AppWidgetHostView v;
	        synchronized (mViews) {
	            v = mViews.get(appWidgetId);
	        }
	        if (v != null) {
	            v.updateAppWidget(null, AppWidgetHostView.UPDATE_FLAGS_RESET);
	        }
	    }
	
	    void updateAppWidgetView(int appWidgetId, RemoteViews views) {
	        AppWidgetHostView v;
	        synchronized (mViews) {
	            v = mViews.get(appWidgetId);
	        }
	        if (v != null) {
	            v.updateAppWidget(views, 0);
	        }
	    }
```


