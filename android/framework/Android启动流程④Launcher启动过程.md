**Android启动流程④Launcher启动过程**

[TOC]

Android系统的Home应用程序Launcher是由ActivityManagerService启动的，而ActivityManagerService和PackageManagerService一样，都是在开机时由SystemServer组件启动的，SystemServer组件首先是启动PackageManagerServic，由它来负责安装系统的应用程序，系统中的应用程序安装好了以后，SystemServer组件接下来就要通过ActivityManagerService来启动Home应用程序Launcher了，Launcher在启动的时候便会通过PackageManagerServic把系统中已经安装好的应用程序以快捷图标的形式展示在桌面上，这样用户就启动这些应用程序。

# Launcher启动流程

启动Launcher时通过调用ActivityManagerService的systemReady函数实现的

frameworks/base/services/java/com/android/server/SystemServer.java

```java
  
 private void startOtherServices() {
 ...
  mActivityManagerService.systemReady(new Runnable() {
            @Override
            public void run() {
                Slog.i(TAG, "Making services ready");
                mSystemServiceManager.startBootPhase(
                        SystemService.PHASE_ACTIVITY_MANAGER_READY);
    ...
    }
...
}
```
SystemServer.java中的startOtherServices方法中调用ActivityManagerService的systemReady函数。

```java
public void systemReady(final Runnable goingCallback) {

    ···
    ···
    mStackSupervisor.resumeFocusedStackTopActivityLocked();
    mUserController.sendUserSwitchBroadcastsLocked(-1, currentUserId);

}

```
systemReady方法会调用ActivityStackSupervisor.java的resumeFocusedStackTopActivityLocked方法

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

```java
    boolean resumeFocusedStackTopActivityLocked(
            ActivityStack targetStack, ActivityRecord target, ActivityOptions targetOptions) {
        if (targetStack != null && isFocusedStack(targetStack)) {
            return targetStack.resumeTopActivityUncheckedLocked(target, targetOptions);
        }
        final ActivityRecord r = mFocusedStack.topRunningActivityLocked();
        if (r == null || r.state != RESUMED) {
            mFocusedStack.resumeTopActivityUncheckedLocked(null, null);
        }
        return false;
    }

```
resumeFocusedStackTopActivityLocked会调用ActivityStack.java的resumeTopActivityUncheckedLocked方法，ActivityStack对象用于描述Activity堆栈。

frameworks/base/services/core/java/com/android/server/am/ActivityStack.java

```java
    //* Ensure that the top activity in the stack is resumed.
    boolean resumeTopActivityUncheckedLocked(ActivityRecord prev, ActivityOptions options) {
        if (mStackSupervisor.inResumeTopActivity) {
            // Don't even start recursing.
            return false;
        }

        boolean result = false;
        try {
            // Protect against recursion.
            mStackSupervisor.inResumeTopActivity = true;
            if (mService.mLockScreenShown == ActivityManagerService.LOCK_SCREEN_LEAVING) {
                mService.mLockScreenShown = ActivityManagerService.LOCK_SCREEN_HIDDEN;
                mService.updateSleepIfNeededLocked();
            }
            //
            result = resumeTopActivityInnerLocked(prev, options);
        } finally {
            mStackSupervisor.inResumeTopActivity = false;
        }
        return result;
    }

```

resumeTopActivityUncheckedLocked会调用resumeTopActivityInnerLocked方法

frameworks/base/services/core/java/com/android/server/am/ActivityStack.java

```java
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, ActivityOptions options) {

···
   
   if (next == null) {
       // There are no more activities!
       ···
       // Only resume home if on home display
       return isOnHomeDisplay() &&
               mStackSupervisor.resumeHomeStackTask(returnTaskType, prev, reason);
   }
···

}

```

resumeTopActivityInnerLocked方法调用ActivityStackSupervisor.java的resumeHomeStackTask方法

frameworks/base/services/core/java/com/android/server/am/ActivityStackSupervisor.java

```java
    boolean resumeHomeStackTask(int homeStackTaskType, ActivityRecord prev, String reason) {
        
        ···
        // Only resume home activity if isn't finishing.
        if (r != null && !r.finishing) {
            mService.setFocusedActivityLocked(r, myReason);
            return resumeFocusedStackTopActivityLocked(mHomeStack, prev, null);
        }
        return mService.startHomeActivityLocked(mCurrentUser, myReason);
    }

```
resumeHomeStackTask最后调用ActivityManagerService.java的startHomeActivityLocked函数

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
String mTopAction = Intent.ACTION_MAIN;
    boolean startHomeActivityLocked(int userId, String reason) {
        //
        if (mFactoryTest == FactoryTest.FACTORY_TEST_LOW_LEVEL
                && mTopAction == null) {
            // We are running in factory test mode, but unable to find
            // the factory test app, so just sit around displaying the
            // error message and don't try to start anything.
            return false;
        }
        //创建要启动的Intent
        Intent intent = getHomeIntent();
        ActivityInfo aInfo = resolveActivityInfo(intent, STOCK_PM_FLAGS, userId);
        if (aInfo != null) {
            intent.setComponent(new ComponentName(aInfo.applicationInfo.packageName, aInfo.name));
            // Don't do this if the home app is currently being
            // instrumented.
            aInfo = new ActivityInfo(aInfo);
            aInfo.applicationInfo = getAppInfoForUser(aInfo.applicationInfo, userId);
            ProcessRecord app = getProcessRecordLocked(aInfo.processName,
                    aInfo.applicationInfo.uid, true);
            //判断符合Action为Intent.ACTION_MAIN，Category为Intent.CATEGORY_HOME的应用程序是否已经启动
            if (app == null || app.instrumentationClass == null) {
                intent.setFlags(intent.getFlags() | Intent.FLAG_ACTIVITY_NEW_TASK);
                //启动Intent
                mActivityStarter.startHomeActivityLocked(intent, aInfo, reason);
            }
        } else {
            Slog.wtf(TAG, "No home screen found for " + intent, new Throwable());
        }

        return true;
    }

```

```java
//创建Intent
Intent getHomeIntent() {
   Intent intent = new Intent(mTopAction, mTopData != null ? Uri.parse(mTopData) : null);
   intent.setComponent(mTopComponent);
   intent.addFlags(Intent.FLAG_DEBUG_TRIAGED_MISSING);
   if (mFactoryTest != FactoryTest.FACTORY_TEST_LOW_LEVEL) {
        //表示Lanucher
       intent.addCategory(Intent.CATEGORY_HOME);
   }
   return intent;
}
```
ActivityManagerService.java的startHomeActivityLocked方法最终完成Lanucher的启动

1. 如何系统运行模式是低级工厂模式并且mTpoicAction为null，则退出方法，否则进行下面的处理
2. 创建Intent，如果系统模式不是低级工厂模式，则将intent的Category设置为Intent.CATEGORY_HOME
3. 判断符合Action为Intent.ACTION_MAIN，Category为Intent.CATEGORY_HOME的应用程序是否已经启动
4. 如果没有启动则启动该应用程序，就是Launcher程序。

Launcher3的AndroidManifest.xml配置文件中的intent-filter匹配Action为android.intent.action.MAIN，Category为android.intent.category.HOME。如下所示

packages/apps/Launcher3/AndroidManifest.xml

```xml

<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.android.launcher3">
    <uses-sdk android:targetSdkVersion="23" android:minSdkVersion="16"/>
 ...
 <application
        ...
        <activity
            android:name="com.android.launcher3.Launcher"
            android:launchMode="singleTask"
            android:clearTaskOnLaunch="true"
            android:stateNotNeeded="true"
            android:theme="@style/Theme"
            android:windowSoftInputMode="adjustPan"
            android:screenOrientation="nosensor"
            android:configChanges="keyboard|keyboardHidden|navigation"
            android:resumeWhilePausing="true"
            android:taskAffinity=""
            android:enabled="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.HOME" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.MONKEY"/>
            </intent-filter>
        </activity>
...
  </application> 
</manifest>
```

# Lanucher应用图标显示过程

Lanucher3继承Activity，所以它的运行从onCreate方法开始

packages/apps/Launcher3/src/com/android/launcher3/Launcher.java

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {

···
        super.onCreate(savedInstanceState);
        //1
        LauncherAppState app = LauncherAppState.getInstance();
···     //2
        mModel = app.setLauncher(this);
        
        //3
        if (!mModel.startLoader(mWorkspace.getRestorePage())) {
            mDragLayer.setAlpha(0);
        } else {
            setWorkspaceLoading(true);
        }
        
        ···
    }

```

1. 获取LauncherAppState的实例app
2. 调用app的setLauncher函数并将Lanucher对象传入
3. LanucherModel调用startLoader开始加载数据

LauncherAppState的setLauncher函数如下

packages/apps/Launcher3/src/com/android/launcher3/LauncherAppState.java

```java
LauncherModel setLauncher(Launcher launcher) {
     getLauncherProvider().setLauncherProviderChangeListener(launcher);
     mModel.initialize(launcher);//1
     mAccessibilityDelegate = ((launcher != null) && Utilities.ATLEAST_LOLLIPOP) ?
         new LauncherAccessibilityDelegate(launcher) : null;
     return mModel;
 }
 
 public void initialize(Callbacks callbacks) {
    synchronized (mLock) {
        unbindItemInfosAndClearQueuedBindRunnables();
        mCallbacks = new WeakReference<Callbacks>(callbacks);
    }
}
```
initialize函数中将callbacks封装成一个弱引用对象，这个mCallbacks对象就是传入的launcher对象，下面将会用到它。

接下来回到LauncherModel的startLoader函数

packages/apps/Launcher3/src/com/android/launcher3/LauncherModel.java

```java
    // 创建了具有消息循环的线程HandlerThread对象
    @Thunk static final HandlerThread sWorkerThread = new HandlerThread("launcher-loader");
    static {
        sWorkerThread.start();
    }
    // 创建了Handler，并且传入HandlerThread的Looper。Hander的作用就是向HandlerThread发送消息
    @Thunk static final Handler sWorker = new Handler(sWorkerThread.getLooper());

    public boolean startLoader(int synchronousBindPage) {
        // Enable queue before starting loader. It will get disabled in Launcher#finishBindingItems
        InstallShortcutReceiver.enableInstallQueue();
        synchronized (mLock) {
            // Don't bother to start the thread if we know it's not going to do anything
            if (mCallbacks != null && mCallbacks.get() != null) {
                final Callbacks oldCallbacks = mCallbacks.get();
                // Clear any pending bind-runnables from the synchronized load process.
                runOnMainThread(new Runnable() {
                    public void run() {
                        oldCallbacks.clearPendingBinds();
                    }
                });

                // If there is already one running, tell it to stop.
                stopLoaderLocked();
                // 创建LoaderTask
                mLoaderTask = new LoaderTask(mApp.getContext(), synchronousBindPage);
                // TODO: mDeepShortcutsLoaded does not need to be true for synchronous bind.
                if (synchronousBindPage != PagedView.INVALID_RESTORE_PAGE && mAllAppsLoaded
                        && mWorkspaceLoaded && mDeepShortcutsLoaded && !mIsLoaderTaskRunning) {
                    mLoaderTask.runBindSynchronousPage(synchronousBindPage);
                    return true;
                } else {
                    sWorkerThread.setPriority(Thread.NORM_PRIORITY);
                    // 将LoaderTask作为消息发送给HandlerThread 
                    sWorker.post(mLoaderTask);
                }
            }
        }
        return false;
    }

```
* 创建LoadTask对象
* 将LoadTask对象作为消息发送到HandlerThread处理，sWorkerThread是具有消息循环的线程HandlerThread对象，sWorker是一个Handler对象，并且传入了sWorkerThread的Looper，所以sWorker发送的消息将由sWorkerThread处理。

LoaderTask实现了Runnable接口，当它被处理时会调用run方法。

packages/apps/Launcher3/src/com/android/launcher3/LauncherModel.java

```java

private class LoaderTask implements Runnable {

···
        public void run() {
            synchronized (mLock) {
                if (mStopped) {
                    return;
                }
                mIsLoaderTaskRunning = true;
            }
            
            keep_running: {
                //step 1: 加载工作区信息
                loadAndBindWorkspace();

                if (mStopped) {
                    break keep_running;
                }

                waitForIdle();
                
                //step 2: 加载系统已经安装的应用程序信息
                loadAndBindAllApps();

                waitForIdle();

                //step 3: loading deep shortcuts
                loadAndBindDeepShortcuts();
            }

            ···
        }
    ···
}

```
* loadAndBindWorkspace方法加载工作区信息
* loadAndBindAllApps方法加载系统以及安装的应用程序

loadAndBindAllApps方法的实现如下：

```java

private void loadAndBindAllApps() {
  if (DEBUG_LOADERS) {
      Log.d(TAG, "loadAndBindAllApps mAllAppsLoaded=" + mAllAppsLoaded);
  }
  
  if (!mAllAppsLoaded) {
    //如果系统没有加载已经安装的应用程序信息，则加载所有应用
      loadAllApps();
      synchronized (LoaderTask.this) {
          if (mStopped) {
              return;
          }
      }
      updateIconCache();
      synchronized (LoaderTask.this) {
          if (mStopped) {
              return;
          }
          mAllAppsLoaded = true;
      }
  } else {
      onlyBindAllApps();
  }
}
```
loadAllApps方法如下

```java
private void loadAllApps() {

···
//就是获取Launcher对象
  final Callbacks oldCallbacks = mCallbacks.get();
     
  final ArrayList<AppInfo> added = mBgAllAppsList.added;
  mBgAllAppsList.added = new ArrayList<AppInfo>();

  // Post callback on main thread
  mHandler.post(new Runnable() {
      public void run() {

          final long bindTime = SystemClock.uptimeMillis();
          final Callbacks callbacks = tryGetCallbacks(oldCallbacks);
          if (callbacks != null) {
                //
              callbacks.bindAllApplications(added);
              if (DEBUG_LOADERS) {
                  Log.d(TAG, "bound " + added.size() + " apps in "
                          + (SystemClock.uptimeMillis() - bindTime) + "ms");
              }
          } else {
              Log.i(TAG, "not binding apps: no Launcher activity");
          }
      }
  });
  ···
}

```

callbacks对象事件上就是Launcher对象，callbacks.bindAllApplications就是调用Launcher的bindAllApplications方法。

packages/apps/Launcher3/src/com/android/launcher3/Launcher.java

```java

    /**
     * Add the icons for all apps.
     *
     * Implementation of the method from LauncherModel.Callbacks.
     */
    public void bindAllApplications(final ArrayList<AppInfo> apps) {
        if (waitUntilResume(mBindAllApplicationsRunnable, true)) {
            mTmpAppsList = apps;
            return;
        }

        if (mAppsView != null) {
            //传入App列表数据
            mAppsView.setApps(apps);
        }
        if (mLauncherCallbacks != null) {
            mLauncherCallbacks.bindAllApplications(apps);
        }
    }
```

packages/apps/Launcher3/src/com/android/launcher3/allapps/AllAppsContainerView.java

```java
    public void setApps(List<AppInfo> apps) {
        mApps.setApps(apps);
    }
```

```java
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        // Load the all apps recycler view
        mAppsRecyclerView = (AllAppsRecyclerView) findViewById(R.id.apps_list_view);
        mAppsRecyclerView.setApps(mApps);
        //GridLayoutManager
        mAppsRecyclerView.setLayoutManager(mLayoutManager);
        //AllAppsGridAdapter
        mAppsRecyclerView.setAdapter(mAdapter);
        mAppsRecyclerView.setHasFixedSize(true);
        mAppsRecyclerView.addOnScrollListener(mElevationController);
        mAppsRecyclerView.setElevationController(mElevationController);

        if (mItemDecoration != null) {
            mAppsRecyclerView.addItemDecoration(mItemDecoration);
        }

        FocusedItemDecorator focusedItemDecorator = new FocusedItemDecorator(mAppsRecyclerView);
        mAppsRecyclerView.addItemDecoration(focusedItemDecorator);
        mAppsRecyclerView.preMeasureViews(mAdapter);
        mAdapter.setIconFocusListener(focusedItemDecorator.getFocusListener());
    }
```
onFinishInflate函数在加载完xml文件时就会调用，AllAppsRecyclerView将显示App列表,这样应用程序的图标就会显示在屏幕上。

# Launcher应用图标点击过程

下面开始了解在Launcher上点击一个应用图标的过程。

Lanucher的xml布局文件会加载AllAppsContainerView这个控件如下：

packages/apps/Launcher3/src/com/android/launcher3/allapps/AllAppsContainerView.java

```java
    public AllAppsContainerView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        Resources res = context.getResources();
        //1
        mLauncher = Launcher.getLauncher(context);
        mSectionNamesMargin = res.getDimensionPixelSize(R.dimen.all_apps_grid_view_start_margin);
        mApps = new AlphabeticalAppsList(context);
        //2
        mAdapter = new AllAppsGridAdapter(mLauncher, mApps, mLauncher, this);
        mApps.setAdapter(mAdapter);
        mLayoutManager = mAdapter.getLayoutManager();
        mItemDecoration = mAdapter.getItemDecoration();
        DeviceProfile grid = mLauncher.getDeviceProfile();
        if (FeatureFlags.LAUNCHER3_ALL_APPS_PULL_UP && !grid.isVerticalBarLayout()) {
            mRecyclerViewBottomPadding = 0;
            setPadding(0, 0, 0, 0);
        } else {
            mRecyclerViewBottomPadding =
                    res.getDimensionPixelSize(R.dimen.all_apps_list_bottom_padding);
        }
        mSearchQueryBuilder = new SpannableStringBuilder();
        Selection.setSelection(mSearchQueryBuilder, 0);
    }
```

1. 获取Launcher的实例
2. 创建AllAppsGridAdapter的实例，并传入Launcher对象

接下来看AllAppsGridAdapter，它是RecyclerView的Adapter，直接看它的onCreateViewHolder方法。

packages/apps/Launcher3/src/com/android/launcher3/allapps/AllAppsGridAdapter.java

```java
        @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        switch (viewType) {
            case VIEW_TYPE_SECTION_BREAK:
                return new ViewHolder(new View(parent.getContext()));
            case VIEW_TYPE_ICON:
                /* falls through */
            case VIEW_TYPE_PREDICTION_ICON: {
                //1
                BubbleTextView icon = (BubbleTextView) mLayoutInflater.inflate(
                        R.layout.all_apps_icon, parent, false);
                //2
                icon.setOnClickListener(mIconClickListener);
                icon.setOnLongClickListener(mIconLongClickListener);
                icon.setLongPressTimeout(ViewConfiguration.get(parent.getContext())
                        .getLongPressTimeout());
                icon.setOnFocusChangeListener(mIconFocusListener);
}

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        switch (holder.getItemViewType()) {
            case VIEW_TYPE_ICON: {
                AppInfo info = mApps.getAdapterItems().get(position).appInfo;
                BubbleTextView icon = (BubbleTextView) holder.mContent;
                //3
                icon.applyFromApplicationInfo(info);
                icon.setAccessibilityDelegate(mLauncher.getAccessibilityDelegate());
                break;
            }
}
```

1. 实例化BubbleTextView就是Launcher上的应用程序图标控件
2. BubbleTextView设置点击事件，mIconClickListener就是传入的Launcher对象，所有点击应用图标就会回调Launcher的onClick方法
3. 给BubbleTextView设置tag为AppInfo

下面看Launcher.java的onClick方法：

packages/apps/Launcher3/src/com/android/launcher3/Launcher.java

```java

    public void onClick(View v) {
        
        ···

        Object tag = v.getTag();
        //ShortcutInfo
        if (tag instanceof ShortcutInfo) {
            onClickAppShortcut(v);
        //AppInfo
        } else if (tag instanceof AppInfo) {
            startAppShortcutOrInfoActivity(v);
        }
        ···
    }
```

当tag是AppInfo时调用startAppShortcutOrInfoActivity方法

```java
private void startAppShortcutOrInfoActivity(View v) {
   ItemInfo item = (ItemInfo) v.getTag();
   Intent intent = item.getIntent();
   if (intent == null) {
       throw new IllegalArgumentException("Input must have a valid intent");
   }
   //
   boolean success = startActivitySafely(v, intent, item);
   getUserEventDispatcher().logAppLaunch(v, intent);
}
```
startAppShortcutOrInfoActivity调用startActivitySafely来启动应用程序。

```java
    public boolean startActivitySafely(View v, Intent intent, ItemInfo item) {
        
        ···

        // Prepare intent
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        if (v != null) {
            intent.setSourceBounds(getViewBounds(v));
        }
        try {
            if (Utilities.ATLEAST_MARSHMALLOW && item != null
                    && (item.itemType == Favorites.ITEM_TYPE_SHORTCUT
                    || item.itemType == Favorites.ITEM_TYPE_DEEP_SHORTCUT)
                    && ((ShortcutInfo) item).promisedIntent == null) {
                // Shortcuts need some special checks due to legacy reasons.
                startShortcutIntentSafely(intent, optsBundle, item);
            } else if (user == null || user.equals(UserHandleCompat.myUserHandle())) {
                // Could be launching some bookkeeping activity
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

# 参考资料

* [ Android系统默认Home应用程序（Launcher）的启动过程源代码分析](http://blog.csdn.net/luoshengyang/article/details/6767736)
* [Android系统启动流程（四）Launcher启动过程与系统启动流程](http://liuwangshu.cn/framework/booting/4-launcher.html)


