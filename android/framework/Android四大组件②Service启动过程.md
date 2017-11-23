**Android四大组件②Service启动过程**

[TOC]

启动一个 Service，我们需要在 Activity 中调用 `startService()`方法，Service 启动就从这个方法开始，下面分两个部分来分析 Service 启动的过程。一是 ContextWrapper 到 AMS 的调用过程，而是 AMS 启动 Service 过程。

# ContextWrapper 到 AMS 的调用过程

先看 ContextWrapper 到 AMS 的调用过程的时序图。

![ContextWrapper到AMS过程](http://onke0yoit.bkt.clouddn.com/ContextWrapper到AMS过程.png)

要启动 Service，我们需要调用 startService 方法，它的实现在 ContentWrapper 中，如下所示

frameworks/base/core/java/android/content/ContextWrapper.java

```java
    @Override
    public ComponentName startService(Intent service) {
        return mBase.startService(service);
    }
```

在 startService 方法中会调用 mBase 的 startService 方法，Content 类型的 mBase 对具体指的是什么呢？我们来看看 ActivityThread 启动 Activity 时的过程，这中间会创建 Activity 的上下文环境。

frameworks/base/core/java/android/app/ActivityThread.java

```java
 private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
  ...
            if (activity != null) {
                Context appContext = createBaseContextForActivity(r, activity);//1
         ...
                }
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window);
                ...
        }
        return activity;
}
```
>1 处创建上下文对象 appContext，并传入 Activity 的 attach 方法中，将 Activity 与上下文对象 appContent 关联起来，这个上下文对象 appContext 的具体类型是什么，我们接着查看 createBaseContentForActivity 方法，如下所示：

frameworks/base/core/java/android/app/ActivityThread.java

```java
private Context createBaseContextForActivity(ActivityClientRecord r, final Activity activity) {
...
    ContextImpl appContext = ContextImpl.createActivityContext(
            this, r.packageInfo, r.token, displayId, r.overrideConfig);
    appContext.setOuterContext(activity);
    Context baseContext = appContext;
    ...
    return baseContext;
}
```
>上面代码可得知，上下文对象 appContext 的具体类型就是 ContextImpl。Activity 的 attach 方法中将 ContentImpl 赋值给 ContentWrapper 的成员变量 mBase 中，因此，mBase 具体指向就是 ComtextImpl。

接下来查看 ComtextImpl 的 startService 方法，如下所示

frameworks/base/core/java/android/app/ContextImpl.java

```java
    @Override
    public ComponentName startService(Intent service) {
        warnIfCallingFromSystemProcess();
        return startServiceCommon(service, mUser);
    }
    
   private ComponentName startServiceCommon(Intent service, UserHandle user) {
   try {
       validateServiceIntent(service);
       service.prepareToLeaveProcess(this);
       //1
       ComponentName cn = ActivityManagerNative.getDefault().startService(
           mMainThread.getApplicationThread(), service, service.resolveTypeIfNeeded(
                       getContentResolver()), getOpPackageName(), user.getIdentifier());
       if (cn != null) {
           if (cn.getPackageName().equals("!")) {
               throw new SecurityException(
                       "Not allowed to start service " + service
                       + " without permission " + cn.getClassName());
           } else if (cn.getPackageName().equals("!!")) {
               throw new SecurityException(
                       "Unable to start service " + service
                       + ": " + cn.getClassName());
           }
       }
       return cn;
   } catch (RemoteException e) {
       throw e.rethrowFromSystemServer();
   }
}
```

startService 方法中会返回 startServiceCommon 方法，1 处调用 AMS 的代理对象 ActivityManagerProxy 的 startService 方法，最终会调用 AMS 的 startService 方法。

frameworks/base/core/java/android/app/ActivityManagerNative.java

```java
class ActivityManagerProxy implements IActivityManager {
···
    public ComponentName startService(IApplicationThread caller, Intent service,
            String resolvedType, String callingPackage, int userId) throws RemoteException
    {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IActivityManager.descriptor);
        data.writeStrongBinder(caller != null ? caller.asBinder() : null);
        service.writeToParcel(data, 0);
        data.writeString(resolvedType);
        data.writeString(callingPackage);
        data.writeInt(userId);
        mRemote.transact(START_SERVICE_TRANSACTION, data, reply, 0);
        reply.readException();
        ComponentName res = ComponentName.readFromParcel(reply);
        data.recycle();
        reply.recycle();
        return res;
    }
···
}
```

/Volumes/android/aosp/frameworks/base/core/java/android/app/ActivityManagerNative.java

```java
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
            throws RemoteException {
        switch (code) {
                case START_SERVICE_TRANSACTION: {
            data.enforceInterface(IActivityManager.descriptor);
            IBinder b = data.readStrongBinder();
            IApplicationThread app = ApplicationThreadNative.asInterface(b);
            Intent service = Intent.CREATOR.createFromParcel(data);
            String resolvedType = data.readString();
            String callingPackage = data.readString();
            int userId = data.readInt();
            // 调用 AMS 的 startService
            ComponentName cn = startService(app, service, resolvedType, callingPackage, userId);
            reply.writeNoException();
            ComponentName.writeToParcel(cn, reply);
            return true;
        }
        }
}
```


# AMS 启动 Service 过程

先看下 AMS 启动 Service 的时序图：

![AMS启动Service时序图](http://onke0yoit.bkt.clouddn.com/AMS启动Service时序图.png)

首先查看 AMS 的 startService 方法。

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
    @Override
    public ComponentName startService(IApplicationThread caller, Intent service,
            String resolvedType, String callingPackage, int userId)
            throws TransactionTooLargeException {
            
            ···
            
        synchronized(this) {
            final int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            ComponentName res = mServices.startServiceLocked(caller, service,
                    resolvedType, callingPid, callingUid, callingPackage, userId); //1
            Binder.restoreCallingIdentity(origId);
            return res;
        }
    }
```
1 处调用 mServices 的 startServiceLocked 方法，mServices 的类型是 ActiveServices，ActiveServices的 startServiceLocked 方法如下：

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

```java
ComponentName startServiceLocked(IApplicationThread caller, Intent service, String resolvedType,
            int callingPid, int callingUid, String callingPackage, final int userId)
            throws TransactionTooLargeException {
      ...
        return startServiceInnerLocked(smap, service, r, callerFg, addToStarting);
    }

```

startServiceLocked 返回调用 startServiceInnerLocked 方法。

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

```java
   ComponentName startServiceInnerLocked(ServiceMap smap, Intent service, ServiceRecord r,
            boolean callerFg, boolean addToStarting) throws TransactionTooLargeException {
 
     ...
        String error = bringUpServiceLocked(r, service.getFlags(), callerFg, false, false);
     ...
        return r.name;
    }
```

startServiceInnerLocked 方法中调用了 bringUpServiceLocked 方法。

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

```java
  private String bringUpServiceLocked(ServiceRecord r, int intentFlags, boolean execInFg,
            boolean whileRestarting, boolean permissionsReviewRequired)
            throws TransactionTooLargeException {
...
  final String procName = r.processName;//1
  ProcessRecord app;
  if (!isolated) {
            app = mAm.getProcessRecordLocked(procName, r.appInfo.uid, false);//2
            if (DEBUG_MU) Slog.v(TAG_MU, "bringUpServiceLocked: appInfo.uid=" + r.appInfo.uid
                        + " app=" + app);
            if (app != null && app.thread != null) {//3
                try {
                    app.addPackage(r.appInfo.packageName, r.appInfo.versionCode,
                    mAm.mProcessStats);
                    realStartServiceLocked(r, app, execInFg);//4
                    return null;
                } catch (TransactionTooLargeException e) {
                    throw e;
                } catch (RemoteException e) {
                    Slog.w(TAG, "Exception when starting service " + r.shortName, e);
                }
            }
        } else {
            app = r.isolatedProc;
        }
 if (app == null && !permissionsReviewRequired) {//5
            if ((app=mAm.startProcessLocked(procName, r.appInfo, true, intentFlags,
                    "service", r.name, false, isolated, false)) == null) {//6
              ...
            }
            if (isolated) {
                r.isolatedProc = app;
            }
        }
 ...     
}
```
1. 得到 ServiceRecord 的 processName 的值赋给 procName，procName 用来描述 Service 想要在哪个进程运行，默认是当前进程，我们也可以在 AndroidManifest 配置文件中设置 android:process 属性来新开启一个进程运行 Service。
2. 将 procName 和 Service 的 uid 传入到 AMS 的 getProcessRecordLocked 方法中，来查询是否存在一个与 Service 对应的 ProcessRecord 类型的对象 app，ProcessRecord 主要用来记录运行的应用程序进程的信息。
3. 判断用来运行 Service 的应用程序进程是否存在。
4. 调用 realStartServiceLocked 方法。
5. app 为 null 说明用来运行 Service 的应用程序进程不存在
6. 调用 AMS 的 startProcessLocked 方法来创建对应的应用程序进程。（详细过程查看 Android 应用程序进程启动过程）

frameworks/base/services/core/java/com/android/server/am/ActiveServices.java

```java
private final void realStartServiceLocked(ServiceRecord r,
        ProcessRecord app, boolean execInFg) throws RemoteException {
   ...
    try {
       ...
        app.thread.scheduleCreateService(r, r.serviceInfo,
                mAm.compatibilityInfoForPackageLocked(r.serviceInfo.applicationInfo),
                app.repProcState);
        r.postNotification();
        created = true;
    } catch (DeadObjectException e) {
      ...
    } 
    ...
}
```

> realStartServiceLocked 调用了 app.thread 的 app.thread.scheduleCreateService 方法。其中 app.thread 是 IApplicationThread 类型的，它的实现是 ActivityThread 的内部类 ApplicationThread，其中 ApplicationThread 继承了 ApplicationThreadNative，而 ApplicationThreadNative 继承了 Binder 并实现了 IApplicationThread 接口。

frameworks/base/core/java/android/app/ActivityThread.java

```java
public final void scheduleCreateService(IBinder token,
      ServiceInfo info, CompatibilityInfo compatInfo, int processState) {
  updateProcessState(processState, false);
  CreateServiceData s = new CreateServiceData();
  s.token = token;
  s.info = info;
  s.compatInfo = compatInfo;

  sendMessage(H.CREATE_SERVICE, s);
}
```

首先将要启动的信息封装成 CreateServiceData 对象并传给 sendMessage 方法，sendMessage 方法向 H 发送 H.CREATE_SERVICE，H 是 ActivityThread 的内部类继承 Handler。

H 中的 handleMessage 会接收并处理 H.CREATE_SERVICE 类型的消息，如下所示。

```java
public void handleMessage(Message msg) {
    
    switch (msg.what) {
        case CREATE_SERVICE:
          handleCreateService((CreateServiceData)msg.obj);
          break;
    }

}
```

frameworks/base/core/java/android/app/ActivityThread.java

```java
private void handleCreateService(CreateServiceData data) {
       unscheduleGcIdler();
       LoadedApk packageInfo = getPackageInfoNoCheck(
               data.info.applicationInfo, data.compatInfo);//1
       Service service = null;
       try {
           java.lang.ClassLoader cl = packageInfo.getClassLoader();//2
           service = (Service) cl.loadClass(data.info.name).newInstance();//3
       } catch (Exception e) {
          ...
           }
       }
       try {
           if (localLOGV) Slog.v(TAG, "Creating service " + data.info.name);
           ContextImpl context = ContextImpl.createAppContext(this, packageInfo);//4
           context.setOuterContext(service);
           Application app = packageInfo.makeApplication(false, mInstrumentation);
           service.attach(context, this, data.info.name, data.token, app,
                   ActivityManagerNative.getDefault());//5
           service.onCreate();//6
           mServices.put(data.token, service);//7
        ...
       } catch (Exception e) {
           ...
       }
   }
```
1. 获取要启动 Service 的应用程序 LoadedApk，LoadedApk 是一个文件的描述类
2. 通过调用 LoadedApk 的 getClassLoader 方法来获取类加载器。
3. 根据 CreateServiceData 对象中存储的 Service 信息，将 Service 加载到内存中。
4. 创建 Service 的上下文环境 ContextImpl。
5. 通过 Service 的 attach 方法来初始化 Service。
6. 调用 Service 的 onCreate 方法，Service 就启动了。
7. 将启动的 Service 加入到 ActivityThread 的成员变量 mServices 中，起重 mServices 是 ArrayMap 类型。

frameworks/base/core/java/android/app/Service.java

```java
    /**
     * Called by the system when the service is first created.  Do not call this method directly.
     */
    public void onCreate() {
    }
```

调用完 Service 的 onCreate 方法后，Service 就启动了。

# 参考资料

* [Android深入四大组件（二）Service的启动过程](http://liuwangshu.cn/framework/component/2-service-start.html)

