**Android四大组件①Activity启动过程**

# 概述

Activity 的启动过程分为两种，一种是根 Activity 的启动过程，另一种是普通 Activity 的启动过程，根 Activity 指的是应用程序启动的第一个 Activity，因此根 Activity 的启动过程也可以理解为应用程序的启动过程。普通的 Activity 指的是应用程序内除启动的第一个 Activity 之外的其他 Activity。

Activity 的启动过程比较复杂，分为三部分来学习，分别是 Launcher 请求 AMS 过程、AMS 到 ApplicationThread 的调用过程和 ApplicationThread 启动 Activity 过程。

# Launcher 请求 AMS 流程

Launcher 启动后会将已安装应用程序的快捷图标显示在桌面上，这些快捷图标就是启动应用主 Activity 的入口，当我们单击某个应用程序的快捷图标时就会通过 Lanucher 请求 AMS 来启动该应用程序。时序图如下所示：

![Launcher到AMS时序图](http://onke0yoit.bkt.clouddn.com/Launcher到AMS时序图.png)

点击快捷图标时，就会调用 Launcher 的 startActivitySafely 方法，如下所示。

packages/apps/Launcher3/src/com/android/launcher3/Launcher.java

```java
    public boolean startActivitySafely(View v, Intent intent, ItemInfo item) {
        
        ···

        // 根Activity在新任务中启动 1
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        if (v != null) {
            intent.setSourceBounds(getViewBounds(v));
        }
        try {
            if (Utilities.ATLEAST_MARSHMALLOW && item != null
                    && (item.itemType == Favorites.ITEM_TYPE_SHORTCUT
                    || item.itemType == Favorites.ITEM_TYPE_DEEP_SHORTCUT)
                    && ((ShortcutInfo) item).promisedIntent == null) {
                
                startShortcutIntentSafely(intent, optsBundle, item);
            } else if (user == null || user.equals(UserHandleCompat.myUserHandle())) {
                // 2
                startActivity(intent, optsBundle);
            } else {
                LauncherAppsCompat.getInstance(this).startActivityForProfile(
                        intent.getComponent(), user, intent.getSourceBounds(), optsBundle);
            }
            return true;
        } catch (ActivityNotFoundException|SecurityException e) {
            Toast.makeText(this, R.string.activity_not_found, Toast.LENGTH_SHORT).show();
            Log.e(TAG, "Unable to launch. tag=" + item + " intent=" + intent, e);
        }
        return false;
    }
```

该方法的主要作用：

1. 设置 Intent 的 Flag 为 Intent.FLAG_ACTIVITY_NEW_TASK，这样 Activity会在新的任务栈中启动。
2. 调用 startActivity 方法，这个方法实现在 Activity 类中，如下所示

frameworks/base/core/java/android/app/Activity.java

```java
public void startActivity(Intent intent, @Nullable Bundle options) {
   if (options != null) {
       startActivityForResult(intent, -1, options);
   } else {
       // Note we want to go through this call for compatibility with
       // applications that may have overridden the method.
       startActivityForResult(intent, -1);
   }
}
```

startActivity 会调用 startActivityForResult 方法，它的第二个参数为 -1，表示 Launcher 不需要知道 Activity 启动的结果，startActivityForResult 方法的实现如下：

frameworks/base/core/java/android/app/Activity.java

```java
    public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
            @Nullable Bundle options) {
        if (mParent == null) { //1
            options = transferSpringboardActivityOptions(options);
            Instrumentation.ActivityResult ar =
                mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
            if (ar != null) {
                mMainThread.sendActivityResult(
                    mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                    ar.getResultData());
            }
            if (requestCode >= 0) {
               
                mStartedActivity = true;
            }

            cancelInputsAndStartExitTransition(options);
            // TODO Consider clearing/flushing other event sources and events for child windows.
        } else {
            ···
        }
    }
```

mParent 是 Activity 类型的，表示当前 Activity 的父类。因为目前根 Activity 还没有创建出来，因此 `mParent == null` 成立，接着调用 
Instrumentation 的 execStartActivity 方法，Instrumentation 主要用来监控应用程序和系统的交互，execStartActivity 方法实现如下：

frameworks/base/core/java/android/app/Instrumentation.java

```java
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {
        ···
      
        try {
            intent.migrateExtraStreamToClipData();
            intent.prepareToLeaveProcess(who);
            //获取 AMS 的代理对象，通过 AMS 来启动
            int result = ActivityManagerNative.getDefault()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            checkStartActivityResult(result, intent);
        } catch (RemoteException e) {
            throw new RuntimeException("Failure from system", e);
        }
        return null;
    }
```

首先会调用 ActivityManagerNative 的 getDefault 方法来获取 ActivityMangerService 的代理对象，接着调用它的 startActivity 方法。

下面我们来查看 ActivityManagerNative 的 getDefault 方法做了什么：

frameworks/base/core/java/android/app/ActivityManagerNative.java

```java
static public IActivityManager getDefault() {
   return gDefault.get();
}
    
    
private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
protected IActivityManager create() {
  IBinder b = ServiceManager.getService("activity"); //1
  if (false) {
      Log.v("ActivityManager", "default service binder = " + b);
  }
  IActivityManager am = asInterface(b); //2
  if (false) {
      Log.v("ActivityManager", "default service = " + am);
  }
  return am;
}
};

static public IActivityManager asInterface(IBinder obj) {
   if (obj == null) {
       return null;
   }
   IActivityManager in =
       (IActivityManager)obj.queryLocalInterface(descriptor);
   if (in != null) {
       return in;
   }

   return new ActivityManagerProxy(obj);
}
    
```

getDefault 调用了 gDefault 的 get 方法，gDefault 是一个 Singleton 类。它首先通过 `ServiceManager.getService("activity")` 得到名为 “activity” 的 Service 引用，也就是 IBinder 类型的 AMS 的引用。接着将它封装成 ActivityManagerProxy 类型对象，并将它保存在 gDefault 中，以后调用 ActivityMangerNative 的 getDefault 方法就会直接获得 AMS 的代理 ActivityManagerProxy 对象。

回到 Instrumentation 类的 execStartActivity 方法中，从上面得知就是调用 ActivityManagerProxy 的 startActivity，其中 ActivityManagerProxy 是 ActivityManagerNative 的内部类，如下所示

frameworks/base/core/java/android/app/ActivityManagerNative.java

```java
    public int startActivity(IApplicationThread caller, String callingPackage, Intent intent,
            String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle options) throws RemoteException {
        Parcel data = Parcel.obtain();
        Parcel reply = Parcel.obtain();
        data.writeInterfaceToken(IActivityManager.descriptor);
        data.writeStrongBinder(caller != null ? caller.asBinder() : null);
        data.writeString(callingPackage);
        intent.writeToParcel(data, 0);
        data.writeString(resolvedType);
        data.writeStrongBinder(resultTo);
        data.writeString(resultWho);
        data.writeInt(requestCode);
        data.writeInt(startFlags);
        if (profilerInfo != null) {
            data.writeInt(1);
            profilerInfo.writeToParcel(data, Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
        } else {
            data.writeInt(0);
        }
        if (options != null) {
            data.writeInt(1);
            options.writeToParcel(data, 0);
        } else {
            data.writeInt(0);
        }
        mRemote.transact(START_ACTIVITY_TRANSACTION, data, reply, 0); //1
        reply.readException();
        int result = reply.readInt();
        reply.recycle();
        data.recycle();
        return result;
    }
```

首先会将传入的参数写入到 Parcel 类型的 data 中。在 1 处通过 IBinder 对象 mRemote 向 AMS 发送一个 START_ACTIVITY_TRANSACTION 类型的进程间通信请求。此时服务端 AMS 就会从 Binder 线程池中读取我们客户端发来的数据，最终会调用 ActivityMangerNative 的 onTransact 方法中执行，如下所示：

frameworks/base/core/java/android/app/ActivityManagerNative.java

```java
    @Override
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
            throws RemoteException {
        switch (code) {
        case START_ACTIVITY_TRANSACTION:
        {
            data.enforceInterface(IActivityManager.descriptor);
            IBinder b = data.readStrongBinder();
            IApplicationThread app = ApplicationThreadNative.asInterface(b);
            String callingPackage = data.readString();
            Intent intent = Intent.CREATOR.createFromParcel(data);
            String resolvedType = data.readString();
            IBinder resultTo = data.readStrongBinder();
            String resultWho = data.readString();
            int requestCode = data.readInt();
            int startFlags = data.readInt();
            ProfilerInfo profilerInfo = data.readInt() != 0
                    ? ProfilerInfo.CREATOR.createFromParcel(data) : null;
            Bundle options = data.readInt() != 0
                    ? Bundle.CREATOR.createFromParcel(data) : null;
            int result = startActivity(app, callingPackage, intent, resolvedType,
                    resultTo, resultWho, requestCode, startFlags, profilerInfo, options);
            reply.writeNoException();
            reply.writeInt(result);
            return true;
        }
    ···
}
```
onTransact 方法中会调用 AMS 的 startActivity 方法，如下所示：

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }
```

>以上就是 Launcher 请求 ActivityManagerService 简要过程。

# AMS 调用 ApplicationThread 流程

Launcher 请求 AMS 后，接着是 AMS 到 ApplicationThread 的调用过程，时序图如下：

![ActivityManageService到ApplicationThread调用过程的时序图](http://onke0yoit.bkt.clouddn.com/ActivityManageService到ApplicationThread调用过程的时序图.png)

AMS 中的 startActivity 方法如下：

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
    @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
                resultWho, requestCode, startFlags, profilerInfo, bOptions,
                UserHandle.getCallingUserId());
    }

```

AMS 中的 startActivity 方法中 return 了 startActivityAsUser 方法，可以看到 startActivityAsUser 方法比 startActivity 方法多了一个参数 `UserHandle.getCallingUserId()`，这个方法会获得调用者的 UserId，AMS 会根据这个 UserId 来确定调用者的权限。

startActivityAsUser 方法实现如下：

```java
    @Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle bOptions, int userId) {
        //  判断调用者进程是否被隔离
        enforceNotIsolatedCaller("startActivity"); //1
        // 检查调用者权限
        userId = mUserController.handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(),
                userId, false, ALLOW_FULL_ONLY, "startActivity", null); //2
        // TODO: Switch to user app stacks here.
        return mActivityStarter.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, bOptions, false, userId, null, null);
    }
```
1. 判断调用者进程是否被隔离，如果被隔离则抛出 SecurityException 异常。
2. 用于检查调用者是否有权限，如果没有权限也会抛出 SecurityException 异常。
3. 最后调用了 ActivityStarter 的 startActivityMayWait 方法。startActivityLocked 方法的最后一个参数类型为 TaskRecord，代表启动的 Activity 所在的栈。

frameworks/base/services/core/java/com/android/server/am/ActivityStarter.java

```java
final int startActivityMayWait(IApplicationThread caller, int callingUid,
         String callingPackage, Intent intent, String resolvedType,
         IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
         IBinder resultTo, String resultWho, int requestCode, int startFlags,
         ProfilerInfo profilerInfo, IActivityManager.WaitResult outResult, Configuration config,
         Bundle bOptions, boolean ignoreTargetSecurity, int userId,
         IActivityContainer iContainer, TaskRecord inTask) {
   ...
         int res = startActivityLocked(caller, intent, ephemeralIntent, resolvedType,
                 aInfo, rInfo, voiceSession, voiceInteractor,
                 resultTo, resultWho, requestCode, callingPid,
                 callingUid, callingPackage, realCallingPid, realCallingUid, startFlags,
                 options, ignoreTargetSecurity, componentSpecified, outRecord, container,
                 inTask);
     ...
         return res;
     }
 }
```

>ActivityStarter 是 Android 7.0 新加入的类，它是加载 Activity 的控制类，会收集所有的逻辑来决定如何将 Intent 和 Flags 转换为 Activity，并将 Activity 和 Task 以及 Stack 想关联。ActivityStarter 的 startActivityMayWait 方法调用了 startActivityLocked 方法，如下所示

frameworks/base/services/core/java/com/android/server/am/ActivityStarter.java

```java
final int startActivityLocked(IApplicationThread caller, Intent intent, Intent ephemeralIntent,
           String resolvedType, ActivityInfo aInfo, ResolveInfo rInfo,
           IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
           IBinder resultTo, String resultWho, int requestCode, int callingPid, int callingUid,
           String callingPackage, int realCallingPid, int realCallingUid, int startFlags,
           ActivityOptions options, boolean ignoreTargetSecurity, boolean componentSpecified,
           ActivityRecord[] outActivity, ActivityStackSupervisor.ActivityContainer container,
           TaskRecord inTask) {
      ...
       doPendingActivityLaunchesLocked(false);
      ...
       return err;
   }
```

startActivityLocked 方法代码非常多，这里只需要关注 doPendingActivityLaunchesLocked 方法，如下所示

frameworks/base/services/core/java/com/android/server/am/ActivityStarter.java

```java
    final void doPendingActivityLaunchesLocked(boolean doResume) {
        while (!mPendingActivityLaunches.isEmpty()) {
            final PendingActivityLaunch pal = mPendingActivityLaunches.remove(0);
            final boolean resume = doResume && mPendingActivityLaunches.isEmpty();
            try {
                final int result = startActivityUnchecked(
                        pal.r, pal.sourceRecord, null, null, pal.startFlags, resume, null, null);
                postStartActivityUncheckedProcessing(
                        pal.r, result, mSupervisor.mFocusedStack.mStackId, mSourceRecord,
                        mTargetStack);
            } catch (Exception e) {
                Slog.e(TAG, "Exception during pending activity launch pal=" + pal, e);
                pal.sendErrorResult(e.getMessage());
            }
        }
    }
```
doPendingActivityLaunchesLocked 又调用了 startActivityUnchecked 方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStarter.java

```java
private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
           IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
           int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask) {
     ...  
        mSupervisor.resumeFocusedStackTopActivityLocked();  
     ... 
       return START_SUCCESS;
   }
```

startActivityUnchecked 方法中调用了 ActivityStackSupervisor 类型的 mSupervisor 的 resumeFocusedStackTopActivityLocked 方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

```java
boolean resumeFocusedStackTopActivityLocked(
           ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {
       if (targetStack != null && isFocusedStack(targetStack)) {
           return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions); //1
       }
       final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
       if (r == null || r.state != RESUMED) {
           mFocusedStack.resumeTopActivityUncheckedLocked(null, null);//1
       }
       return false;
   }
```
resumeFocusedStackTopActivityLocked 又调用了 ActivityStack 类型的 mFocusedStack 的 resumeTopActivityUncheckedLocked 方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStack.java

```java
 boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
...
        try {
           ...
            result = resumeTopActivityInnerLocked(prev, options);
        } finally {
            mStackSupervisor.inResumeTopActivity = false;
        }
        return result;
    }
```
紧接着调用 resumeTopActivityInnerLocked 方法

frameworks/base/services/core/java/com/android/server/am/ActivityStack.java

```java
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {
      ...
           mStackSupervisor.startSpecificActivityLocked(next, true, true);
       }
        if (DEBUG_STACK) mStackSupervisor.validateTopActivitiesLocked();
       return true;
```

我们只关注 resumeTopActivityInnerLocked 方法中 调用了 ActivityStackSupervisor 类型的 mStackSupervisor 的 startSpecificActivityLocked 方法。

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

```java
void startSpecificActivityLocked(ActivityRecord r,
          boolean andResume, boolean checkConfig) {
      ProcessRecord app = mService.getProcessRecordLocked(r.processName,
              r.info.applicationInfo.uid, true); //1
      r.task.stack.setLaunchTime(r);
      if (app != null && app.thread != null) {//2
          try {
              if ((r.info.flags&ActivityInfo.FLAG_MULTIPROCESS) == 0
                      || !"android".equals(r.info.packageName)) {
                  app.addPackage(r.info.packageName, r.info.applicationInfo.versionCode,
                          mService.mProcessStats);
              }
              realStartActivityLocked(r, app, andResume, checkConfig);//3
              return;
          } catch (RemoteException e) {
              Slog.w(TAG, "Exception when starting activity "
                      + r.intent.getComponent().flattenToShortString(), e);
          }
      }
      mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
              "activity", r.intent.getComponent(), false, false, true);
  }
```
1. 获取即将要启动的 Activity 所在的应用程序进程
2. 判断要启动的 Activity 所在的应用程序进程是否已经运行，是就调用 3 处的 realStartActivityLocked 方法，这个方法的第二个参数 是代表要启动的 Activity 所在的应用程序进程的 ProcessRecord。

realStartActivityLocked 方法如下所示

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

```java
final boolean realStartActivityLocked(ActivityRecord r, ProcessRecord app,
          boolean andResume, boolean checkConfig) throws RemoteException {
   ...
          app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
                  System.identityHashCode(r), r.info, new Configuration(mService.mConfiguration),
                  new Configuration(task.mOverrideConfig), r.compat, r.launchedFromPackage,
                  task.voiceInteractor, app.repProcState, r.icicle, r.persistentState, results,
                  newIntents, !andResume, mService.isNextTransitionForward(), profilerInfo);
  ...      
      return true;
  }
```

这里的 app.thread 指的是 IApplicationThread 接口，它的实现是 ActivityThread 的内部类 ApplicationThread，其中 ApplicationThread 继承了 ApplicationThreadNative，而 ApplicationThreadNative 继承了 Binder 并实现了 IApplicationThread 接口。app 指的是传入的要启动的 Activity 的所在的应用程序进程，realStartActivityLocked 方法就是要在目标应用程序进程中启动 Activity，而当前代码逻辑是运行在 AMS 所在的进程（SystemServer 进程），通过 ApplicationThread 来与应用程序进程进行 Binder 通信， ApplicationThread 可以说是 AMS 所在进程（SystemServer 进程）和应用程序进程的通信桥梁。

![](http://upload-images.jianshu.io/upload_images/1417629-04cd897216bdbfa8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>以上就是 AMS 到 ApplicationThread 调用过程的简要分析。

# ActivityThread 启动 Activity 流程

AMS 调用 ApplicationThread 后，进入最后流程 ActivityThread 启动 Activity，时序图如下所示：

![ActivityThread启动Activity的时序图](http://onke0yoit.bkt.clouddn.com/ActivityThread启动Activity的时序图.png)

首先来看 ApplicationThread 的 scheduleLaunchActivity 方法：

frameworks/base/core/java/android/app/ActivityThread.java

```java
   public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
           ActivityInfo info, Configuration curConfig, Configuration overrideConfig,
           CompatibilityInfo compatInfo, String referrer, IVoiceInteractor voiceInteractor,
           int procState, Bundle state, PersistableBundle persistentState,
           List<ResultInfo> pendingResults, List<ReferrerIntent> pendingNewIntents,
           boolean notResumed, boolean isForward, ProfilerInfo profilerInfo) {

       updateProcessState(procState, false);

       ActivityClientRecord r = new ActivityClientRecord();

       r.token = token;
       r.ident = ident;
       r.intent = intent;
       r.referrer = referrer;
       r.voiceInteractor = voiceInteractor;
       r.activityInfo = info;
       r.compatInfo = compatInfo;
       r.state = state;
       r.persistentState = persistentState;

       r.pendingResults = pendingResults;
       r.pendingIntents = pendingNewIntents;

       r.startsNotResumed = notResumed;
       r.isForward = isForward;

       r.profilerInfo = profilerInfo;

       r.overrideConfig = overrideConfig;
       updatePendingConfiguration(curConfig);

       sendMessage(H.LAUNCH_ACTIVITY, r);
   }
```

scheduleLaunchActivity 方法会将启动 Activity 参数封装成 ActivityClientRecord，然后通过 sendMessage 方法向 H 类发送类型为 H.LAUNCH_ACTIVITY 的消息，并将 ActivityClientRecord 对象传递出去，sendMessage 方法如下：

frameworks/base/core/java/android/app/ActivityThread.java

```java
private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
   if (DEBUG_MESSAGES) Slog.v(
       TAG, "SCHEDULE " + what + " " + mH.codeToString(what)
       + ": " + arg1 + " / " + obj);
   Message msg = Message.obtain();
   msg.what = what;
   msg.obj = obj;
   msg.arg1 = arg1;
   msg.arg2 = arg2;
   if (async) {
       msg.setAsynchronous(true);
   }
   mH.sendMessage(msg);
}
```
sendMessage 方法实际上是通过 mH 发送消息，mH 是 H 类对象，它是 ActivityThread 的内部类并继承 Handler，H 的代码如下所示

frameworks/base/core/java/android/app/ActivityThread.java

```java
private class H extends Handler {

        public static final int LAUNCH_ACTIVITY         = 100;
        public static final int PAUSE_ACTIVITY          = 101;
        
      public void handleMessage(Message msg) {
  if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
  switch (msg.what) {
      case LAUNCH_ACTIVITY: {
          Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
          final ActivityClientRecord r = (ActivityClientRecord) msg.obj; //1

          r.packageInfo = getPackageInfoNoCheck(
                  r.activityInfo.applicationInfo, r.compatInfo); //2 
          handleLaunchActivity(r, null, "LAUNCH_ACTIVITY"); //3
          Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
      } break;
      case RELAUNCH_ACTIVITY: {
          Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityRestart");
          ActivityClientRecord r = (ActivityClientRecord)msg.obj;
          handleRelaunchActivity(r);
          Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
      } break;
    }
}
```

查看 H 的 handleMessage 方法中对 LAUNCH_ACTIVITY 类型消息的处理，

1. 首先将传过来的 msg 的成员变量 obj 转换为 ActivityClientRecord。
2. 通过 getPackageInfoNoCheck 方法获取 LoadedApk 类型的对象并赋值给 ActivityClientRecord 的成员变量 packageInfo。应用程序进程要启动 Activity 时需要将该 Activity 所属的 APK 加载进来，而 LoadedApk 类就是用来描述已加载的 APK 文件。
3. 调用 handleLaunchActivity 方法，如下所示

```java
    private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent, String reason) {
        
···
        Activity a = performLaunchActivity(r, customIntent); //1

        if (a != null) {
            r.createdConfig = new Configuration(mConfiguration);
            reportSizeConfigurations(r);
            Bundle oldState = r.state;
            handleResumeActivity(r.token, false, r.isForward,
                    !r.activity.mFinished && !r.startsNotResumed, r.lastProcessedSeq, reason); //2

            if (!r.activity.mFinished && r.startsNotResumed) {
                performPauseActivityIfNeeded(r, reason);
                if (r.isPreHoneycomb()) {
                    r.state = oldState;
                }
            }
        } else {
            // 
            try {
                ActivityManagerNative.getDefault()
                    .finishActivity(r.token, Activity.RESULT_CANCELED, null,
                            Activity.DONT_FINISH_TASK_WITH_ACTIVITY);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }
    }
```

1. performLaunchActivity 方法用来启动 Activity，
2. 用来将 Activity 的状态置为 Resume。
3. 如果该 Activity 为 null 则会通知 ActivityManger 停止启动 Activity。

frameworks/base/core/java/android/app/ActivityThread.java

```java
  private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
  ...
        ActivityInfo aInfo = r.activityInfo;//1
        if (r.packageInfo == null) {
            r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
                    Context.CONTEXT_INCLUDE_CODE);//2
        }
        ComponentName component = r.intent.getComponent();//3
      ...
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);//4
           ...
            }
        } catch (Exception e) {
         ...
        }
        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);//5
        ...
            if (activity != null) {
                Context appContext = createBaseContextForActivity(r, activity);//6
         ...
                }
                /**
                *7
                */
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window); //7
              ...
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);//8
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                ...
        }
        return activity;
}
```

1. 用来获取 ActivityInfo
2. 获取 APK 文件的描述类 LoadedApk
3. 获取要启动的 Activity 的 ComponentName 类，ComponentName 类中保存了该 Activity 的包名和类名。
4. 根据 ComponentName 中存储的 Activity 类名，用类加载器来创建该 Activity 实例。
5. 创建 Application，makeApplication 方法内部会调用 Application 的 onCreate 方法。
6. 用来创建要启动 Activity 的上下文环境。
7. 调用 Activity 的 attach 方法初始化 Activity，attach 方法中会创建 Window 对象 （PhoneWindow） 并与 Activity 自身相关联。
8. 调用 Instrumentation 的 callActivityOnCreate 方法来启动 Activity。

frameworks/base/core/java/android/app/Instrumentation.java

```java
    public void callActivityOnCreate(Activity activity, Bundle icicle) {
        prePerformCreate(activity);
        activity.performCreate(icicle); //1
        postPerformCreate(activity);
    }
```
callActivityOnCreate 会调用 Activity 的 performCreate 方法。

frameworks/base/core/java/android/app/Activity.java

```java
    final void performCreate(Bundle icicle) {
        restoreHasCurrentPermissionRequest(icicle);
        onCreate(icicle);
        mActivityTransitionState.readState(icicle);
        performCreateCommon();
    }
```
performCreate 方法中会调用 Activity 的 onCreate 方法，这样 Activity 就启动了。

frameworks/base/core/java/android/app/Activity.java

```java
    @MainThread
    @CallSuper
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        if (DEBUG_LIFECYCLE) Slog.v(TAG, "onCreate " + this + ": " + savedInstanceState);
        if (mLastNonConfigurationInstances != null) {
            mFragments.restoreLoaderNonConfig(mLastNonConfigurationInstances.loaders);
        }
        if (mActivityInfo.parentActivityName != null) {
            if (mActionBar == null) {
                mEnableDefaultActionBarUp = true;
            } else {
                mActionBar.setDefaultDisplayHomeAsUpEnabled(true);
            }
        }
        if (savedInstanceState != null) {
            Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);
            mFragments.restoreAllState(p, mLastNonConfigurationInstances != null
                    ? mLastNonConfigurationInstances.fragments : null);
        }
        mFragments.dispatchCreate();
        getApplication().dispatchActivityCreated(this, savedInstanceState);
        if (mVoiceInteractor != null) {
            mVoiceInteractor.attachActivity(this);
        }
        mCalled = true;
    }
```

>以上就是 ActivityThread 启动 Activity 的简要过程。

# 参考资料

* [Android深入四大组件（一）应用程序启动过程（前篇）](http://liuwangshu.cn/framework/component/1-activity-start-1.html)
* [Android深入四大组件（一）应用程序启动过程（后篇）](http://liuwangshu.cn/framework/component/1-activity-start-2.html)
* [Android深入四大组件 Android8.0 根 Activity启动过程（前篇）
](http://liuwangshu.cn/framework/component/6-activity-start-1.html)


