**Activity与Activity调用栈分析**

[TOC]

# 起源

Activity是与用户交互的第一接口，它提供了一个用户完成指令的窗口。当开发者创建Activity之后，通过调用setContentView(view)方法来给该Activity指定一个显示的界面，并以此为基础提供给用户交互的接口。系统采用Activity栈的方式来管理Activity。

# Activity形态

Activity一个最大的特点就是拥有多种形态，它可以在多种形态间进行切换，以此来控制自己的生命周期。

*	Active/Running  
这时候，Activity出于Activity栈的最顶层，可见，并与用户进行交互。

*	Pause  
当Activity失去焦点，被一个新的非全屏的Activity或者一个透明的Activity放置在栈顶时，Activity就转化为Paused形态。但它只是失去了与用户交互的能力，所有状态信息、成员变量都还保持着，只有在系统内存极低的情况下，才会被系统回收掉。

*	Stopped
如果一个Activity被另一个Activity完全覆盖，那么Activity就会进入Stopped形态。此时，它不再可见，但却依然保持了所有状态信息和成员变量。

*	Killed
当Activity被系统回收掉或者Activity从来没有创建过，Activity就出于Killed形态。

> 由此可见，用户的不同动作，会让Activity在这四种状态间切换。而开发者，虽然可以控制Activity如何“生”，却无法空着Activity何时“死”。

# 生命周期

谷歌给了我们一张图来揭示Activity的生命周期，他希望Activity能被开发者所控制，而不是变成一匹脱缰的野马。

![ALT TEXT](http://hi.csdn.net/attachment/201109/1/0_1314838777He6C.gif)

开发者当然不必实现所有的生命周期方法，但知道每一个生命周期状态的含义，可以让我们更好的掌控Activity，让它能更好地完成你所期望的效果。

上图中列举了Activity生命周期的全部状态，但其中只有三个状态是稳定的，而其他状态都是过渡状态，很快就会结束。

*	Resume  
这个状态，也就是1.2节中的Active/Running状态，此时，Activity出于Activity栈顶，处理用户的交互。

* Paused 
当Activity的一部分被挡住的时候进入这个状态，这个状态下的Activity不会接收用户输入。

*	Stopped
当Activity完全被覆盖时进入这个状态，此时Activity不可见，仅在后台运行。

## Acticity的启动和销毁过程

**Activity的启动和销毁过程：** 在系统调用`onCreate()`之后，就会马上调用`onStart()`，然后继续调用`onResume()`以进入Resumed状态，最后就会停在Resumed状态，完成启动。系统会调用onDestroy()来结束一个Activity的声明周期让它回到Killed形态。

onCreate()中：创建基本的UI元素。  
onPause()与onStop()：清除Activity的资源、避免浪费。  
onDestroy()中：因为引用会在Activity销毁的时候销毁，而线程不会，所以清除开启的线程。  

启动: onCreate() -> onStart() -> onResume()   
销毁: onPause() -> onStop() -> onDestroy()  

## Acticity的暂停与恢复过程

**Acticity的暂停与恢复过程:** 当栈顶的Activity部分不可见后，就会导致Activity进入Pause状态，此时就会调用onPause()方法，当结束阻塞后，就会调用onResume()方法来恢复到Resume状态。  

onPause()： 释放系统资源，如Camera、sensor、receivers。  
onResume()： 需要重新初始化onPause()中释放的资源。  

暂停：onResume() -> onPause()  
恢复：onPause()  -> onResume()  

## Activity的停止过程  

栈顶的Activity部分不可见时，实际上后续会有两可能，从部分不可见到部分可见，也就是恢复过程；从部分不可见到完全不可见，也就是停止过程。  

*	系统在当前Activity不可见的时候，总会调用onPause()方法。
*	系统在当前Activity由不可见到可见时，都会调用onStart()方法。

## Activity的重新创建过程

如果你的系统长时间处于stopped状态而且此时系统需要更多内存或者系统内存极为紧张时，系统就会回收你的Activity，而此时系统为了补偿你，会将Activity状态通过onSaveInstanceState()方法保存到Bundle对象中，当然你也可以增加额外的键值对存入Bundle对象以保存这些状态。当你需要重新创建这些Activi的时候，保存的Bundle对象就会传递到Activity的onRestoreInstanceState()方法与onCreate()方法中，这也就是onCreate()方法中参数---Bundle savaedInstanceState的来源。  

Activity重新创建过程:  

```
Resumed(visible) -------->onSavaInstanceState()  --->  Destroyed    
Created ------------>onRestoreInstanceState() ------>  Resumed(visible)
```

> **不过这里需要注意的是**onSaveInstanceState()方法并不是每次当Activity离开前台时都会调用的，如果用户使用finish()方法结束了Acticity，则不会调用。而且Android系统已经默认实现类控件的状态缓存，以此来减少开发者需要实现的缓存逻辑。  

#  Android任务栈简介

一个Android应用程序功能通常会被拆分为多个Activity，而各个Activity之间通过Intent进行连接，而Android系统，通过栈结构来保存整个App的Activity，栈底的元素是整个任务栈的发起者。一个合理的任务调度栈不仅是性能的保证，更是提供性能的基础。  

当一个App启动时，如果当前环境中部存在该App的任务栈，那么系统就会创建一个任务栈。此后，这个App所启动的Acticity都将在这个任务栈中被管理，这个栈也被称为一个Task，即表示若干个Activity的集合，他们组合在一起形成一个Task。另外，需要特别注意的是，**一个Task中的Activity可以来自不同的App，同一个App的Activity也肯能不在一个Task中**。  

根据Activity在当前栈结构的位置，来决定该Activity的状态。正常情况下的Android任务栈，当一个Activity启动了另一个Activity的时候，新启动的Activityjiuhui置于任务栈的顶端，并处于活动状态，而启动它的Activity虽然功成身退，但依然保留在任务栈中，处于停止恢复活动状态，当用户按下返回键或者调用finish()方法时，系统会移除顶部Activity，让后面的Activity恢复活动状态。

# AndroidMainifest启动模式

当然，世界不可能一直这么“和谐”，可以给Activity设置一些“特权”，来打破这种“和谐”的模式。这种特权，就是通过在AndroidManifest文件中的属性android:launchMode来设置或者是通过Intent的flag来设置的。  

Android设计者在AndroidManifest文件中设计了四种启动模式  

*	standard
*	singleTop
*	singleTask
*	singleInstance

## standard

默认的启动模式，如果不指定Activity的启动模式，则使用这种方式启动Activity。这种启动模式每次都会创建新的实例，每次点击standard模式创建Activity后，都会创建新的MainActivity覆盖在原Activity上。

## singleTop

如果指定启动Activity为singleTop模式，那么在启动时，系统会判断当前栈顶Activity是不是要启动的Activity，如果是则不创建新的Activity而直接引用这个Activity；如果不是则创建新的Activity。这种启动模式通常适应于接收到消息后显示的界面，例如QQ接收到消息后弹出Acticity，如果一次来10条消息，总不能一次弹出10个Activity。  

这种启动模式虽然不会创建新的实例，但是系统仍然会在Activity启动时调用onNewIntent()方法。例如：如果当前任务栈中有A、B、C三个Activity，而且C的启动模式是singleTop的，那么这时候如果再次启动C，那么系统的就不会创建新的实例，而是会调用C的onNewIntent()方法，当前任务栈中依然是A、B、C三个Activity；如果B的启动模式是singleTop的，那么这时候如果C启动的是B，则会创建一个全新的B，当前任务栈为A、B、C、B。

## singleTask

采用该加载模式时，Activity在同一个Task内只有一个实例。

singleTask与singleTop模式类似，只不过singleTop是检测栈顶元素是否是需要启动的Activity，而SingleTask是检测整个Activity栈中是否存在当前需要启动的Activity。如果存在，则将该Activity置于栈顶，并将该Activity以上的Activity都销毁。

*	如果要启动的Activity不存在，那么系统将会创建该实例，并将其加入Task栈顶  
*	如果将要启动的Activity已存在，且存在栈顶，那么此时与singleTop模式的行为相同
*	如果将要启动的Activity存在但是没有位于栈顶，那么此时系统会把位于该Activity上面的所有其他Activity全部移除Task，从而使得该目标Activity位于栈顶。

上述是指在同一个App中启动这个singleTask的Activity，如果是其他程序以singleTask模式来启动Activity，那么它将创建一个新的任务栈。**需要注意的是，**如果启动模式为singleTask的Activity已经在后台一个任务栈中了，那么启动后，后台的这个任务栈将会一起被切换到前台，如下图所示。

当Activity2启动ActivityY(启动模式为singleTask)时，它所在的Task都被切换到前台，且按返回键返回时，也会先返回ActivityY所在Task的Activity。

可以发现，使用这个模式创建的Activity不是在新的任务栈中被打开，就是将已打开的Activity切换到前台，所以这种启动模式通常可以用来退出整个应用：将主Activity设为singleTask模式，然后在要退出的Activity中转到主Activity，从而将主Activity之上的Activity都清除，然后重写主Activity的onNewIntent()方法，在方法中加入一句finish()，将最后一个Activity结束掉。

![ALT TEXT](http://img.mukewang.com/54b4b55d0001b4a705000281.jpg)

## singleInstance

在该加载模式下，无论哪个Task中启动目标Activity，只会创建一个目标Acticity实例且会用一个全新的Task栈来装载该Activity实例。

当系统采用singleInstance模式加载Activity时，分为以下两种情况：

*	如果将要启动的Activity不存在，那么系统将会先创建一个全新的Task，再创建目标Activity实例并将该Activity实例放入此全新的Task中
*	如果将要启动的Activity已经存在，那么无论它位于哪个应用程序、哪个Task中，系统都会把该Acticity所在的Task转到前台，从而使该Activity显示出来。

这种启动模式常用于需要与程序分离的界面，如在SetupWizard中调用紧急呼叫，就是使用这种启动模式。例如闹铃提醒，将闹铃提醒与闹铃设置分离


# Intent Flag启动模式

系统提供两种方式来设置一个Activity的启动模式，下面要介绍的就是通过设置Intent的Flag来设置一个Activity的启动模式。

*	Intent.FLAG_ACTIVITY_NEW_TASK

使用一个新的Task来启动一个Activity，但启动的每个Activity都将在一个新的Task中。该Flag通常使用在从Service中启动Activity的场景，由于在Service中并不存在Activity栈，所有使用该Flag来创建一个新的Activity栈，并创建新的Activity实例。

*	Intent.FLAG_ACTIVITY_SINGLE_TOP

使用singleTop模式来启动一个Activity，与制定android:lanchMode="singleTop"效果相同。

*	Intent.FLAG_ACTIVITY_CLEAR_TOP

使用singleTask模式来启动一个Activity，与制定android:lanchMode="singleTask"效果相同。

*	Intent.FLAG_ACTIVITY_NO_HISTORY

使用这种模式启动Activity，当该Acticity启动其他Activi后，该Activity就消失了，不会保留在Activity栈中，例如A-B，B中以这种模式启动C，C再启动D，则当前Activity栈为ABD。


# 清空任务栈

系统提供了清楚任务栈的方法将一个Task全部清楚。通常情况下，可以在AndroidManifest文件中<activity>标签中使用以下几种属性来清理任务栈。

*	clearTaskOnTouch

clearTaskOnTouch，就是在每次返回该Activity时，都将该Activity之上的所有Activity清除。通过这个属性，可以让这个Task每次在初始化的时候，都只有这一个Activity。

*	finishOnTaskLaunch

finishOnTaskLaunch与clearTaskOnTouch属性类似，只不过clearTaskOnTouch作用在别人身上，而finishOnTaskLaunch作用在自己身上。通过这个属性，当离开这个Activity所处的Task，那么用户再返回时，该Activity就会被finish掉。

*	alwaysRetainTaskState

alwaysRetainTaskState属性给Task一道“免试金牌”，如果将Activity的这个属性设置为True，那么该Activity所在的Task将不接受任何清理命令，一直爆出当前Task状态。



