**Android四大组件④BroadcastReceiver的注册发送和接收过程**

[TOC]

BroadcastReceiver 的注册分为两种，分别是静态注册和动态注册，静态注册是在 AndroidManifest 文件中注册，在应用安装时由 PackageManagerService 来完成注册过程；动态注册是由代码完成注册，这里介绍 BroadcastReceiver 的动态注册。

# 广播的注册过程

![广播注册过程时序图](http://onke0yoit.bkt.clouddn.com/广播注册过程时序图.png)

要动态注册 BroadcastReceiver，需要调用 registerReceiver 方法，它的实现在 ContextWrapper 中，代码如下所示

frameworks/base/core/java/android/content/ContextWrapper.java

```java
public Intent registerReceiver(
    BroadcastReceiver receiver, IntentFilter filter) {
    return mBase.registerReceiver(receiver, filter);
}
```
mBase 具体指向的是 ContextImpl，ContextImpl 的 registerReceiver 方法最终会调用 registerReceiverInternal 方法。

frameworks/base/core/java/android/app/ContextImpl.java

```java
private Intent registerReceiverInternal(BroadcastReceiver receiver, int userId,
          IntentFilter filter, String broadcastPermission,
          Handler scheduler, Context context) {
      IIntentReceiver rd = null;
      if (receiver != null) {
          if (mPackageInfo != null && context != null) {//1
              if (scheduler == null) {
                  scheduler = mMainThread.getHandler();
              }
              rd = mPackageInfo.getReceiverDispatcher(
                  receiver, context, scheduler,
                  mMainThread.getInstrumentation(), true);//2
          } else {
              if (scheduler == null) {
                  scheduler = mMainThread.getHandler();
              }
              rd = new LoadedApk.ReceiverDispatcher(
                      receiver, context, scheduler, null, true).getIIntentReceiver();//3
          }
      }
      try {
          final Intent intent = ActivityManagerNative.getDefault().registerReceiver(
                  mMainThread.getApplicationThread(), mBasePackageName,
                  rd, filter, broadcastPermission, userId);//4
          if (intent != null) {
              intent.setExtrasClassLoader(getClassLoader());
              intent.prepareToEnterProcess();
          }
          return intent;
      } catch (RemoteException e) {
          throw e.rethrowFromSystemServer();
      }
}
```
1. 判断 LoadedApk 类型 mPackageInfo 不等于 null 并且 context 不等于 null
2. 1 为 true 时调用，通过 mPackageInfo 的 getReceiverDispatcher 方法获取 rd 对象
3. 1 为 false 时调用，创建 rd 对象

2 和 3 的目的都是要获取 IIntentReceiver 类型的 rd 对象，IIntentReceiver 是一个 Binder 接口，用于进行跨进程的通信，它的具体实现在 LoadedApk.ReceiverDispatcher.InnerReceiver，如下所示

frameworks/base/core/java/android/app/LoadedApk.java

```java
static final class ReceiverDispatcher {
      final static class InnerReceiver extends IIntentReceiver.Stub {
          final WeakReference<LoadedApk.ReceiverDispatcher> mDispatcher;
          final LoadedApk.ReceiverDispatcher mStrongRef;
          InnerReceiver(LoadedApk.ReceiverDispatcher rd, boolean strong) {
              mDispatcher = new WeakReference<LoadedApk.ReceiverDispatcher>(rd);
              mStrongRef = strong ? rd : null;
          }
        ...  
       }
      ... 
 }
```
回到 registerReceiverInternal 方法，在 4 处调用了 ActivityManagerProxy 的 registerReceiver 方法，最终会调用 AMS 的 registerReceiver 方法，并将 rd 传递过去。

AMS 的 registerReceiver 方法如下：

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
  public Intent registerReceiver(IApplicationThread caller, String callerPackage,
            IIntentReceiver receiver, IntentFilter filter, String permission, int userId) {
...
 synchronized(this) {
  ...
            Iterator<String> actions = filter.actionsIterator();//1
  ...
            // Collect stickies of users
            int[] userIds = { UserHandle.USER_ALL, UserHandle.getUserId(callingUid) };
            while (actions.hasNext()) {
                String action = actions.next();
                for (int id : userIds) {
                    ArrayMap<String, ArrayList<Intent>> stickies = mStickyBroadcasts.get(id);
                    if (stickies != null) {
                        ArrayList<Intent> intents = stickies.get(action);
                        if (intents != null) {
                            if (stickyIntents == null) {
                                stickyIntents = new ArrayList<Intent>();
                            }
                            stickyIntents.addAll(intents);//2
                        }
                    }
                }
            }
        }
  ArrayList<Intent> allSticky = null;	
        if (stickyIntents != null) {
            final ContentResolver resolver = mContext.getContentResolver();
            for (int i = 0, N = stickyIntents.size(); i < N; i++) {
                Intent intent = stickyIntents.get(i);
                if (filter.match(resolver, intent, true, TAG) >= 0) {
                    if (allSticky == null) {
                        allSticky = new ArrayList<Intent>();
                    }
                    allSticky.add(intent);//3
                }
            }
        }
 ...       
}
```
1. 根据传入的 IntentFilter 类型的 filter 得到 actions 列表，根据 actions 列表和 userIds 得到所有的粘性广播的 intent。
2. 将 1 处的集合加入到 stickyIntents 中
3. 将这些粘性广播的 intent 加入到 allSticky 列表中，从这里可以看出粘性广播是存储在 AMS 中的。

接着查看 AMS 的 registerReceiver 方法的剩余内容：

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
  public Intent registerReceiver(IApplicationThread caller, String callerPackage,
            IIntentReceiver receiver, IntentFilter filter, String permission, int userId) {
...
        synchronized (this) {
          ...
            ReceiverList rl = mRegisteredReceivers.get(receiver.asBinder());//1
            if (rl == null) {
                rl = new ReceiverList(this, callerApp, callingPid, callingUid,
                        userId, receiver);//2
                if (rl.app != null) {
                    rl.app.receivers.add(rl);
                } 
                ...
            }
            ...
            BroadcastFilter bf = new BroadcastFilter(filter, rl, callerPackage,
                    permission, callingUid, userId);//3
            rl.add(bf);//4
            if (!bf.debugCheck()) {
                Slog.w(TAG, "==> For Dynamic broadcast");
            }
            mReceiverResolver.addFilter(bf);//5
            ...
            return sticky;
        }
}
```
1. 获取 ReceiverList 列表
2. 如果 ReceiverList 为 null，则创建，ReceiverList 继承自 ArrayList，用来存储广播接收者。
3. 创建 BroadcastFilter 并传入创建的 ReceiverList 对象，BroadcastFilter 用来描述注册的广播接收者
4. 通过 add 方法将自身添加到 ReceiverList 中
5. 将 BroadcastFilter 添加到 mReceiverResolver 中，这样当 AMS 接收到广播就可以从 mReceiverResolver 中找到对应的广播接收者。

# 广播的发送过程

![广播发送过程时序图](http://onke0yoit.bkt.clouddn.com/广播发送过程时序图.png)

广播可以发送多种类型，包括无序广播（普通广播）、有序广播和粘性广播，这里以无序广播为例，来分析广播的发送过程。

要发送无序广播需要调用 sendBroadcast 方法，它的实现同样在 ContextWrapper 中：

frameworks/base/core/java/android/content/ContextWrapper.java

```java
@Override
  public void sendBroadcast(Intent intent) {
      mBase.sendBroadcast(intent);
  }
```

mBase 指向的是 ContextImpl，ContextImpl 中的 sendBroadcast 方法如下：

frameworks/base/core/java/android/app/ContextImpl.java

```java
@Override
  public void sendBroadcast(Intent intent) {
      warnIfCallingFromSystemProcess();
      String resolvedType = intent.resolveTypeIfNeeded(getContentResolver());
      try {
          intent.prepareToLeaveProcess(this);
          ActivityManagerNative.getDefault().broadcastIntent(
                  mMainThread.getApplicationThread(), intent, resolvedType, null,
                  Activity.RESULT_OK, null, null, null, AppOpsManager.OP_NONE, null, false, false,
                  getUserId());//1
      } catch (RemoteException e) {
          throw e.rethrowFromSystemServer();
      }
  }
```
调用 AMS 的代理类 ActivityManagerProxy 的 broadcastIntent 方法，最终会调用 AMS 的 broadcastIntent 方法。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
public final int broadcastIntent(IApplicationThread caller,
          Intent intent, String resolvedType, IIntentReceiver resultTo,
          int resultCode, String resultData, Bundle resultExtras,
          String[] requiredPermissions, int appOp, Bundle bOptions,
          boolean serialized, boolean sticky, int userId) {
      enforceNotIsolatedCaller("broadcastIntent");
      synchronized(this) {
          intent = verifyBroadcastLocked(intent);//1
        ...
          /**
          * 2
          */
          int res = broadcastIntentLocked(callerApp,
                  callerApp != null ? callerApp.info.packageName : null,
                  intent, resolvedType, resultTo, resultCode, resultData, resultExtras,
                  requiredPermissions, appOp, bOptions, serialized, sticky,
                  callingPid, callingUid, userId);
          Binder.restoreCallingIdentity(origId);
          return res;
      }
  }
```

先看 verifyBroadcastLocked 方法：

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
final Intent verifyBroadcastLocked(Intent intent) {
       // Refuse possible leaked file descriptors
       if (intent != null && intent.hasFileDescriptors() == true) {//1
           throw new IllegalArgumentException("File descriptors passed in Intent");
       }
       int flags = intent.getFlags();//2
       if (!mProcessesReady) {
           if ((flags&Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT) != 0) {//3
           } else if ((flags&Intent.FLAG_RECEIVER_REGISTERED_ONLY) == 0) {//4
             
               throw new IllegalStateException("Cannot broadcast before boot completed");
           }
       }
       if ((flags&Intent.FLAG_RECEIVER_BOOT_UPGRADE) != 0) {
           throw new IllegalArgumentException(
                   "Can't use FLAG_RECEIVER_BOOT_UPGRADE here");
       }
       return intent;
   }
```
verifyBroadcastLocked 方法主要是验证广播是否合法。

1. 验证 Intent 是否为 null 并且有文件描述符。
2. 获取 Intent 中的flag。
3. 如果系统正在启动过程中，判断 flag，如果为 Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT（启动检查时只接受动态主持的广播接收者）则不做处理
4. 如果 flag 没有设置为 Intent.FLAG_RECEIVER_REGISTERED_ONLY（只接受动态注册的广播接收者），则会抛出异常

再回到 broadcastIntent 方法，2 处调用了 broadcastIntentLocked 方法，如下所示

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
 final int broadcastIntentLocked(ProcessRecord callerApp,
            String callerPackage, Intent intent, String resolvedType,
            IIntentReceiver resultTo, int resultCode, String resultData,
            Bundle resultExtras, String[] requiredPermissions, int appOp, Bundle bOptions,
            boolean ordered, boolean sticky, int callingPid, int callingUid, int userId) {
  ...
       if ((receivers != null && receivers.size() > 0)
                || resultTo != null) {
            BroadcastQueue queue = broadcastQueueForIntent(intent);
            /**
            * 1
            */
            BroadcastRecord r = new BroadcastRecord(queue, intent, callerApp,
                    callerPackage, callingPid, callingUid, resolvedType,
                    requiredPermissions, appOp, brOptions, receivers, resultTo, resultCode,
                    resultData, resultExtras, ordered, sticky, false, userId);
     ...               
            boolean replaced = replacePending && queue.replaceOrderedBroadcastLocked(r);
            if (!replaced) {
                queue.enqueueOrderedBroadcastLocked(r);
                queue.scheduleBroadcastsLocked();//2
            }
        } 
        ...
        }
        return ActivityManager.BROADCAST_SUCCESS;
}
```

这里省略了很多代码，前面的工作主要是将动态注册的广播接收者和静态注册的广播接收者按照优先级高低存储在不同的列表中，再将这两个列表合并到 receivers 列表中，这样 receivers 列表包含了所有的广播接收者（无序广播和有序广播）。

1. 创建 BroadcastRecord 对象并将 receivers 传进去。
2. 调用 BroadcastQueue 的 scheduleBroadcastLocked 方法。

# 广播的接收过程

![广播接收过程时序图](http://onke0yoit.bkt.clouddn.com/广播接收过程时序图.png)

BroadcastQueue 的 scheduleBroadcastLocked 方法代码如下：

```java
public void scheduleBroadcastsLocked() {
...
    mHandler.sendMessage(mHandler.obtainMessage(BROADCAST_INTENT_MSG, this));//1
    mBroadcastsScheduled = true;
}
```
向 BroadcastHandler 类型的 mHandler 对象发送了 BROADCAST_INTENT_MSG 类型的消息，这个消息在 BroadcastHandler 的 handleMessage 方法中进行处理。

frameworks/base/services/core/java/com/android/server/am/BroadcastQueue.java

```java
private final class BroadcastHandler extends Handler {
    public BroadcastHandler(Looper looper) {
        super(looper, null, true);
    }
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case BROADCAST_INTENT_MSG: {
                if (DEBUG_BROADCAST) Slog.v(
                        TAG_BROADCAST, "Received BROADCAST_INTENT_MSG");
                processNextBroadcast(true);
            } break;
       ...
        }
    }
}
```
在 handleMessage 方法中调用了 processNextBroadcast 方法，processNextBroadcast 方法对无序广播和有序广播分别进行处理，旨在将广播发送给广播接收者，下面给出 processNextBroadcast 方法中对无序广播的处理部分。

frameworks/base/services/core/java/com/android/server/am/BroadcastQueue.java

```java
final void processNextBroadcast(boolean fromMsg) {
...
          if (fromMsg) {
              mBroadcastsScheduled = false;//1
          }
          // First, deliver any non-serialized broadcasts right away.
          while (mParallelBroadcasts.size() > 0) {//2
              r = mParallelBroadcasts.remove(0);//3
             ...
              for (int i=0; i<N; i++) {
                Object target = r.receivers.get(i);
                deliverToRegisteredReceiverLocked(r, (BroadcastFilter)target, false, i);//4
              }
         ...
          }
}
```
fromMsg 的值为 true，因此 1 处会将 mBroadcastScheduled 设置为 false，表示对于此前发来的 BROADCAST_INTENT_MSG 类型的消息已经处理了。2 处的 mParallelBroadcasts 列表用来存储无序广播，通过 while 循环将 mParallelBroadcasts 列表中的无序广播发送给对于的广播接收者。3 处获取每一个 mParallelBroadcasts 列表中存储的 BroadcastRecord 类型的 r 对象。4 处将这些 r 对象描述的广播发送给对应的广播接收者，deliverToRegisteredReceiverLocked 方法如下：

frameworks/base/services/core/java/com/android/server/am/BroadcastQueue.java

```java
private void deliverToRegisteredReceiverLocked(BroadcastRecord r,
        BroadcastFilter filter, boolean ordered, int index) {
...
   try {
            if (DEBUG_BROADCAST_LIGHT) Slog.i(TAG_BROADCAST,
                    "Delivering to " + filter + " : " + r);
            if (filter.receiverList.app != null && filter.receiverList.app.inFullBackup) {
             ...   
            } else {
                performReceiveLocked(filter.receiverList.app, filter.receiverList.receiver,
                        new Intent(r.intent), r.resultCode, r.resultData,
                        r.resultExtras, r.ordered, r.initialSticky, r.userId);//1
            }
            if (ordered) {
                r.state = BroadcastRecord.CALL_DONE_RECEIVE;
            }
        } catch (RemoteException e) {
 ...
        }
}
```
中间省略了很多代码，这些代码时用来检查广播发送者和广播接收者的权限。如果通过了权限的检查，则会调用 1 处的 performReceiveLocked 方法。

frameworks/base/services/core/java/com/android/server/am/BroadcastQueue.java

```java
void performReceiveLocked(ProcessRecord app, IIntentReceiver receiver,
          Intent intent, int resultCode, String data, Bundle extras,
          boolean ordered, boolean sticky, int sendingUser) throws RemoteException {
      // Send the intent to the receiver asynchronously using one-way binder calls.
      if (app != null) {//1
          if (app.thread != null) {//2
              // If we have an app thread, do the call through that so it is
              // correctly ordered with other one-way calls.
              try {
                  app.thread.scheduleRegisteredReceiver(receiver, intent, resultCode,
                          data, extras, ordered, sticky, sendingUser, app.repProcState);//3          
              } 
          } 
          ...
      } else {
          receiver.performReceive(intent, resultCode, data, extras, ordered,
                  sticky, sendingUser);/
      }
  }
```
1. 判断广播接收者所在的应用程序是否存在
2. 判断广播接收者所在的应用程序进程是否正在进行
3. 用广播接收者所在的应用程序进程来接收广播，app.thread 指的是 ApplicationThread。

frameworks/base/core/java/android/app/ActivityThread.java

```java
public void scheduleRegisteredReceiver(IIntentReceiver receiver, Intent intent,
         int resultCode, String dataStr, Bundle extras, boolean ordered,
         boolean sticky, int sendingUser, int processState) throws RemoteException {
     updateProcessState(processState, false);
     receiver.performReceive(intent, resultCode, dataStr, extras, ordered,
             sticky, sendingUser);//1
 }
```
调用 IIntentReceiver 类型的对象 receiver 的 performReceive 方法，这里实现 receiver 的类为 LoadedApk.ReceiverDispatcher.InnerReceiver，如下所示

frameworks/base/core/java/android/app/LoadedApk.java

```java
  static final class ReceiverDispatcher {
        final static class InnerReceiver extends IIntentReceiver.Stub {
        ...
            InnerReceiver(LoadedApk.ReceiverDispatcher rd, boolean strong) {
                mDispatcher = new WeakReference<LoadedApk.ReceiverDispatcher>(rd);
                mStrongRef = strong ? rd : null;
            }
            @Override
            public void performReceive(Intent intent, int resultCode, String data,
                    Bundle extras, boolean ordered, boolean sticky, int sendingUser) {
                final LoadedApk.ReceiverDispatcher rd;
                ...
                if (rd != null) {
                    rd.performReceive(intent, resultCode, data, extras,
                            ordered, sticky, sendingUser);//1
                } else {
             ...
            }
...
}
```
调用 LoadedApk.ReceiverDispatcher 类型的 rd 对象的 performReceive 方法

frameworks/base/core/java/android/app/LoadedApk.java

```java
public void performReceive(Intent intent, int resultCode, String data,
              Bundle extras, boolean ordered, boolean sticky, int sendingUser) {
          final Args args = new Args(intent, resultCode, data, extras, ordered,
                  sticky, sendingUser);//1
          ...
          if (intent == null || !mActivityThread.post(args)) {//2
              if (mRegistered && ordered) {
                  IActivityManager mgr = ActivityManagerNative.getDefault();
                  if (ActivityThread.DEBUG_BROADCAST) Slog.i(ActivityThread.TAG,
                          "Finishing sync broadcast to " + mReceiver);
                  args.sendFinished(mgr);
              }
          }
      }
```
1. 将广播的 intent 等信息封装成 Args 对象
2. 调用 mActivityThread 的 post 方法并传入了 Args 对象，这个 mActivityThread 是一个 Handler 对象，具体指向就是 H，作用就是将 Args 对象通过 H 发送到主线程的消息队列中。Args 继承自 Runnable，这个消息最终会在 Args 的 run 方法执行。

frameworks/base/core/java/android/app/LoadedApk.java

```java
public void run() {
    final BroadcastReceiver receiver = mReceiver;
  ...
     try {
         ClassLoader cl =  mReceiver.getClass().getClassLoader();
         intent.setExtrasClassLoader(cl);
         intent.prepareToEnterProcess();
         setExtrasClassLoader(cl);
         receiver.setPendingResult(this);
         receiver.onReceive(mContext, intent);//1
     } catch (Exception e) {
        ...
     }
    ...
 }
```
执行广播接收者的 onReceive 方法，这样注册的广播接收者就收到了广播并得到了 intent。

以上就是广播的注册、发送和接收过程。

# 参考资料

* [http://liuwangshu.cn/framework/component/4-broadcastreceiver.html](http://liuwangshu.cn/framework/component/4-broadcastreceiver.html)

