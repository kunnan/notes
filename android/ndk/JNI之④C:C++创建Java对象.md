**JNI之④C/C++创建Java对象**

[TOC]

# 步骤

1. 获取Java对象的jclass
2. 获取构造函数的id，方法名使用<init>
3. NewObject创建Java对象jobject
4. 获取并调用jobject中的方法

# 示例

```java
package com.michael.ndk.write;

/**
 * C访问Java字段和方法
 * Created by liuguoquan on 2017/1/3.
 */

public class WriteJava {
  /**
   * C/C++ 调用Java对象
   */
  public native long c2JavaClass();

  static {
    System.loadLibrary("native-lib");
  }
}

```

```cpp
extern "C"
/**
 * C/C++ 创建Java对象
 */
JNIEXPORT jlong JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaClass(JNIEnv *env, jobject instance) {

    //获取Date jclass
    jclass clz = env->FindClass("java/util/Date");
    //获取构造方法id
    jmethodID methodId = env->GetMethodID(clz,"<init>","()V");
    //创建Date对象
    jobject obj = env->NewObject(clz,methodId);
    //获取getTime的方法ID
    jmethodID getTimeId = env->GetMethodID(clz,"getTime","()J");
    //调用getTime方法
    jlong time = env->CallLongMethod(obj,getTimeId);
    LOGI("time = %ld\n",time);
    return time;
}
```

[JNI开发系列④C语言调用构造方法](http://www.jianshu.com/p/b0403771944f)



