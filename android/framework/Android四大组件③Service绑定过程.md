**Android四大组件③Service绑定过程**

[TOC]

Service 启动方式有两种，一是调用 Context 的 startService 来启动 Service，二是调用 Context 的 bindService 来绑定 Service。这篇文章来分析第二种方式，实际上绑定 Service 的过程也包含了第一种方式中启动 Service 的过程。

下面分三个步骤来分析 Service 绑定过程：

# ContextImpl 到 AMS的调用过程

![bindservice-ContextImpl到AMS调用过程](http://onke0yoit.bkt.clouddn.com/bindservice-ContextImpl到AMS调用过程.png)

首先调用 Context 的 bindService 方法绑定 Service，它的实现在 ContextWrapper 中。

frameworks/base/core/java/android/content/ContextWrapper.java

```java
    @Override
    public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {
        return mBase.bindService(service, conn, flags);
    }
```

mBase 具体就是指的是 ContextImpl，我们在 Service 的启动过程已经分析了，下面查看 ContextImpl 的 bindService 方法。

frameworks/base/core/java/android/app/ContextImpl.java

```java
    @Override
    public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {
        warnIfCallingFromSystemProcess();
        return bindServiceCommon(service, conn, flags, mMainThread.getHandler(),
                Process.myUserHandle());
    }
```
在 bindService 方法中，调用了 bindServiceCommon 方法，并返回 bindServiceCommon 方法的返回值。

frameworks/base/core/java/android/app/ContextImpl.java

```java
private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags, Handler
        handler, UserHandle user) {
    IServiceConnection sd;
    if (conn == null) {
        throw new IllegalArgumentException("connection is null");
    }
    if (mPackageInfo != null) {
        sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(), handler, flags);//1
    } else {
        throw new RuntimeException("Not supported in system context");
    }
    validateServiceIntent(service);
    try {
     ...
     /**
     * 2
     */
        int res = ActivityManagerNative.getDefault().bindService(
            mMainThread.getApplicationThread(), getActivityToken(), service,
            service.resolveTypeIfNeeded(getContentResolver()),
            sd, flags, getOpPackageName(), user.getIdentifier());
      ...
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}

```

1. 调用了 LoadedApk 类型的对象 mPackageInfo 的 getServiceDispatcher 方法，它的主要作用是将 ServiceConnection 封装为 IServiceConnection 类型的对象 sd，IServiceConnection 实现了 Binder 机制，所以 Service 的绑定就支持跨进程。
2. 通过调用 AMS 的代理对象 ActivityManagerProxy 的 bindService 方法，最终会调用 AMS 的 bindService 方法。

# Service 的绑定过程

![Service的绑定过程](http://onke0yoit.bkt.clouddn.com/Service的绑定过程.png)

AMS 的 bindService 方法如下所示

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
    public int bindService(IApplicationThread caller, IBinder token, Intent service,
            String resolvedType, IServiceConnection connection, int flags, String callingPackage,
            int userId) throws TransactionTooLargeException {
        enforceNotIsolatedCaller("bindService");

        // Refuse possible leaked file descriptors
        if (service != null && service.hasFileDescriptors() == true) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }

        if (callingPackage == null) {
            throw new IllegalArgumentException("callingPackage cannot be null");
        }

        synchronized(this) {
            return mServices.bindServiceLocked(caller, token, service,
                    resolvedType, connection, flags, callingPackage, userId);
        }
    }
```

bindService 最后会调用 ActiveService 类型的对象 mServices 的 bindServiceLocked 方法。

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

```java
 int bindServiceLocked(IApplicationThread caller, IBinder token, Intent service,
            String resolvedType, final IServiceConnection connection, int flags,
            String callingPackage, final int userId) throws TransactionTooLargeException {
 ...
 if ((flags&Context.BIND_AUTO_CREATE) != 0) {
                s.lastActivity = SystemClock.uptimeMillis();
                /**
                *  1
                */
                if (bringUpServiceLocked(s, service.getFlags(), callerFg, false,
                        permissionsReviewRequired) != null) {
                    return 0;
                }
            }
          ...
            if (s.app != null && b.intent.received) {//2
                try {
                    c.conn.connected(s.name, b.intent.binder);//3
                } catch (Exception e) {
                ...
                }
                if (b.intent.apps.size() == 1 && b.intent.doRebind) {//4
                    requestServiceBindingLocked(s, b.intent, callerFg, true);//5
                }
            } else if (!b.intent.requested) {//6
                requestServiceBindingLocked(s, b.intent, callerFg, false);//7
            }
            getServiceMap(s.userId).ensureNotStartingBackground(s);
        } finally {
            Binder.restoreCallingIdentity(origId);
        }
        return 1;
}
```

1. 调用 bringUpServiceLocked 方法，bringUpServiceLocked 方法中又会调用 realStartServiceLocked 方法，最终由 ActivityThread 调用 Service 的 onCreate 方法启动 Service。详细分析见文章 Service 的启动过程。
2. `s.app != null`表示 Service 已经运行，其中 s 是 ServiceRecord 对象，app 是 ProcessRecord 对象。`b.intent.received`表示当前应用程序的 Client 端已经接收到绑定 Service 时返回的 Binder，这样应用程序进程的 Client 端就可以通过 Binder 来获取要绑定的 Service 的访问接口。
3. 调用 c.conn 的 connected 方法，其中 c.conn 指的是 IServiceConnection，它的具体实现为 ServiceDispatcher.InnerConnection，其中 ServiceDispatcher 是 LoadedApk 的内部类，InnerConnection 的 connected 方法内部会调用 H 的 post 方法向主线程发送消息，从而解决当前应用程序和 Service 跨进程通信的问题。
4. 判断当前应用程序进程的 Client 端是否第一次与 Service 进行绑定，并且 Service 已经调用过 onUnBind 方法
5. 4 为 true 则运行。
6. 判断当前应用程序进程的 Client 端是否没有发送过绑定 Service 的请求。
7. 6 为 true 则运行

这里假设为第一次绑定 Service，则运行 7 处的代码，下面来看 requestServiceBindingLocked 方法：

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

```java
private final boolean requestServiceBindingLocked(ServiceRecord r, IntentBindRecord i,
        boolean execInFg, boolean rebind) throws TransactionTooLargeException {
   ...
    if ((!i.requested || rebind) && i.apps.size() > 0) {//1
        try {
            bumpServiceExecutingLocked(r, execInFg, "bind");
            r.app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_SERVICE);
            r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind,
                    r.app.repProcState);//2
           ...
        } 
        ...
    }
    return true;
}
```

1. i.requested 表示是否发送过绑定 Service 的请求，上面可知 !i.requested = true，rebind 值为 false ，那么（! i.requested || rebind）的值为 true。如果 IntentBindRecord 中的应用程序记录大于 0，则调用 2.
2. r.app.thread 的类型为 IApplicationThread，它的实现是 ActivityThread 的内部类 ApplicationThread，即调用 ApplicationThread 的 scheduleBindService 方法。

frameworks/base/core/java/android/app/ActivityThread.java

```java
        public final void scheduleBindService(IBinder token, Intent intent,
                boolean rebind, int processState) {
            updateProcessState(processState, false);
            BindServiceData s = new BindServiceData();
            s.token = token;
            s.intent = intent;
            s.rebind = rebind; //false

            if (DEBUG_SERVICE)
                Slog.v(TAG, "scheduleBindService token=" + token + " intent=" + intent + " uid="
                        + Binder.getCallingUid() + " pid=" + Binder.getCallingPid());
            sendMessage(H.BIND_SERVICE, s);
        }
```

首先将 Service 信息封装成 BindServiceData 对象，接着将次对象传入到 sendMessage 方法中。sendMessage 向 H 发送消息。

```java
public void handleMessage(Message msg) {

    switch (msg.what) {
     case BIND_SERVICE:
     handleBindService((BindServiceData)msg.obj);
     break;
    }

}
```

H 接收到 BIND_SERVICE 类型消息时，会在 handleMessage 方法中会调用 handleBindService 方法。

frameworks/base/core/java/android/app/ActivityThread.java

```java
private void handleBindService(BindServiceData data) {
       Service s = mServices.get(data.token);//1
       if (DEBUG_SERVICE)
           Slog.v(TAG, "handleBindService s=" + s + " rebind=" + data.rebind);
       if (s != null) {
           try {
               data.intent.setExtrasClassLoader(s.getClassLoader());
               data.intent.prepareToEnterProcess();
               try {
                   if (!data.rebind) {//2
                       IBinder binder = s.onBind(data.intent);//3
                       ActivityManagerNative.getDefault().publishService(
                               data.token, data.intent, binder);//4
                   } else {
                       s.onRebind(data.intent);//5
                       ActivityManagerNative.getDefault().serviceDoneExecuting(
                               data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
                   }
                   ensureJitEnabled();
               } 
               ...
           } 
           ...
       }
   }
```
1. 获取到绑定的 Service。
2. BindServiceData 对象的成员变量 rebind 的值为 false，所以会调用 3
3. 调用 Service 的 onBind 方法，这样 Service 就处于绑定状态了。
4. 实际上最终调用 AMS 的 publishService 方法
5. 如果 rebind 的值为 true，就会调用 Service 的 onRebind 

下面接着看 AMS 的 publishService 方法。

# AMS 调用 ServiceConnection 过程

![AMS到ServiceConnection过程](http://onke0yoit.bkt.clouddn.com/AMS到ServiceConnection过程.png)

```java
    public void publishService(IBinder token, Intent intent, IBinder service) {
        // Refuse possible leaked file descriptors
        if (intent != null && intent.hasFileDescriptors() == true) {
            throw new IllegalArgumentException("File descriptors passed in Intent");
        }

        synchronized(this) {
            if (!(token instanceof ServiceRecord)) {
                throw new IllegalArgumentException("Invalid service token");
            }
            mServices.publishServiceLocked((ServiceRecord)token, intent, service);
        }
    }
```

publishService 调用了 ActiveService 类型的 mServices 对象的 publishServiceLocked

```java
void publishServiceLocked(ServiceRecord r, Intent intent, IBinder service) {
       final long origId = Binder.clearCallingIdentity();
       try {
          ...
                   for (int conni=r.connections.size()-1; conni>=0; conni--) {
                       ArrayList<ConnectionRecord> clist = r.connections.valueAt(conni);
                       for (int i=0; i<clist.size(); i++) {
                        ...
                           try {
                               c.conn.connected(r.name, service);//1
                           } catch (Exception e) {
                            ...
                           }
                       }
                   }
               }
               serviceDoneExecutingLocked(r, mDestroyingServices.contains(r), false);
           }
       } finally {
           Binder.restoreCallingIdentity(origId);
       }
   }
```
1. c.conn 指的是 IServiceConnection，它的具体实现为 ServiceDispatcher.InnerConnection，其中 ServiceDispatcher 是 LoadedApk 的内部类，ServiceDispatcher.InnerConnection 的 connected 方法如下

frameworks/base/core/java/android/app/LoadedApk.java

```java
static final class ServiceDispatcher {
     ...
        private static class InnerConnection extends IServiceConnection.Stub {
            final WeakReference<LoadedApk.ServiceDispatcher> mDispatcher;
            InnerConnection(LoadedApk.ServiceDispatcher sd) {
                mDispatcher = new WeakReference<LoadedApk.ServiceDispatcher>(sd);
            }
            public void connected(ComponentName name, IBinder service) throws RemoteException {
                LoadedApk.ServiceDispatcher sd = mDispatcher.get();
                if (sd != null) {
                    sd.connected(name, service);//1
                }
            }
        }
 ...
 }
```
调用 1 处的 ServiceDispatcher 类型的 sd 对象的 connected 方法。

```java
public void connected(ComponentName name, IBinder service) {
           if (mActivityThread != null) {
               mActivityThread.post(new RunConnection(name, service, 0));//1
           } else {
               doConnected(name, service);
           }
       }
```
调用 Handler 类型的对象 mActivityThread 的 post
方法，mActivityThread 实际上指向的是 H。因此通过调用 H 的 post 方法将 RunConnection 对象的内容运行在主线程中。RunConnection 的定义如下

frameworks/base/core/java/android/app/LoadedApk.java

```java
private final class RunConnection implements Runnable {
      RunConnection(ComponentName name, IBinder service, int command) {
          mName = name;
          mService = service;
          mCommand = command;
      }
      public void run() {
          if (mCommand == 0) {
              doConnected(mName, mService);
          } else if (mCommand == 1) {
              doDeath(mName, mService);
          }
      }
      final ComponentName mName;
      final IBinder mService;
      final int mCommand;
  }
```

RunConnection 的 run 方法调用了 doConnected 方法。

```java
public void doConnected(ComponentName name, IBinder service) {
  ...
    // If there was an old service, it is not disconnected.
    if (old != null) {
        mConnection.onServiceDisconnected(name);
    }
    // If there is a new service, it is now connected.
    if (service != null) {
        mConnection.onServiceConnected(name, service);//1
    }
}
```
doConnected 中调用 ServiceConnection 类型的对象 mConnection 的 onServiceConnected 方法，这样在客户端中实现了 ServiceConnection 接口的类的 onServiceConnected 方法就会被执行。

>上面就是 Service 绑定的全部过程。

2017.11.22 16:22


# 参考资料

* [Android深入四大组件（三）Service的绑定过程
](http://liuwangshu.cn/framework/component/3-service-bind.html)

