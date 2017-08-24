**Zygote进程启动过程**

[TOC]

上一篇分析了init进程的启动过程，最后就是创建Zygote进程，这篇我们将了解Zygote进程是什么，它由什么功能。

在Android系统中，所有的应用程序进程以及系统服务进程SystemServer都是由Zygote进程fork出来的，Zygote进程在启动时会创建DVM，因此通过Zygote进程创建的应用程序和SystemServer都可以在内部获得一个DVM的实例。

下面开始分析Android 7.0 中Zygote的启动过程：

# app_main.cpp

init进程启动Zygote是通过调用app_main.cpp中的mian函数中的AppRuntime的start函数实现的，我们从app_main.cpp中的mian函数开始分析：

目录位于frameworks/base/cmds/app_process/app_main.cpp

```cpp
int main(int argc, char* const argv[])
{
    ···

    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
    // Process command line arguments
    // ignore argv[0]
    argc--;
    argv++;

    ···

    int i;
    for (i = 0; i < argc; i++) {
        if (argv[i][0] != '-') {
            break;
        }
        if (argv[i][1] == '-' && argv[i][2] == 0) {
            ++i; // Skip --.
            break;
        }
        runtime.addOption(strdup(argv[i]));
    }

    // Parse runtime arguments.  Stop at first unrecognized option.
    bool zygote = false;
    bool startSystemServer = false;
    bool application = false;
    String8 niceName;
    String8 className;

    ++i;  // 解析输入参数
    while (i < argc) {
        const char* arg = argv[i++];
        //Start in zygote mode
        if (strcmp(arg, "--zygote") == 0) {
            //init.zygote.rc中定义了该字段
            zygote = true;
            ////记录app_process进程名的nice name，即zygote
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
        //Start the system server.init.zygote.rc中定义了该字段
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
        //The nice name for this process.
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }

    Vector<String8> args;
    if (!className.isEmpty()) {
        //启动zygote时，class name is empty，不进入该分支
        args.add(application ? String8("application") : String8("tool"));
        runtime.setClassNameAndArgs(className, argc - i, argv + i);
    } else {
        //创建dalvikCache所需的目录，并定义权限
        maybeCreateDalvikCache();

        if (startSystemServer) {
            //将start-system-server加入到启动参数
            args.add(String8("start-system-server"));
        }

        char prop[PROP_VALUE_MAX];
        if (property_get(ABI_LIST_PROPERTY, prop, NULL) == 0) {
            LOG_ALWAYS_FATAL("app_process: Unable to determine ABI list from property %s.",
                ABI_LIST_PROPERTY);
            return 11;
        }

        String8 abiFlag("--abi-list=");
        abiFlag.append(prop);
        args.add(abiFlag);

        // In zygote mode, pass all remaining arguments to the zygote
        // main() method.
        for (; i < argc; ++i) {
            args.add(String8(argv[i]));
        }
    }

    if (!niceName.isEmpty()) {
        //将app_process的进程名，替换为zygote
        runtime.setArgv0(niceName.string());
        set_process_name(niceName.string());
    }

    if (zygote) {
        //调用runtime的start函数
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        ////启动zygote没有进入这个分支，但这个分支说明，通过配置init.rc文件，其实是可以不通过zygote来启动一个进程
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
        return 10;
    }
```

* 将“start-system-server”加入到启动的参数args中
* 调用AppRuntime的start函数启动zygote进程，并将args参数传入到zygoteInit类中，这样启动zygote进程后，会将SystemServer进程启动。

这个函数的主要作用是创建一个AppRuntime的变量，然后调用它的start成员函数，AppRuntime 继承自 AndroidRuntime类，实际上调用的就是 AndroidRuntime类 的 start 函数，下面看看AndroidRuntime 的start函数。

## AndroidRuntime

目录位于frameworks/base/core/jni/AndroidRuntime.cpp

```cpp
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
   
    ···

    /* start the virtual machine */
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    //1. startVm调用用JNI_CreateVM创建出虚拟机
    if (startVm(&mJavaVM, &env, zygote) != 0) {
        return;
    }
    onVmCreated(env);

    /*
     * 2. 注册jni函数
     */
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives\n");
        return;
    }

    jclass stringClass;
    jobjectArray strArray;
    jstring classNameStr;

    stringClass = env->FindClass("java/lang/String");
    assert(stringClass != NULL);
    //创建数组
    strArray = env->NewObjectArray(options.size() + 1, stringClass, NULL);
    assert(strArray != NULL);
    //从app_main的main函数得知className为com.android.internal.os.ZygoteInit
    classNameStr = env->NewStringUTF(className);
    assert(classNameStr != NULL);
    env->SetObjectArrayElement(strArray, 0, classNameStr);

    for (size_t i = 0; i < options.size(); ++i) {
        jstring optionsStr = env->NewStringUTF(options.itemAt(i).string());
        assert(optionsStr != NULL);
        env->SetObjectArrayElement(strArray, i + 1, optionsStr);
    }

    //将"com.android.internal.os.ZygoteInit"替换为"com/android/internal/os/ZygoteInit"
    char* slashClassName = toSlashClassName(className);
    jclass startClass = env->FindClass(slashClassName);
    if (startClass == NULL) {
        ALOGE("JavaVM unable to locate class '%s'\n", slashClassName);
        /* keep going */
    } else {
        //3. 找到ZygoteInit的main函数
        jmethodID startMeth = env->GetStaticMethodID(startClass, "main",
            "([Ljava/lang/String;)V");
        if (startMeth == NULL) {
            ALOGE("JavaVM unable to find main() in '%s'\n", className);
            /* keep going */
        } else {
            //4. 通过JNI调用ZygoteInit的main函数
            env->CallStaticVoidMethod(startClass, startMeth, strArray);

#if 0
            if (env->ExceptionCheck())
                threadExitUncaughtException(env);
#endif
        }
    }
    free(slashClassName);

    ALOGD("Shutting down VM\n");
    if (mJavaVM->DetachCurrentThread() != JNI_OK)
        ALOGW("Warning: unable to detach main thread\n");
    if (mJavaVM->DestroyJavaVM() != 0)
        ALOGW("Warning: VM did not shut down cleanly\n");
}
```

该函数的主要作用是：

1. startVm调用用JNI_CreateVM创建出虚拟机
2. startReg注册JNI函数
3. 找到ZygoteInit的main函数
4. 通过JNI调用ZygoteInit的main函数

接下来，Zygote进入到Java的世界。

# ZygoteInit.java

目录位于com/android/internal/os/ZygoteInit.java

```Java
public static void main(String argv[]) {
    // Mark zygote start. only internal daemon threads are allowed there。
    ZygoteHooks.startZygoteNoThreadCreation();

    try {

        boolean startSystemServer = false;
        String socketName = "zygote";
       
       ···
       
        //创建了一个socket接口,用来和ActivityManagerService通讯
        registerZygoteSocket(socketName);
        ···
        //预加载类和资源
        preload();
        ···
        ZygoteHooks.stopZygoteNoThreadCreation();

        //启动SystemServer组件
        if (startSystemServer) {
            startSystemServer(abiList, socketName);
        }

        //zygote进程进入无限循环，处理请求
        runSelectLoop(abiList);

        closeServerSocket();
    } catch (MethodAndArgsCaller caller) {
        //runSelectLoop 异常时抛出
        caller.run();
    } catch (Throwable ex) {
        Log.e(TAG, "Zygote died with exception", ex);
        closeServerSocket();
        throw ex;
    }
}
```

1. registerZygoteSocket创建socket服务端，用来等待ActivityManagerService请求创建新的应用程序
2. preload方法预加载类和资源
3. startSystemServer函数启动SystemServer组件
4. runSelectLoop使zygote进程进入无限循环，处理请求

接下来分来看看4个步骤的实现过程：

## registerZygoteSocket

com/android/internal/os/ZygoteInit.java

```Java
private static void registerZygoteSocket(String socketName) {
   if (sServerSocket == null) {
       int fileDesc;
       //此处的socket name，就是zygote  ANDROID_SOCKET_zygote
       final String fullSocketName = ANDROID_SOCKET_PREFIX + socketName;
       try {
           String env = System.getenv(fullSocketName);
           fileDesc = Integer.parseInt(env);
       } catch (RuntimeException ex) {
           throw new RuntimeException(fullSocketName + " unset or invalid", ex);
       }

       try {
           FileDescriptor fd = new FileDescriptor();
           //获取zygote socket的文件描述符
           fd.setInt$(fileDesc);
           //将zygote socket包装成一个server socket
           sServerSocket = new LocalServerSocket(fd);
       } catch (IOException ex) {
           throw new RuntimeException(
                   "Error binding to local socket '" + fileDesc + "'", ex);
       }
   }
}
```

* 获取zygote socket的文件描述符
* 创建服务端的Socket即LocalServerSocket，等待ActivityManagerService请求创建新的应用程序

## preload

com/android/internal/os/ZygoteInit.java

```Java
    static void preload() {
        Log.d(TAG, "begin preload");
        beginIcuCachePinning();
        //读取文件framework/base/preloaded-classes，然后通过反射加载对应的类
        //需要加载数千个类，启动慢的原因之一
        preloadClasses();
        //加载一些常用的系统资源
        preloadResources();
        //图形相关的
        preloadOpenGL();
        //基础库
        preloadSharedLibraries();
        //文字相关
        preloadTextResources();
        // Ask the WebViewFactory to do any initialization 
        //that must run in the zygote process,
        // for memory sharing purposes.
        WebViewFactory.prepareWebViewInZygote();
        endIcuCachePinning();
        warmUpJcaProviders();
    }
```

## startSystemServer

com/android/internal/os/ZygoteInit.java

```Java
private static boolean startSystemServer(String abiList, String socketName) 
        throws MethodAndArgsCaller, RuntimeException {
   ····
   ····
   //准备capabilities参数
   //用来创建args数组，这个数组用来保存启动SystemServer的启动参数
   String args[] = {
       "--setuid=1000",
       "--setgid=1000",
       "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,
       1010,1018,1021,1032,
       3001,3002,3003,3006,3007,3009,3010",
       "--capabilities=" + capabilities + "," + capabilities,
       "--nice-name=system_server",
       "--runtime-args",
       "com.android.server.SystemServer",
   };
   ZygoteConnection.Arguments parsedArgs = null;

   int pid;

   try {
       ////将上面准备的参数，按照ZygoteConnection的风格进行封装
       parsedArgs = new ZygoteConnection.Arguments(args);
       ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
       ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

       //调用Zygote的forkSystemServer，主要通过fork函数在当前进程创建一个子进程，
       pid = Zygote.forkSystemServer(
               parsedArgs.uid, parsedArgs.gid,
               parsedArgs.gids,
               parsedArgs.debugFlags,
               null,
               parsedArgs.permittedCapabilities,
               parsedArgs.effectiveCapabilities);
   } catch (IllegalArgumentException ex) {
       throw new RuntimeException(ex);
   }

   //如果返回的pid 为0，也就是表示在新创建的子进程中执行的
   if (pid == 0) {
       if (hasSecondZygote(abiList)) {
           waitForSecondaryZygote(socketName);
       }
        //启动SystemServer进程
       handleSystemServerProcess(parsedArgs);
   }

   return true;
}
```

## runSelectLoop

zygote调用startSystemServer启动SystemServer进程后，就调用runSelectLoop方法，处理ServerSocket接收到的命令。

com/android/internal/os/ZygoteInit.java

```Java
private static void runSelectLoop(String abiList) throws MethodAndArgsCaller {
   ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
   ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();
    
    //将server socket加入到FileDescriptor集合
    //sServerSocket就是在registerZygoteSocket方法中创建的socket
   fds.add(sServerSocket.getFileDescriptor());
   peers.add(null);
    
    //无限循环用来等待ActivityManagerService请求Zygote进程创建新的应用程序进程
   while (true) {
        //将fds存储的信息转移到pollFds数组中
       StructPollfd[] pollFds = new StructPollfd[fds.size()];
       for (int i = 0; i < pollFds.length; ++i) {
           pollFds[i] = new StructPollfd();
           pollFds[i].fd = fds.get(i);
           pollFds[i].events = (short) POLLIN;
       }
       try {
       //等待事件到来
           Os.poll(pollFds, -1);
       } catch (ErrnoException ex) {
           throw new RuntimeException("poll failed", ex);
       }
       for (int i = pollFds.length - 1; i >= 0; --i) {
           if ((pollFds[i].revents & POLLIN) == 0) {
               continue;
           }
           //server socket最先加入fds， 因此这里是server socket收到数据
           if (i == 0) {
                //收到新的建立通信的请求，建立通信连接
               ZygoteConnection newPeer = acceptCommandPeer(abiList);
               ////加入到peers和fds
               peers.add(newPeer);
               fds.add(newPeer.getFileDesciptor());
           } else {
            //不是server socket
            //调用ZygoteConnection的runOnce函数来创建一个新的应用程序进程。
               boolean done = peers.get(i).runOnce();
               if (done) {
                //成功创建后将这个连接从Socket连接列表peers和fd列表fds中清除
                   peers.remove(i);
                   fds.remove(i);
               }
           }
       }
   }
}

private static ZygoteConnection acceptCommandPeer(String abiList) {
   try {
       return new ZygoteConnection(sServerSocket.accept(), abiList);
   } catch (IOException ex) {
       throw new RuntimeException(
               "IOException during accept()", ex);
   }
}
```

# 小结

Zygote进程的工作流程主要是：

1. app_main.cpp中main函数创建AppRuntime对象，并调用其start方法来启动Zygote进程
2. app_main.cpp中main函数调用JNI_CreateVM创建出虚拟机
3. app_main.cpp中main函数通过JNI调用ZygoteInit的mian函数进入Zygote的Java层
4. ZygoteInit调用registerZygoteSocket创建socket服务端
5. ZygoteInit通过调用runSelectLoop方法等待ActivityManagerService请求创建新的应用程序
6. ZygoteInit调用startSystemServer启动SystemServer进程

# 参考资料

* [Android系统启动流程（二）解析Zygote进程启动过程](http://liuwangshu.cn/framework/booting/2-zygote.html)
* [Android6.0 Zygote进程](http://blog.csdn.net/gaugamela/article/details/52252916)
* [Android系统进程Zygote启动过程的源代码分析](http://blog.csdn.net/luoshengyang/article/details/6768304)

