**Android7.1应用程序进程启动过程**

[TOC]

在Android应用程序框架层中，是由ActivityManagerService组件负责为Android应用程序创建新的进程的，它本来也是运行在一个独立的进程之中，不过这个进程是在系统启动的过程中创建的。ActivityManagerService组件一般会在什么情况下会为应用程序创建一个新的进程呢？当系统决定要在一个新的进程中启动一个Activity或者Service时，它就会创建一个新的进程了，然后在这个新的进程中启动这个Activity或者Service。在应用程序创建过程中除了获取虚拟机实例，还可以获得Binder线程池和消息循环，这样运行在应用进程中应用程序就可以方便的使用Binder进行进程间通信以及消息处理机制了。先给出应用程序进程启动过程的时序图，然后对每一个步骤进行详细分析，如下图所示。

![Activity进程启动过程](http://onke0yoit.bkt.clouddn.com/Activity进程启动过程.png)


# 应用程序进程创建过程

## 发送创建进程请求

 ActivityManagerService启动新的进程是从其成员函数startProcessLocked开始的，它向Zygote进程发送请求：

frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

```java
private final void startProcessLocked(ProcessRecord app, String hostingType,
        String hostingNameStr, String abiOverride, String entryPoint, String[] entryPointArgs) {
    ···

    try {
        try {
            final int userId = UserHandle.getUserId(app.uid);
            AppGlobals.getPackageManager().checkPackageStartable(app.info.packageName, userId);
        } catch (RemoteException e) {
            throw e.rethrowAsRuntimeException();
        }

        //1.获取用户ID
        int uid = app.uid;
        //2.用户组ID
        int[] gids = null;
        int mountExternal = Zygote.MOUNT_EXTERNAL_NONE;
        if (!app.isolated) {
            int[] permGids = null;
            try {
               ···
            /*
             * 创建gids和赋值
             */
            if (ArrayUtils.isEmpty(permGids)) {
                gids = new int[2];
            } else {
                gids = new int[permGids.length + 2];
                System.arraycopy(permGids, 0, gids, 2, permGids.length);
            }
            gids[0] = UserHandle.getSharedAppGid(UserHandle.getAppId(uid));
            gids[1] = UserHandle.getUserGid(UserHandle.getUserId(uid));
        }
    
    ···
    
        boolean isActivityProcess = (entryPoint == null);
        //指定className
        if (entryPoint == null) entryPoint = "android.app.ActivityThread";
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "Start proc: " +
                app.processName);
        checkTime(startTime, "startProcess: asking zygote to start proc");
        //传递参数到Process中
        Process.ProcessStartResult startResult = Process.start(entryPoint,
                app.processName, uid, uid, gids, debugFlags, mountExternal,
                app.info.targetSdkVersion, app.info.seinfo, requiredAbi, instructionSet,
                app.info.dataDir, entryPointArgs);
      ···
}
```
startProcessLocked方法的主要作用：

1. 获取用户ID
2. 获取用户组ID
3. 指定要启动的class为`android.app.ActivityThread`
4. 传递上述参数到Process.java中

接下来看Process.java的start方法：

frameworks/base/core/java/android/os/Process.java

```java
public static final ProcessStartResult start(final String processClass,
                             final String niceName,
                             int uid, int gid, int[] gids,
                             int debugFlags, int mountExternal,
                             int targetSdkVersion,
                             String seInfo,
                             String abi,
                             String instructionSet,
                             String appDataDir,
                             String[] zygoteArgs) {
   try {
        //processClass的值是“android.app.ActivityThread”
       return startViaZygote(processClass, niceName, uid, gid, gids,
               debugFlags, mountExternal, targetSdkVersion, seInfo,
               abi, instructionSet, appDataDir, zygoteArgs);
   } catch (ZygoteStartFailedEx ex) {
       Log.e(LOG_TAG,
               "Starting VM process through Zygote failed");
       throw new RuntimeException(
               "Starting VM process through Zygote failed", ex);
   }
}
```
这里processClass的值是“android.app.ActivityThread”，通过函数名可以知道通过Zygote进程启动。start函数中调用startViaZygote方法。

frameworks/base/core/java/android/os/Process.java

```java
private static ProcessStartResult startViaZygote(final String processClass,
                              final String niceName,
                              final int uid, final int gid,
                              final int[] gids,
                              int debugFlags, int mountExternal,
                              int targetSdkVersion,
                              String seInfo,
                              String abi,
                              String instructionSet,
                              String appDataDir,
                              String[] extraArgs)
                              throws ZygoteStartFailedEx {
    synchronized(Process.class) {
        ArrayList<String> argsForZygote = new ArrayList<String>();
        //写入参数到argsForZygote中
        argsForZygote.add("--runtime-args");
        argsForZygote.add("--setuid=" + uid);
        argsForZygote.add("--setgid=" + gid);
        
        ···

        // --setgroups is a comma-separated list
        if (gids != null && gids.length > 0) {
            StringBuilder sb = new StringBuilder();
            sb.append("--setgroups=");

            int sz = gids.length;
            for (int i = 0; i < sz; i++) {
                if (i != 0) {
                    sb.append(',');
                }
                sb.append(gids[i]);
            }

            argsForZygote.add(sb.toString());
        }

       ···

        argsForZygote.add(processClass);

        if (extraArgs != null) {
            for (String arg : extraArgs) {
                argsForZygote.add(arg);
            }
        }

        return zygoteSendArgsAndGetResult(openZygoteSocketIfNeeded(abi), argsForZygote);
    }
}
```
  这个函数将创建进程的参数放到argsForZygote列表中去，然后调用zygoteSendArgsAndGetResult函数进一步操作，它的第一个参数调用openZygoteSocketIfNeeded方法，第二个参数为参数集合argsForZygote，先看openZygoteSocketIfNeeded方法。

frameworks/base/core/java/android/os/Process.java

```java
    private static ZygoteState openZygoteSocketIfNeeded(String abi) 
    throws ZygoteStartFailedEx {
        //如果没有可用使用的socket则创建一个
        if (primaryZygoteState == null || primaryZygoteState.isClosed()) {
            try {
            //ZYGOTE_SOCKET 运行在64位进程中
                primaryZygoteState = ZygoteState.connect(ZYGOTE_SOCKET);
            } catch (IOException ioe) {
                throw new ZygoteStartFailedEx("Error connecting to primary zygote", ioe);
            }
        }
        
        //如果有可用的socket则使用这个
        if (primaryZygoteState.matches(abi)) {
            return primaryZygoteState;
        }

        
        if (secondaryZygoteState == null || secondaryZygoteState.isClosed()) {
            try {
            //运行在32位进程中
            secondaryZygoteState = ZygoteState.connect(SECONDARY_ZYGOTE_SOCKET);
            } catch (IOException ioe) {
                throw new ZygoteStartFailedEx("Error connecting to secondary zygote", ioe);
            }
        }

        if (secondaryZygoteState.matches(abi)) {
            return secondaryZygoteState;
        }

        throw new ZygoteStartFailedEx("Unsupported zygote ABI: " + abi);
    }
```

这里创建的Socket由frameworks/base/core/java/com/android/internal/os/ZygoteInit.java文件中的ZygoteInit类在runSelectLoopMode函数监听的。

再来看zygoteSendArgsAndGetResult函数

frameworks/base/core/java/android/os/Process.java

```java
private static ProcessStartResult zygoteSendArgsAndGetResult(
        ZygoteState zygoteState, ArrayList<String> args)
        throws ZygoteStartFailedEx {
    try {
 
 ···
        //获取socket的outputstream和inputstream
        final BufferedWriter writer = zygoteState.writer;
        final DataInputStream inputStream = zygoteState.inputStream;

        writer.write(Integer.toString(args.size()));
        writer.newLine();

        for (int i = 0; i < sz; i++) {
            String arg = args.get(i);
            writer.write(arg);
            writer.newLine();
        }
        
        //将消息发往zygote
        writer.flush();

        // Should there be a timeout on this?
        ProcessStartResult result = new ProcessStartResult();
        
        result.pid = inputStream.readInt();
        result.usingWrapper = inputStream.readBoolean();
        
        //阻塞直到收到结果，pid大于0则说明进程启动成功
        if (result.pid < 0) {
            throw new ZygoteStartFailedEx("fork() failed");
        }
        return result;
    } catch (IOException ex) {
        zygoteState.close();
        throw new ZygoteStartFailedEx(ex);
    }
}
```
zygoteSendArgsAndGetResult方法的主要作用是将传入的启动参数argsForZygote通过ZygoteState中的输出流发送到zygote。

## 接收并创建进程过程

Zygote进程就会收到一个创建新的应用程序进程的请求，回到ZygoteInit的main函数，如下所示。

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

```java
public static void main(String argv[]) {
       ...
        try {
         ...       
            //注册Zygote用的Socket
            registerZygoteSocket(socketName);//1
           ...
           //预加载类和资源
           preload();//2
           ...
            if (startSystemServer) {
            //启动SystemServer进程
                startSystemServer(abiList, socketName);//3
            }
            Log.i(TAG, "Accepting command socket connections");
            //等待客户端请求
            runSelectLoop(abiList);//4
            closeServerSocket();
        } catch (MethodAndArgsCaller caller) {
            caller.run();
        } catch (RuntimeException ex) {
            Log.e(TAG, "Zygote died with exception", ex);
            closeServerSocket();
            throw ex;
        }
    }
```
runSelectLoopMode函数来监听请求

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

```java
    private static void runSelectLoop(String abiList) throws MethodAndArgsCaller {
        ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
        ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();

        fds.add(sServerSocket.getFileDescriptor());
        peers.add(null);

        while (true) {
            StructPollfd[] pollFds = new StructPollfd[fds.size()];
            for (int i = 0; i < pollFds.length; ++i) {
                pollFds[i] = new StructPollfd();
                pollFds[i].fd = fds.get(i);
                pollFds[i].events = (short) POLLIN;
            }
            try {
                Os.poll(pollFds, -1);
            } catch (ErrnoException ex) {
                throw new RuntimeException("poll failed", ex);
            }
            for (int i = pollFds.length - 1; i >= 0; --i) {
                if ((pollFds[i].revents & POLLIN) == 0) {
                    continue;
                }
                if (i == 0) {
                    ZygoteConnection newPeer = acceptCommandPeer(abiList);
                    peers.add(newPeer);
                    fds.add(newPeer.getFileDesciptor());
                } else {
                    //当有新请求时会执行下面代码
                    boolean done = peers.get(i).runOnce();
                    if (done) {
                        peers.remove(i);
                        fds.remove(i);
                    }
                }
            }
        }
    }
```
当有ActivityManagerService的请求数据到来时会调用`peers.get(i).runOnce()`，实际上就是调用ZygoteConnection.java的runOnce()函数来处理数据。

frameworks/base/core/java/com/android/internal/os/ZygoteConnection.java

```java
boolean runOnce() throws ZygoteInit.MethodAndArgsCaller {

        Arguments parsedArgs = null;
        FileDescriptor[] descriptors;
        try {
            //1 获取应用程序进程的启动参数
            args = readArgumentList();
            descriptors = mSocket.getAncillaryFileDescriptors();
        } catch (IOException ex) {
            Log.w(TAG, "IOException on command socket " + ex.getMessage());
            closeSocket();
            return true;
        }
...
        try {
            //2 readArgumentList函数返回的字符串封装到Arguments对象parsedArgs中
            parsedArgs = new Arguments(args);//2
        .......
        //3 创建应用程序进程，参数为parsedArgs中存储的应用进程启动参数，返回值为pid
        pid = Zygote.forkAndSpecialize(parsedArgs.uid, parsedArgs.gid, parsedArgs.gids, parsedArgs.debugFlags, rlimits, parsedArgs.mountExternal, parsedArgs.seInfo, parsedArgs.niceName, fdsToClose, parsedArgs.instructionSet, parsedArgs.appDataDir);
    }  catch (ErrnoException ex) {
        .......   
    } catch (IllegalArgumentException ex) {
        ....... 
    } catch (ZygoteSecurityException ex) {
        ....... 
    }

    try {
        //创建成功
        if (pid == 0) {
            // in child 子进程也就是应用程序进程
            IoUtils.closeQuietly(serverPipeFd);
            serverPipeFd = null;
            //4 子进程根据参数利用handleChildProc作进一步处理
            handleChildProc(parsedArgs, descriptors, childPipeFd, newStderr);

            // should never get here, the child is expected to either
            // throw ZygoteInit.MethodAndArgsCaller or exec().
            return true;
        } else {
            // in parent...pid of < 0 means failure
            IoUtils.closeQuietly(childPipeFd);
            childPipeFd = null;
            //父进程进行一些后续操作，例如清理工作等
            return handleParentProc(pid, descriptors, serverPipeFd, parsedArgs);
        }
    } finally {
        IoUtils.closeQuietly(childPipeFd);
        IoUtils.closeQuietly(serverPipeFd);
    }
}
```
最终调用handleChildProc方法来启动这个应用程序进程。

frameworks/base/core/java/com/android/internal/os/ZygoteConnection.java

```java
private void handleChildProc(Arguments parsedArgs, FileDescriptor[] descriptors, FileDescriptor pipeFd, PrintStream newStderr) throws ZygoteInit.MethodAndArgsCaller {
    //关闭fork过来的zygote server socket
    closeSocket();
    ZygoteInit.closeServerSocket();
    //处理参数
    ........
    if (parsedArgs.invokeWith != null) {
        .......
    } else {
        //完成进程的初始化，然后抛异常
        RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, null /* classLoader */);
    }
}
```
handleChildProc函数中调用了RuntimeInit的zygoteInit函数

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```java
public static final void zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
          throws ZygoteInit.MethodAndArgsCaller {
      if (DEBUG) Slog.d(TAG, "RuntimeInit: Starting application from zygote");
      Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "RuntimeInit");
      redirectLogStreams();
      commonInit();
      nativeZygoteInit();//1 创建Binder线程池
      applicationInit(targetSdkVersion, argv, classLoader);//2 应用初始化
  }
```
frameworks/base/core/java/com/android/internal/os/RuntimeInit.java


```java
 private static void applicationInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
           throws ZygoteInit.MethodAndArgsCaller {
...
       final Arguments args;
       try {
           args = new Arguments(argv);
       } catch (IllegalArgumentException ex) {
           Slog.e(TAG, ex.getMessage());       
           return;
       }
       Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
       //args.startClass就是前面传过来的android.app.ActivityThread
       invokeStaticMain(args.startClass, args.startArgs, classLoader);//1
   }
```
applicationInit调用invokeStaticMain方法来启动进程，startClass即是前面传进来的"android.app.ActivityThread"值，表示要执行android.app.ActivityThread类的main函数。

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```java
private static void invokeStaticMain(String className, String[] argv, ClassLoader classLoader)
        throws ZygoteInit.MethodAndArgsCaller {
    Class<?> cl;
    try {
        //通过反射来获得android.app.ActivityThread类
        cl = Class.forName(className, true, classLoader);
    } catch (ClassNotFoundException ex) {
        throw new RuntimeException(
                "Missing class when invoking static main " + className,
                ex);
    }
    Method m;
    try {
        //获得ActivityThread的main函数
        m = cl.getMethod("main", new Class[] { String[].class });
    } catch (NoSuchMethodException ex) {
        throw new RuntimeException(
                "Missing static main on " + className, ex);
    }
    ...
    //将main函数传入到ZygoteInit中的MethodAndArgsCaller类的构造函数中，
    throw new ZygoteInit.MethodAndArgsCaller(m, argv);
}
```
最终MethodAndArgsCaller类内部会通过反射调用ActivityThread的main函数，这样应用程序的创建过程就完成了。

# Bindler线程池启动过程

首先来看RuntimeInit类的zygoteInit函数，如下

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```java
public static final void zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
          throws ZygoteInit.MethodAndArgsCaller {
      if (DEBUG) Slog.d(TAG, "RuntimeInit: Starting application from zygote");
      Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "RuntimeInit");
      redirectLogStreams();
      commonInit();
      nativeZygoteInit();//1
      applicationInit(targetSdkVersion, argv, classLoader);//2
  }
```

nativeZygoteInit函数会在新创建的应用程序进程中创建Binder线程池，nativeZygoteInit函数的原型为

```cpp
private static final native void nativeZygoteInit();
```

nativeZygoteInit是一个native方法，对应的函数为com_android_internal_os_RuntimeInit_nativeZygoteInit，位于frameworks/base/core/jni/AndroidRuntime.cpp中

```cpp
/*
 * JNI registration.
 */
static const JNINativeMethod gMethods[] = {
    { "nativeFinishInit", "()V",
        (void*) com_android_internal_os_RuntimeInit_nativeFinishInit },
    { "nativeZygoteInit", "()V",
        (void*) com_android_internal_os_RuntimeInit_nativeZygoteInit },
    { "nativeSetExitWithoutCleanup", "(Z)V",
        (void*) com_android_internal_os_RuntimeInit_nativeSetExitWithoutCleanup },
};
```
接下来看com_android_internal_os_RuntimeInit_nativeZygoteInit的实现，就是调用AndroidRuntime的onZygoteInit方法。

frameworks/base/core/jni/AndroidRuntime.cpp

```cpp
static AndroidRuntime* gCurRuntime = NULL;

static void com_android_internal_os_RuntimeInit_nativeZygoteInit(JNIEnv* env, jobject clazz)
{
    gCurRuntime->onZygoteInit();
}
```

```java
AndroidRuntime::AndroidRuntime(char* argBlockStart, const size_t argBlockLength) :
        mExitWithoutCleanup(false),
        mArgBlockStart(argBlockStart),
        mArgBlockLength(argBlockLength)
{
    ···
    //初始化中赋值
    gCurRuntime = this;
}
```
onZygoteInit的实现实际上是在AppRuntime类中，AppRuntime类继承AndroidRuntime。

frameworks/base/cmds/app_process/app_main.cpp

```cpp
    virtual void onZygoteInit()
    {
        sp<ProcessState> proc = ProcessState::self();
        ALOGV("App process: starting thread pool.\n");
        proc->startThreadPool();
    }
```
调用ProcessState的startThreadPool函数启动线程池，这个线程池中的线程就是用来和Binder驱动程序进行交互的了。

frameworks/native/libs/binder/ProcessState.cpp

```cpp
void ProcessState::startThreadPool()
{
    AutoMutex _l(mLock);
    if (!mThreadPoolStarted) {
        mThreadPoolStarted = true;
        spawnPooledThread(true);
    }
}
```
ProcessState类是Binder进程间通信机制的一个基础组件，它里面有一个mThreadPoolStarted 变量，来表示Binder线程池是否已经被启动过，默认值为false。在每次调用这个函数时都会先去检查这个标记，从而确保Binder线程池只会被启动一次。如果Binder线程池未被启动则设置mThreadPoolStarted为true，最后调用spawnPooledThread函数来创建线程池中的第一个线程，也就是线程池的main线程，如下所示。

frameworks/native/libs/binder/ProcessState.cpp

```cpp
void ProcessState::spawnPooledThread(bool isMain)
{
    if (mThreadPoolStarted) {
        String8 name = makeBinderThreadName();
        ALOGV("Spawning new pooled thread, name=%s\n", name.string());
        sp<Thread> t = new PoolThread(isMain);
        t->run(name.string());
    }
}
```
 这里它会创建一个PoolThread线程类，然后执行它的run函数，最终就会执行PoolThread类的threadLoop函数了。
 
frameworks/native/libs/binder/ProcessState.cpp

```cpp
class PoolThread : public Thread
{
public:
    PoolThread(bool isMain)
        : mIsMain(isMain)
    {
    }
    
protected:
    virtual bool threadLoop()
    {
        IPCThreadState::self()->joinThreadPool(mIsMain);
        return false;
    }
    
    const bool mIsMain;
};
```

这里它执行了IPCThreadState::joinThreadPool函数将当前线程注册到Binder驱动中。IPCThreadState也是Binder进程间通信机制的一个基础组件。至此我们创建的线程就加入了Binder线程池中，这样新创建的应用程序进程就支持Binder进程间通信了。

# 消息循环Looper启动过程

回到RuntimeInit的invokeStaticMain函数

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```java
private static void invokeStaticMain(String className, String[] argv, ClassLoader classLoader)
        throws ZygoteInit.MethodAndArgsCaller {
    Class<?> cl;
  ...
    throw new ZygoteInit.MethodAndArgsCaller(m, argv);
}
```

invokeStaticMain最后会抛出ZygoteInit.MethodAndArgsCaller类型的异常，这个异常将会被ZygoteInit.javademain函数捕获

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

```java
public static void main(String argv[]) {
   ...
      try {
         ...
      } catch (MethodAndArgsCaller caller) {
          caller.run();//1
      } catch (RuntimeException ex) {
          Log.e(TAG, "Zygote died with exception", ex);
          closeServerSocket();
          throw ex;
      }
  }
```

main函数捕获到MethodAndArgsCaller异常时时会执行caller的run函数

frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

```java
public static class MethodAndArgsCaller extends Exception
           implements Runnable {
       private final Method mMethod;
       private final String[] mArgs;
       public MethodAndArgsCaller(Method method, String[] args) {
           mMethod = method;
           mArgs = args;
       }
       public void run() {
           try {
               mMethod.invoke(null, new Object[] { mArgs });//1
           } catch (IllegalAccessException ex) {
               throw new RuntimeException(ex);
           }
           ...
               throw new RuntimeException(ex);
           }
       }
   }
```

mMethod指的就是ActivityThread的main函数，mArgs 指的是应用程序进程的启动参数。在注释1处调用ActivityThread的main函数

frameworks/base/core/java/android/app/ActivityThread.java

```java
 public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
        SamplingProfilerIntegration.start();
...
        //在应用程序主线程即UI线程创建消息循环
        Looper.prepareMainLooper();
        //创建ActivityThread
        ActivityThread thread = new ActivityThread();
        thread.attach(false);
        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }
        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        //启动消息循环，开始处理消息
        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

系统在应用程序进程启动完成后，就会创建一个消息循环，用来方便的使用Android的消息处理机制。

# 参考资料

* [ Android6.0 SystemServer进程](http://blog.csdn.net/gaugamela/article/details/52261075)
* [Android应用程序进程启动过程1](http://liuwangshu.cn/framework/applicationprocess/1.html)
* [Android应用程序进程启动过程的源代码分析](http://blog.csdn.net/luoshengyang/article/details/6747696)

