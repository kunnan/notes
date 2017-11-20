**Android 7.1 PackageManagerService①启动过程**

# 概述

PackageManagerService 是 Android 中重要的服务之一，它主要负责系统中 APK 包的管理，应用程序的安装、卸载、信息查询等功能。

![pkms_um](http://onke0yoit.bkt.clouddn.com/pkms_uml-1.png)

上图主要展示了 PackageManagerService 及客户端的通信方式，以及类的继承方式。

PKMS 的客户端必须通过 PackageManager 才能发送请求给 PKMS，可认为 PackageManager 是 PKMS 的代理对象。

PackageManager 是一个抽象类，它的实现类是 ApplicationPackageManager。当客户端利用 Context 的 getPackageManager 函数获取 PackageManager 时，获取的就是 ApplicationPackageManager。

ApplicationPackageManager 与 PKMS 的交互是基于 Binder 通信的，利用 AIDL 文件封装了实现细节。看下 ApplicationPackageManager 构造函数，ApplicationPackageManager 持有了 IPackageManager 对象。

```java
    ApplicationPackageManager(ContextImpl context,
                              IPackageManager pm) {
        mContext = context;
        mPM = pm;
    }
```

IPackageManager 对象是如何得到的？

在ActivityThread.java 中的 handleBindApplication 函数中有如下代码：

```java
try {
//getPackageManager()获取到 IPackageManager 对象
 ii = new ApplicationPackageManager(null, getPackageManager())
         .getInstrumentationInfo(data.instrumentationName, 0);
} catch (PackageManager.NameNotFoundException e) {
 throw new RuntimeException(
         "Unable to find instrumentation info for: " + data.instrumentationName);
}

public static IPackageManager getPackageManager() {
   if (sPackageManager != null) {
       //Slog.v("PackageManager", "returning cur default = " + sPackageManager);
       return sPackageManager;
   }
   // 返回BindProxy对象
   IBinder b = ServiceManager.getService("package");
   // 将BinderProxy对象，转换成实际的业务代理对象
   sPackageManager = IPackageManager.Stub.asInterface(b);
   return sPackageManager;
}
```
>IPackageManager是PKMS的业务代理，其中服务端和客户端通信的业务函数由 
framework/base/core/java/android/content/pm/IPackageManager.aidl文件来定义。如上图所示，PKMS继承自IPackageManager.Stub，其实上也是基于Binder的服务端。

PackageManagerService 是系统启动的时候由 SystemServer 启动的，启动它就会执行应用程序的安装过程。下面主要介绍 SystemServer 启动 PackageManagerServiced的过程。


# SystemServer 启动 PackageManagerService

frameworks/base/services/java/com/android/server/SystemServer.java

```java
public static void main(String[] args) {
   new SystemServer().run();
}

private void run() {
       
    ···

   // Start services.
   try {
       //启动系统引导服务
       startBootstrapServices();
       //启动系统核心服务
       startCoreServices();
       //启动其他服务
       startOtherServices();
   } 
   
   ···
}
```

frameworks/base/services/java/com/android/server/SystemServer.java

```java
private void startBootstrapServices() {
   // Wait for installd to finish starting up so that it has a chance to
   // create critical directories such as /data/user with the appropriate
   // permissions.  We need this to complete before we initialize other services.
   Installer installer = mSystemServiceManager.startService(Installer.class);

   ···

   // Only run "core" apps if we're encrypting the device.
   String cryptState = SystemProperties.get("vold.decrypt");
   if (ENCRYPTING_STATE.equals(cryptState)) {
       Slog.w(TAG, "Detected encryption in progress - only parsing core apps");
       mOnlyCore = true;
   } else if (ENCRYPTED_STATE.equals(cryptState)) {
       Slog.w(TAG, "Device encrypted - only parsing core apps");
       mOnlyCore = true;
   }

   // Start the package manager.
   // 调用PKMS的main函数
   mPackageManagerService = PackageManagerService.main(mSystemContext, installer,
           mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore);
    //判断是否为初次启动
   mFirstBoot = mPackageManagerService.isFirstBoot();
   mPackageManager = mSystemContext.getPackageManager();
   
 if (!mOnlyCore) {
  boolean disableOtaDexopt = SystemProperties.getBoolean("config.disable_otadexopt",
          false);
  if (!disableOtaDexopt) {
      traceBeginAndSlog("StartOtaDexOptService");
      try {
          OtaDexoptService.main(mSystemContext, mPackageManagerService);
      }
  }
}
}
```

下面看下 SystemServer 的 startOtherServices 方法。

frameworks/base/services/java/com/android/server/SystemServer.java

```java
private void startOtherServices() {
    ......
    if (!mOnlyCore) {
        ........
        try {
            //将调用performDexOpt:Performs dexopt on the set of packages
            mPackageManagerService.updatePackagesIfNeeded();
        }.......
        ........
        try {
            //执行Fstrim，执行磁盘维护操作，未看到详细的资料
            //可能类似于TRIM技术，将标记为删除的文件，彻底从硬盘上移除
            //而不是等到写入时再移除，目的是提高写入时效率
            mPackageManagerService.performFstrimIfNeeded();
        }
        .......
        .......
        try {
            //PKMS 进入就绪状态
            mPackageManagerService.systemReady();
        }
        ........
        .......
    }
}
```

PackageManagerService 最终会调用 systemReady 方法通知系统进入就绪状态。

# PackageManagerService 的 main 函数

frameworks/base/services/core/java/com/android/server/pm/PackageManagerService.java

```java
public static PackageManagerService main(Context context, Installer installer,
       boolean factoryTest, boolean onlyCore) {
   // 1. 检查系统属性
   PackageManagerServiceCompilerMapping.checkProperties();
  
  // 2. 调用构造函数
  //factoryTest 是否是测试版本
  // onlyCore 是否只解析系统目录
   PackageManagerService m = new PackageManagerService(context, installer,
           factoryTest, onlyCore);
    //3. 根据条件 enable 一些 app
   m.enableSystemUserPackages();
   //利用 Binder 通信，将 PKMS 注册到 ServiceManager中
   ServiceManager.addService("package", m);
   return m;
}
```

## 检查系统属性

frameworks/base/services/core/java/com/android/server/pm/PackageManagerServiceCompilerMapping.java

```java
static void checkProperties() {
   // We're gonna check all properties and collect the exceptions, so we can give a general
   // overview. Store the exceptions here.
   RuntimeException toThrow = null;

    //reson 包括 REASON_FIRST_BOOT REASON_BOOT REASON_INSTALL 等
   for (int reason = 0; reason <= PackageManagerService.REASON_LAST; reason++) {
       try {
           // 将 reason 转化为 "pm.dexopt." + REASON_STRINGS[reason]
           String sysPropName = getSystemPropertyName(reason);
           if (sysPropName == null ||
                   sysPropName.isEmpty() ||
                   sysPropName.length() > SystemProperties.PROP_NAME_MAX) {
               throw new IllegalStateException("Reason system property name \"" +
                       sysPropName +"\" for reason " + REASON_STRINGS[reason]);
           }

           // Check validity, ignore result.
           getAndCheckValidity(reason);
       } catch (Exception exc) {
           if (toThrow == null) {
               toThrow = new IllegalStateException("PMS compiler filter settings are bad.");
           }
           //收集每一个异常
           toThrow.addSuppressed(exc);
       }
   }

   if (toThrow != null) {
       throw toThrow;
   }
}
```


```java
private static String getAndCheckValidity(int reason) {
   String sysPropValue = SystemProperties.get(getSystemPropertyName(reason));
   if (sysPropValue == null || sysPropValue.isEmpty() ||
           !DexFile.isValidCompilerFilter(sysPropValue)) {
       throw new IllegalStateException("Value \"" + sysPropValue +"\" not valid "
               + "(reason " + REASON_STRINGS[reason] + ")");
   }

   // Ensure that some reasons are not mapped to profile-guided filters.
   switch (reason) {
       case PackageManagerService.REASON_SHARED_APK:
       case PackageManagerService.REASON_FORCED_DEXOPT:
           if (DexFile.isProfileGuidedCompilerFilter(sysPropValue)) {
               throw new IllegalStateException("\"" + sysPropValue + "\" is profile-guided, "
                       + "but not allowed for " + REASON_STRINGS[reason]);
           }
           break;
   }

   return sysPropValue;
}
```

## 调用 PKMS 的构造函数

PKMS 的构造函数很长，主要功能有：扫描 Android 系统中几个目标文件夹中的 APK，从而建立合适的数据结构来管理 Package 信息、四大组件、权限等各种信息。

```java
public PackageManagerService(Context context, Installer installer,
       boolean factoryTest, boolean onlyCore) {
       
    ···
       
}
```

## m.enableSystemUserPackages()

```java
    /**
     * @hide
     * @return Whether the device is running with split system user. It means the system user and
     * primary user are two separate users. Previously system user and primary user are combined as
     * a single owner user.  see @link {android.os.UserHandle#USER_OWNER}
     */
    public static boolean isSplitSystemUser() {
        return SystemProperties.getBoolean("ro.fw.system_user_split", false);
    }

    private void enableSystemUserPackages() {
        ////system和primary未分离，则退出
        if (!UserManager.isSplitSystemUser()) {
            return;
        }
        // For system user, enable apps based on the following conditions:
        // - app is whitelisted or belong to one of these groups:
        //   -- system app which has no launcher icons
        //   -- system app which has INTERACT_ACROSS_USERS permission
        //   -- system IME app
        // - app is not in the blacklist
        AppsQueryHelper queryHelper = new AppsQueryHelper(this);
        Set<String> enableApps = new ArraySet<>();
        enableApps.addAll(queryHelper.queryApps(AppsQueryHelper.GET_NON_LAUNCHABLE_APPS
                | AppsQueryHelper.GET_APPS_WITH_INTERACT_ACROSS_USERS_PERM
                | AppsQueryHelper.GET_IMES, /* systemAppsOnly */ true, UserHandle.SYSTEM));

//增加白名单里的应用，移除黑名单里的应用
        ArraySet<String> wlApps = SystemConfig.getInstance().getSystemUserWhitelistedApps();
        enableApps.addAll(wlApps);
        enableApps.addAll(queryHelper.queryApps(AppsQueryHelper.GET_REQUIRED_FOR_SYSTEM_USER,
                /* systemAppsOnly */ false, UserHandle.SYSTEM));
        ArraySet<String> blApps = SystemConfig.getInstance().getSystemUserBlacklistedApps();
        enableApps.removeAll(blApps);
        
        //得到system user已安装的应用
        Log.i(TAG, "Applications installed for system user: " + enableApps);
        List<String> allAps = queryHelper.queryApps(0, /* systemAppsOnly */ false,
                UserHandle.SYSTEM);
        final int allAppsSize = allAps.size();
        synchronized (mPackages) {
            for (int i = 0; i < allAppsSize; i++) {
                String pName = allAps.get(i);
                PackageSetting pkgSetting = mSettings.mPackages.get(pName);
                // Should not happen, but we shouldn't be failing if it does
                if (pkgSetting == null) {
                    continue;
                }
                ///从已安装中筛选出可使用的
                boolean install = enableApps.contains(pName);
                if (pkgSetting.getInstalled(UserHandle.USER_SYSTEM) != install) {
                    Log.i(TAG, (install ? "Installing " : "Uninstalling ") + pName
                            + " for system user");
                    pkgSetting.setInstalled(install, UserHandle.USER_SYSTEM);
                }
            }
        }
    }
```


```java

```

>PKMS 其实就是管理手机中的 APK 信息，主要工作就是解析这些 APK 信息，并用相应的数据结构存储解析出来的内容。

# 参考资料

* [http://blog.csdn.net/gaugamela/article/details/52619720](http://blog.csdn.net/gaugamela/article/details/52619720)

