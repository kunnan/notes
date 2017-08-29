**Android启动流程③SystemServer启动过程**

[TOC]

上一篇分析了Zygote进程启动过程，了解到Zygote进程启动了 SyetemServer 进程，接下来开始学习SyetemServer进程的启动过程。

# Zygote 启动 SystemServer

ZygoteInit 在 main 函数中通过调用 startSystemServer 函数启动 SystemServer进程。

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

```java
private static boolean startSystemServer(String abiList, String socketName)
       throws MethodAndArgsCaller, RuntimeException {
···
   /* For child process */
   if (pid == 0) {
       if (hasSecondZygote(abiList)) {
           waitForSecondaryZygote(socketName);
       }
        
       handleSystemServerProcess(parsedArgs);
   }

   return true;
}
```
代码可知实际上是调用了 handleSystemServerProcess 来启动 SystemServer进程。

# SystemServer 启动过程

```Java
    /**
     * Finish remaining work for the newly forked system server process.
     */
    private static void handleSystemServerProcess(
            ZygoteConnection.Arguments parsedArgs)
            throws ZygoteInit.MethodAndArgsCaller {
        //关闭从zygote继承下来的socket
        closeServerSocket();

···
        //加载SystemServer对应的文件
        final String systemServerClasspath = Os.getenv("SYSTEMSERVERCLASSPATH");
        if (systemServerClasspath != null) {
            performSystemServerDexOpt(systemServerClasspath);
        }

        if (parsedArgs.invokeWith != null) {
        
        ···    
        
        } else {
            ClassLoader cl = null;
            //利用systemServerClass对应的路径构建对应的ClassLoader
            if (systemServerClasspath != null) {
                cl = createSystemServerClassLoader(systemServerClasspath,
                                                   parsedArgs.targetSdkVersion);

                Thread.currentThread().setContextClassLoader(cl);
            }

            /*
             * 将剩余参数传递到SystemServer.
             */
            RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
        }
    }
```
handleSystemServerProcess函数的主要工作是：

* 关闭从zygote继承下来的socket
* 利用systemServerClass对应的路径构建对应的ClassLoader
* 调用RuntimeInit.zygoteInit函数将剩余参数传递到SystemServer.

接下来进入到 RuntimeInit.zygoteInit函数

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```Java
public static final void zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
       throws ZygoteInit.MethodAndArgsCaller {
    //常规初始化化如日期、设置HTTP User-Agent
   commonInit();
   //启动Binder线程池
   nativeZygoteInit();
   //
   applicationInit(targetSdkVersion, argv, classLoader);
}
```

## nativeZygoteInit()

nativeZygoteInit 很明显是一个 native 函数，它的实现在 frameworks/base/core/jni/AndroidRuntime.cpp 中，下面来看下它的实现，只有一行代码就是 gCurRuntime 调用 onZygoteInit 函数，gCurRuntime 指向AndroidRuntime 对象。

```Java
static AndroidRuntime* gCurRuntime = NULL;

static void com_android_internal_os_RuntimeInit_nativeZygoteInit(JNIEnv* env, jobject clazz)
{
    gCurRuntime->onZygoteInit();
}

```

由于SystemServer进程由zygote进程fork出来，于是system server进程中也存在gCurRuntime对象，类型为AppRuntime。至此我们知道，Native函数中gCurRuntime->onZygoteInit将调用AppRuntime中的onZygoteInit。

frameworks/base/cmds/app_process/app_main.cpp

```Java
class AppRuntime : public AndroidRuntime
{

···

    virtual void onZygoteInit()
    {
        sp<ProcessState> proc = ProcessState::self();
        ALOGV("App process: starting thread pool.\n");
        //启动一个Binder线程池，用于SystemServer与其他进程通信
        proc->startThreadPool();
    }
};

···

}
```

## applicationInit

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```Java
    private static void applicationInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
            throws ZygoteInit.MethodAndArgsCaller {
        
        nativeSetExitWithoutCleanup(true);

        //设置堆上限
        VMRuntime.getRuntime().setTargetHeapUtilization(0.75f);
        //设置目标sdk
        VMRuntime.getRuntime().setTargetSdkVersion(targetSdkVersion);

        // Remaining arguments are passed to the start class's static main
        invokeStaticMain(args.startClass, args.startArgs, classLoader);
    }
```

applicationInit 函数主要通过调用 invokeStaticMain 函数来实现。

```Java
    private static void invokeStaticMain(String className, String[] argv, ClassLoader classLoader)
            throws ZygoteInit.MethodAndArgsCaller {
        Class<?> cl;
   //className为进行初始化工作的进程类名
    //在SystemServer初始化时，为com.android.server.SystemServer
        try {
            cl = Class.forName(className, true, classLoader);
        } catch (ClassNotFoundException ex) {
            throw new RuntimeException(
                    "Missing class when invoking static main " + className,
                    ex);
        }

        Method m;
        try {
        //找到SystemServer的main方法
            m = cl.getMethod("main", new Class[] { String[].class });
        } catch (NoSuchMethodException ex) {
            throw new RuntimeException(
                    "Missing static main on " + className, ex);
        } catch (SecurityException ex) {
            throw new RuntimeException(
                    "Problem getting static main on " + className, ex);
        }

        int modifiers = m.getModifiers();
        if (! (Modifier.isStatic(modifiers) && Modifier.isPublic(modifiers))) {
            throw new RuntimeException(
                    "Main method is not public and static on " + className);
        }

        //将main传入到MethodAndArgsCaller异常中并抛出该异常
        throw new ZygoteInit.MethodAndArgsCaller(m, argv);
    }
```

invokeStaticMain 函数的主要作用：

* 根据类名创建一个类，实际上是 SystemServer 类。
* 找到 SystemServer 类的 mian 方法。
* 将 main 方法传入到 MethodAndArgsCaller 异常中并抛出异常。

那么这个抛出的异常 MethodAndArgsCaller 在哪里捕获呢？注释已经提示在 ZygoteInit.java的main函数 中。

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

```Java
public static void main(String argv[]) {
    try {
        ........
        if (startSystemServer) {
            startSystemServer(abiList, socketName);
        }

        Log.i(TAG, "Accepting command socket connections");
        runSelectLoop(abiList);

        closeServerSocket();
    } catch (MethodAndArgsCaller caller) {
        //调用MethodAndArgsCaller的run方法
        caller.run();
    } catch (RuntimeException ex) {
        Log.e(TAG, "Zygote died with exception", ex);
        closeServerSocket();
        throw ex;
    }
}

```

捕获 MethodAndArgsCaller 异常后调用 MethodAndArgsCaller 的run ff。

```Java
public void run() {
    try {
        mMethod.invoke(null, new Object[] { mArgs });
    } catch (IllegalAccessException ex) {
        throw new RuntimeException(ex);
    } catch (InvocationTargetException ex) {
        ......
    }
}
```

run 方法利用反射调用对应类的 main 方法，这里是 SystemServer.java 的 main 方法。

>这里的问题是，为什么不在RuntimeInit.java的invokeStaticMain中，直接利用反射调用每个类的main方法？
参考invokeStaticMain中抛出异常的注释，注意到，我们此时运行在SystemServer进程中。由于zygote进程fork的原因，SystemServer调用到invokeStaticMain时，整个堆栈实际上包含了大量zygote进程复制过来的调用信息。此时，我们通过抛异常捕获的方式，让位于栈底的ZygoteInit.main函数来进行处理，可起到刷新整个调用栈的作用（旧的无用调用出栈）。

# SystemServer 工作过程

frameworks/base/services/java/com/android/server/SystemServer.java

```Java
public static void main(String[] args) {
   new SystemServer().run();
}
```

SystemServer 的 main 方法直接创建一个 SystemServer 对象，并调用其 run 方法。

```Java
private void run() {

       //设置一些属性
       
        ···

       // 确保主线程的优先级
       android.os.Process.setThreadPriority(
           android.os.Process.THREAD_PRIORITY_FOREGROUND);
       android.os.Process.setCanSelfBackground(false);
       //初始化SystemServer的looper
       Looper.prepareMainLooper();

       // 加载native库文件
       System.loadLibrary("android_servers");

       // 初始化系统环境变量
       createSystemContext();

       // 创建SystemServiceManager,用于创建和管理系统服务
       mSystemServiceManager = new SystemServiceManager(mSystemContext);
       mSystemServiceManager.setRuntimeRestarted(mRuntimeRestart);
       LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);

   // Start services.
   try {
       //启动系统引导服务
       startBootstrapServices();
       //启动系统核心服务
       startCoreServices();
       //启动其他服务
       startOtherServices();
   } catch (Throwable ex) {
       Slog.e("System", "******************************************");
       Slog.e("System", "************ Failure starting system services", ex);
       throw ex;
   } finally {
       Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
   }

   //启动Looper 处理消息
   Looper.loop();
   throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

run 方法的主要作用：

* 设置主线程的优先级
* 初始化SystemServer的looper
* 加载 native 库文件
* 初始化系统环境变量 mSystemContext
* 创建SystemServiceManager,用于创建和管理系统服务
* 启动各种系统服务
* 启动 Looper。

至此 SystemServer 的启动及工作过程就完成了。

下图列出了部分系统服务以及它们的作用：

![引导服务](http://7xs7a3.com1.z0.glb.clouddn.com/引导服务.png)
![核心服务](http://7xs7a3.com1.z0.glb.clouddn.com/核心服务.png)

下面介绍一下SystemServiceManager启动系统服务的简要过程，以PowerManagerService启动为例：

```
mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);
```
SystemServiceManager 的 startService 函数如下所示。

frameworks/base/services/core/java/com/android/server/SystemServiceManager.java

```Java
    /**
     * Creates and starts a system service. The class must be a subclass of
     * {@link com.android.server.SystemService}.
     *
     * @param serviceClass A Java class that implements the SystemService interface.
     * @return The service instance, never null.
     * @throws RuntimeException if the service fails to start.
     */
    public <T extends SystemService> T startService(Class<T> serviceClass) {
        try {
            final String name = serviceClass.getName();
            Slog.i(TAG, "Starting " + name);
            Trace.traceBegin(Trace.TRACE_TAG_SYSTEM_SERVER, "StartService " + name);

            // 1 Create the service.
            final T service;
            try {
                Constructor<T> constructor = serviceClass.getConstructor(Context.class);
                service = constructor.newInstance(mContext);
            } catch (InstantiationException ex) {
                throw new RuntimeException("Failed to create service " + name
                        + ": service could not be instantiated", ex);
            }
            
            // 2 Register it.
            mServices.add(service);

            // 3 Start it.
            try {
                service.onStart();
            } catch (RuntimeException ex) {
                throw new RuntimeException("Failed to start service " + name
                        + ": onStart threw an exception", ex);
            }
            return service;
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
        }
    }
```

* 1.创建SystemService
* 2.将SystemService添加到mServices中，mServices是一个存储SystemService类型的集合
* 3.调用onstart启动服务，并返回服务。


# 参考资料

* [Android系统启动流程（三）解析SyetemServer进程启动过程
](http://liuwangshu.cn/framework/booting/3-syetemserver.html)
* [Android6.0 SystemServer进程](http://blog.csdn.net/Gaugamela/article/details/52261075)

