EventBus使用详解
====

## 概述

EventBus是一个Android事件发布/订阅框架，通过解耦发布者和订阅者简化Android事件传递，这里的事件可以理解为消息。事件传递既可以用于Android四大组件间通讯，也可以用于异步线程和主线程间通讯等。
传统的事件传递方式包括：Handler、BroadcastReceiver、Interface回调，相比之下EventBus的有点是代码简洁，使用简单，并将事件发布和 订阅充分解耦。

## 概念

**事件Event： **又可成为消息，其实就是一个对象，可以是网络请求返回的字符串，也可以是某个开关状态等等。事件类型EventType是指事件所属的Class。

事件分为一般事件和Sticky事件，相对于一般事件，Sticky事件不同之处在于，当事件发布后，再有订阅者开始订阅该类型事件，依然能收到该类型事件的最近一个Sticky事件。

**订阅者Subscriber： **订阅某种事件类型的对象，当有发布者发布这类事件后，EventBus会执行订阅者的onEvent函数，这个函数叫事件响应函数。订阅者通过register接口订阅某个事件类型，unregister接口退订。订阅者存在优先级，优先级高的订阅者可以取消事件继续向优先级低的订阅者分发，默认所有订阅者优先级都为0。

**发布者Publisher： **发布某事件的对象，通过post接口发布事件。

## GitHub地址

[EventBus源码：https://github.com/greenrobot/EventBus](https://github.com/greenrobot/EventBus)

## 基本使用

### 自定义一个事件类

```Java
public class AnyEventType {
     public AnyEventType(){}
 }
```
### 在要接受消息的页面注册

```Java
EventBus.getDefault().register(this);
```

### 接收消息的方法

```Java
@Subscribe
public void onEvent(AnyEventType event) {/* Do something */};
```

### 发送消息

```Java
EventBus.getDefault().post(event);
```

### 取消注册

```Java
EventBus.getDefault().unregister(this);
```

## 实例

下面我们来实现一个具体的例子来介绍EventBus的基本使用。

需求如下：在MainActivity中注册EventBus事件，并实现事件响应方法，当点击MainActivity中的按钮时跳转到SecondActivity，当点击SecondActivity中的按钮时向MainActivity发送Event事件，当MainActivity收到事件后，将事件内容显示在TextView中。

1. MainActivity
![MainActivity](http://7xs7a3.com1.z0.glb.clouddn.com/EventBus_before.png)

2. SecondActivity
![](http://7xs7a3.com1.z0.glb.clouddn.com/EnventBus_middle.png)

3. 事件处理
![](http://7xs7a3.com1.z0.glb.clouddn.com/EventBus_Afer.png)

### 事件类Event

```Java
public class Event {

    private String messgae;

    public Event(String messgae) {
        this.messgae = messgae;
    }

    public String getMessgae() {
        return messgae;
    }
}
```

### MainActivity

在OnCreate()函数中注册EventBus，在Ondestroy()函数中反注册。

```Java
public class MainActivity extends AppCompatActivity {

    @Bind(R.id.btn_open)
    Button mOpenBtn;

    @Bind(R.id.tv_showinfo)
    TextView mInfoTxt;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ButterKnife.bind(this);
        //注册
        EventBus.getDefault().register(this);

    }

    /**
     * 事件响应方法
     * 接收消息
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onEvent(Event event) {

        String msg = event.getMessgae();
        mInfoTxt.setText(msg);
    }

    //绑定点击事件
    @OnClick(R.id.btn_open)
    public void openSecondActivity(View view) {
            Intent intent = new Intent(MainActivity.this, SecondActivity.class);
            startActivity(intent);

    }


    @Override
    protected void onDestroy() {
        super.onDestroy();
        //反注册
        EventBus.getDefault().unregister(this);
    }
}
```

### SecondActivity

```Java
public class SecondActivity extends AppCompatActivity {

    @Bind(R.id.btn_post)
    Button mPostBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);

        ButterKnife.bind(this);


        mPostBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        //发送事件
                        EventBus.getDefault().post(new Event("Just do it"));
                    }
                }).start();

            }
        });
    }

}
```

## EventBus的事件订阅函数

### 类别

在上面的例子中，我们再注解`@Subscribe(threadMode = ThreadMode.MAIN)`中使用了ThreadMode.MAIN这个模式，表示该函数在主线程即UI线程中执行，实际上EventBus总共有四种线程模式，分别是：

-	ThreadMode.MAIN：表示无论事件是在哪个线程发布出来的，该事件订阅方法onEvent都会在UI线程中执行，这个在Android中是非常有用的，因为在Android中只能在UI线程中更新UI，所有在此模式下的方法是不能执行耗时操作的。

-	ThreadMode.POSTING：表示事件在哪个线程中发布出来的，事件订阅函数onEvent就会在这个线程中运行，也就是说发布事件和接收事件在同一个线程。使用这个方法时，在onEvent方法中不能执行耗时操作，如果执行耗时操作容易导致事件分发延迟。

-	ThreadMode.BACKGROUND：表示如果事件在UI线程中发布出来的，那么订阅函数onEvent就会在子线程中运行，如果事件本来就是在子线程中发布出来的，那么订阅函数直接在该子线程中执行。

-	ThreadMode.AYSNC：使用这个模式的订阅函数，那么无论事件在哪个线程发布，都会创建新的子线程来执行订阅函数。

## 实战

### 如何调用不同的订阅函数

要调用四种不同模式的订阅函数，我们首先要用清楚EventBus是如何指定调用的函数的？

先回顾一下上一节中的例子是如何调用订阅函数onEvent的，首先新建一个事件类：

```Java
public class Event {

    private String messgae;

    public Event(String messgae) {
        this.messgae = messgae;
    }

    public String getMessgae() {
        return messgae;
    }
}

```

发布事件：
```Java
EventBus.getDefault().post(new Event("Just do it"));
```

订阅事件：

```Java
    /**
     * 事件响应方法
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onEventMain(Event event) {

        String msg = event.getMessgae();
        mInfoTxt.setText(msg);
    }
```

观察可以发现：发布事件中的参数是Event的实例，而订阅函数中的参数也是Event的实例，可以推断EventBus是通过post函数传进去的类的实例来确定调用哪个订阅函数的，是哪个就调用哪个，如果有多个订阅函数呢，那么这些函数都会被调用！

### 示例

下面我们来验证这个推断：

我们在基本使用章节的例子上进行扩展，首先建立四个类：FirstEvent、SecondEvent、ThirdEvent、FourthEvent。

FirstEvent.java

```Java
public class FirstEvent {

    private String messgae;

    public FirstEvent(String messgae) {
        this.messgae = messgae;
    }

    public String getMessgae() {
        return messgae;
    }
}
```

SecondEvent.java

```Java
public class SecondEvent {

    private String messgae;

    public SecondEvent(String messgae) {
        this.messgae = messgae;
    }

    public String getMessgae() {
        return messgae;
    }
}
```

ThirdEvent.java

```Java
public class ThirdEvent {

    private String messgae;

    public ThirdEvent(String messgae) {
        this.messgae = messgae;
    }

    public String getMessgae() {
        return messgae;
    }
}
```

FourthEvent.java

```Java
public class FourthEvent {

    private String messgae;

    public FourthEvent(String messgae) {
        this.messgae = messgae;
    }

    public String getMessgae() {
        return messgae;
    }
}
```
然后在MainActivity中，增加四种模式的订阅函数

```Java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Bind(R.id.btn_open)
    Button mOpenBtn;

    @Bind(R.id.tv_showinfo)
    TextView mInfoTxt;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ButterKnife.bind(this);
        //注册
        EventBus.getDefault().register(this);

    }

    /**
     * 事件响应方法
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onEventMain(FirstEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventMain: " + event.getMessgae());
    }

    @Subscribe(threadMode = ThreadMode.POSTING)
    public void onEventPosting(SecondEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventPosting: "+ event.getMessgae());
    }

    @Subscribe(threadMode = ThreadMode.BACKGROUND)
    public void onEventBackgroud(ThirdEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventBackgroud: " + event.getMessgae());
    }

    @Subscribe(threadMode = ThreadMode.ASYNC)
    public void onEventAsync(FourthEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventAsync: " + event.getMessgae());
    }

    //绑定点击事件
    @OnClick(R.id.btn_open)
    public void openSecondActivity(View view) {
            Intent intent = new Intent(MainActivity.this, SecondActivity.class);
            startActivity(intent);

    }


    @Override
    protected void onDestroy() {
        super.onDestroy();
        //反注册
        EventBus.getDefault().unregister(this);
    }
}
```

接下来在SecondActivity中增加四个按钮，分别发送不同类别的事件

```Java
public class SecondActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);

        ButterKnife.bind(this);
    }

    @OnClick(R.id.btn_post)
    public void onPostA() {
        EventBus.getDefault().post(new FirstEvent("FirstEvent"));
    }

    @OnClick(R.id.btn_post2)
    public void onPostB() {

        EventBus.getDefault().post(new SecondEvent("SecondEvent"));
    }

    @OnClick(R.id.btn_post3)
    public void onPostC() {

        EventBus.getDefault().post(new ThirdEvent("ThirdEvent"));
    }

    @OnClick(R.id.btn_post4)
    public void onPostD() {

        EventBus.getDefault().post(new FourthEvent("FourthEvent"));
    }
}
```
运行后，分别顺序点击SecondActivity的四个按钮，打印信息如下：

```
03-31 02:53:45.950 4779-4779/com.example.michael.eventbusdemo I/MainActivity: onEventMain: FirstEvent
03-31 02:53:47.528 4779-4779/com.example.michael.eventbusdemo I/MainActivity: onEventPosting: SecondEvent
03-31 02:53:48.882 4779-4940/com.example.michael.eventbusdemo I/MainActivity: onEventBackgroud: ThirdEvent
03-31 02:53:50.462 4779-4940/com.example.michael.eventbusdemo I/MainActivity: onEventAsync: FourthEvent
```
>由此可见，通过发布不同的事件类的实例，EventBus根据类的实例分别调用了不同的订阅函数来处理事件。

那么，当同一个类的实例有多个函数订阅时，结果会是怎样呢？答案是，这些函数都会执行。下面我们来验证一下，将MainActivity中订阅函数的参数都改为FirstEvent，代码如下:

```Java
   /**
     * 事件响应方法
     * @param event
     */
    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onEventMain(FirstEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventMain: " + event.getMessgae());
    }

    @Subscribe(threadMode = ThreadMode.POSTING)
    public void onEventPosting(FirstEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventPosting: "+ event.getMessgae());
    }

    @Subscribe(threadMode = ThreadMode.BACKGROUND)
    public void onEventBackgroud(FirstEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventBackgroud: " + event.getMessgae());
    }

    @Subscribe(threadMode = ThreadMode.ASYNC)
    public void onEventAsync(FirstEvent event) {

        String msg = event.getMessgae();
        Log.i(TAG, "onEventAsync: " + event.getMessgae());
    }
```

运行程序，点击SecondActivity的FirstEvent按钮，打印信息如下：

```
03-31 03:14:07.032 23611-23746/com.example.michael.eventbusdemo I/MainActivity: onEventAsync: FirstEvent
03-31 03:14:07.033 23611-23611/com.example.michael.eventbusdemo I/MainActivity: onEventMain: FirstEvent
03-31 03:14:07.033 23611-23611/com.example.michael.eventbusdemo I/MainActivity: onEventPosting: FirstEvent
03-31 03:14:07.034 23611-23748/com.example.michael.eventbusdemo I/MainActivity: onEventBackgroud: FirstEvent
```
>分析可知，当SecondActivity发送FirstEvent事件过来的时候，这个四个订阅函数会同时接收到这个事件并执行。

**总结： **订阅函数的执行是根据参数中的事件类的类名来决定的。


