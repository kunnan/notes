**JNI之⑤C/C++处理Java对象引用**

[TOC]

# 数组引用的处理

在Java中,使用new关键字创建对象,创建之后我们就可以随意使用这个对象,我们无需关心这个对象是什么时候被回收的 ,对象的回收已经托管到了JVM的GC,由GC来帮我们回收无引用的对象。将对象引用传递给C/C++时，C/C++层就会持有Java对象，如果不进行妥善处理，对象多了就会出现内存泄漏问题，所以在C/C++层使用Java对象后，需要释放这个引用 。

```java
package com.michael.ndk.write;

/**
 * 
 * Created by liuguoquan on 2017/1/3.
 */

public class WriteJava {

  // 对数组进行排序
  public native void c2JavaArraySort(int[] array) ;

  static {
    System.loadLibrary("native-lib");
  }
}

```

```cpp
#include <jni.h>
#include <string>
#include <stdlib.h>
#include <android/log.h>
//logcat日志
#define  LOG_TAG    "NDK"
#define  LOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)
#define  LOGE(...)  __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)


#ifdef  __cplusplus
extern "C" {
#endif

//比较
int compare(const void *a, const void *b);

// 对数组进行排序
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaArraySort(JNIEnv *env, jobject instance,
                                                     jintArray array_) {
    jint *array = env->GetIntArrayElements(array_, NULL);
    // 数组长度
    jsize array_size = env->GetArrayLength(array_);
    // 快速排序函数
    qsort(array, (size_t) array_size, sizeof(jint), compare);
    // 释放引用,因为数组和对象在java中都是引用,都会在堆内存中开辟一块空间,但我们使用完对象之后
    // 需要将引用释放掉,不然会很耗内存,在一定程度上可能会造成内存溢出 。
    // JNI_ABORT, Java数组不进行更新，但是释放C/C++数组
    // JNI_COMMIT，Java数组进行更新，不释放C/C++数组（函数执行完，数组还是会释放）
    //// 0，Java数组进行更新，释放C/C++数组
    env->ReleaseIntArrayElements(array_, array, 0);
}

int compare(const void *a, const void *b) {
    LOGI("a = %d,b = %d",*((int*)a),*((int*)b));
    return *((int*)a) - *((int*)b);
}

#ifdef  __cplusplus
}
#endif
```

# 引用释放

只要是Java对象,在C中都需要释放,如String类型引用：

```
// String类型引用释放
void (JNICALL *ReleaseStringUTFChars)
      (JNIEnv *env, jstring str, const char* chars);
```

在C中创建的对象引用也需要进行引用释放.

>创建一个数组对象,并将引用传递给了Java层,将引用交给了Java之后,C就需要释放这个引用,不然会一直持有,GC也不会回收这个对象 

```
/*返回int类型的数组*/
JNIEXPORT jintArray JNICALL Java_com_zeno_jni_HelloJNI_getIntArray
(JNIEnv *env, jobject jobj,jint len) {

    // 创建一个jint类型的数组
    jintArray jArray = (*env)->NewIntArray(env, len);

    // 得到数组首个元素指针
    jint* arrayElements = (*env)->GetIntArrayElements(env, jArray, NULL);

    // 指针运算
    int i = 0;
    for (; i < len; i++)
    {
        arrayElements[i] = i;
    }

    // 同步
    (*env)->ReleaseIntArrayElements(env, jArray, arrayElements, JNI_COMMIT);

    return jArray;
}

java code
// 在C中生存数组 ， 返回到Java中
private native int[] getIntArray(int len) ;

int[] intArray = jni.getIntArray(20);
    for (int i = 0; i < intArray.length; i++) {
        System.out.println("int array === "+intArray[i]);
}
```

# JNI引用分类

## 局部引用

局部引用在本地方法调用的时间内有效，本地方法调用结束后会自动释放，每个局部引用都消耗一定的JVM资源，所以我们必须确保本地方法中不能过多的分配局部引用，尽管局部引用能够在方法调用后自动释放，但是过多的局部引用仍然可能导致虚拟机内存溢出。

```cpp
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_localRef
(JNIEnv *env, jobject jobj) {

    // 找到类
    jclass dateClass = (*env)->FindClass(env, "java/util/Date");

    // 得到构造方法ID
    jmethodID dateConstructorId = (*env)->GetMethodID(env, dateClass, "<init>", "()V");

    // 创建Date对象
    jobject dateObject = (*env)->NewObject(env, dateClass, dateConstructorId);

    // 创建一个局部引用
    jobject dateLocalRef = (*env)->NewLocalRef(env, dateObject);

    // 省略N行代码

    // 不再使用对象,则通知GC回收对象，手动释放对象
    (*env)->DeleteLocalRef(env, dateLocalRef);
    // 因为dateObject也是局部对象，可以直接回收dateObject对象
    //(*env)->DeleteLocalRef(env, dateObject);

}
```

## 全局引用

```cpp
jstring globalStr;

/*创建全局引用*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_createGlobalRef
(JNIEnv *env, jobject jobj) {

    jstring jStr = (*env)->NewStringUTF(env, "I want your love !");

    // 创建一个全局引用
    globalStr = (*env)->NewGlobalRef(env, jStr);
}

/*使用全局引用*/
JNIEXPORT jstring JNICALL Java_com_zeno_jni_HelloJNI_useGlobalRef
(JNIEnv *env, jobject jobj) {

    return globalStr;

}

/*释放全局引用*/
JNIEXPORT void JNICALL Java_com_zeno_jni_HelloJNI_deleteGlobalRef
(JNIEnv *env, jobject jobj) {

    // 释放全局引用
    (*env)->DeleteGlobalRef(env, globalStr);

}
```

## 弱全局引用

节省内存，在内存不足时可以释放所引用的对象
可以引用一个不常用的对象，如果为NULL，临时创建

```cpp
//创建
jweak NewWeakGlobalRef(JNIEnv *env, jobject obj);
//销毁
void DeleteWeakGlobalRef(JNIEnv *env, jweak obj);
```

[JNI开发系列⑤对象引用的处理](http://www.jianshu.com/p/09469f692a4b)


