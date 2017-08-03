**Android 5.x之 Notification**

[TOC]

Notification可以让我们在获得消息的时候，在状态栏、锁屏界面来显示相应的消息，如果没有Notification的话，很难想象我们的QQ和微信以及其他应用就没有办法主动通知我们，我们需要时候打开手机检查是否有新的消息到来，而这着实让人不爽。接下来，我们介绍三种Notification，分别是普通Notification，折叠式Notification和悬挂式Notification。

##1.普通Notification

首先，创建Builder对象，创建一个PendingIntent来实现消息点击跳转事件。

```
Notification.Builder builder = new Notification.Builder(this);
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://www.baidu.com/"));
PendingIntent pendingIntent = PendingIntent.getActivity(this,0,intent,0);
```

其次，通过builder给Notification添加不同的属性：

```
builder.setContentIntent(pendingIntent);
builder.setSmallIcon(R.mipmap.ic_launcher);
builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
builder.setAutoCancel(true);
builder.setContentTitle("普通Notification");
```

最后，创建NotifcationManager对象，调用notify发送一个通知

```
mManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
mManager.notify(0, builder.build());
```

效果如下：


![普通Notification](http://img.blog.csdn.net/20160312221647150)


##2.折叠式Notification

折叠式Notification是一种自定义视图的Notification，用来显示长文本和一些自定义的布局的场景。它有两种状态，一种是普通状态下的视图（如果不是自定义的话和上面普通通知的视图样式一样）；一种是展开状态下的视图，它需要自定义视图，并且这个自定义视图的显示的进程和我们创建视图的进程不是一个进程，需要使用RemoteViews。

首先，使用RemoteViews创建自定义视图

```
//用RemoteViews来创建自定义Notification视图
RemoteViews remoteViews = new RemoteViews(getPackageName(),R.layout.fold_notification);
```

视图的布局：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:background="#000000"
    android:orientation="horizontal">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#ffffffff"
        android:text="自定义的视图"/>
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="#ffffffff"
        android:text="折叠式通知"/>

</LinearLayout>
```
其次，把自定义视图赋给Notification

```
//赋值给Notification展开时的视图
notification.bigContentView = remoteViews;
或
//赋值给Notification普通状态时的视图
notification.contentView = remoteViews;
```

折叠式Notification的完整代码为:

```
Notification.Builder builder = new Notification.Builder(this);
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://www.baidu.com/"));
PendingIntent pendingIntent = PendingIntent.getActivity(this,0,intent,0);

builder.setContentIntent(pendingIntent);
builder.setSmallIcon(R.mipmap.ic_launcher);
builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
builder.setAutoCancel(true);
builder.setContentTitle("折叠式Notification");

//用RemoteViews来创建自定义Notification视图
RemoteViews remoteViews = new RemoteViews(getPackageName(),R.layout.fold_notification);
Notification notification = builder.build();
//指定展开的视图
notification.bigContentView = remoteViews;

mManager.notify(1,notification);
```

如果不是自定义普通视图的话，折叠式Notification普通状态和普通Notification没有什么区别，效果如下。

![普通Notification]()

接着把通知栏往下拉，使折叠式Notification完全展开就会出现自定义视图

![折叠式Notification](http://img.blog.csdn.net/20160312221555540)

##3.悬挂式Notification

悬挂式Notification是Android 5.0新加的通知方式，与前两种通知不同的是，悬挂式Notification不需要下拉通知栏就直接显示出来悬挂在屏幕上方并且不会占用用户的焦点因此不会打断用户的操作，过几秒就自动消失。

需要调用setFullScreenIntent来讲Notification变为悬挂式Notification。

悬挂式Notification的代码如下:

```
Notification.Builder builder = new Notification.Builder(this);
Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://www.baidu.com/"));
PendingIntent pendingIntent = PendingIntent.getActivity(this,0,intent,0);

builder.setContentIntent(pendingIntent);
builder.setSmallIcon(R.mipmap.ic_launcher);
builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
builder.setAutoCancel(true);
builder.setContentTitle("悬挂式Notification");

//设置点击跳转
Intent hangIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://www.baidu.com/"));
hangIntent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
//如果描述的PendingIntent已经存在，则在产生新的Intent之前会先取消当前的
PendingIntent hangPendingIntent = PendingIntent.getActivity(this,0,hangIntent,PendingIntent.FLAG_CANCEL_CURRENT);
builder.setFullScreenIntent(hangPendingIntent,true);

Notification notification = builder.build();

mManager.notify(2,notification);
```

效果如下：

![折叠式Notification](http://img.blog.csdn.net/20160312221613018)

##4.Notification的等级

Android5.0加入了一种新的模式Notification的等级，有三种：

*	VISIBILITY_PUBLIC 只有在没有锁屏时会显示通知
*	VISIBILITY_PRIVATE 任何情况都会显示通知
*	VISIBILITY_SECRET 在安全锁和没有锁屏的情况下显示通知

只需通过Builder的setVisibility方法就可以了

```
builder.setVisibility(Notification.VISIBILITY_PUBLIC);
```

##参考文章

[ Android5.x Notification应用解析 ](http://blog.csdn.net/itachi85/article/details/50096609)

