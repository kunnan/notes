**JNI之③C/C++调用Java字段与方法**

[TOC]

# native函数定义

```java
package com.michael.ndk.write;

/**
 * C访问Java字段和方法
 * Created by liuguoquan on 2017/1/3.
 */

public class WriteJava {
  //Java成员变量
  public String name = "liu";
  private int age = 20;
  public static String sex = "男";

  public void setName(String name) {
    this.name = name;
  }

  public static String getName() {
    return "刘涤生";
  }
  /**
   * C/C++修改java String 类型字段本地方法
   */
  public native void c2JavaStringField();
  public native void c2JavaIntField();
  public native void c2JavaStringStaticField();
  /**
   *  C/C++ 访问Java方法
   */
  public native void c2JavaMethod();
  public native void c2JavaStaticMethod();

  /**
   * 静态native方法访问字段
   */
  public static native void native2JavaStringField();

  static {
    System.loadLibrary("native-lib");
  }
}
```

# C/C++访问Java字段

1. 获取jclass对象
2. 获取字段ID
3. 设置字段的值

```cpp
/*C语言访问java String类型字段*/
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaStringField(JNIEnv *env, jobject instance) {
    // 1.获取jclass
    jclass clz = env->GetObjectClass(instance);
    // 2.获取字段ID
    jfieldID fieldId = env->GetFieldID(clz,"name","Ljava/lang/String;");
    //3.获取字段的值
    jstring str = (jstring)env->GetObjectField(instance,fieldId);
    //4.将jstring类型转换成字符指针
    const char *cstr = env->GetStringUTFChars(str,JNI_FALSE);
    LOGI("%s",cstr);
    //释放内存
    env->ReleaseStringUTFChars(str,cstr);
    //拼接字符
    const char *newstr = "lee";
    //5.创建新jstring
    jstring new_str = env->NewStringUTF(newstr);
    //6.将jstring类型的变量值 设置到java字段中
    env->SetObjectField(instance,fieldId,new_str);
}

/**
 * C/C++访问修改java int字段
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaIntField(JNIEnv *env, jobject instance) {

    // jclass
    jclass clz = env->GetObjectClass(instance);
    // 字段id
    jfieldID fieldId = env->GetFieldID(clz,"age","I");
    //得到字段值
    jint age = env->GetIntField(instance,fieldId);
    LOGI("age = %d",age);
    env->SetIntField(instance,fieldId,age + 1);
}
```

# C/C++访问Java方法

1. 获取jclass对象
2. 获取方法ID
3. 调用方法

```cpp
/**
 * C/C++ 访问java方法
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaMethod(JNIEnv *env, jobject instance) {

    // 获取jclass
    jclass clz = env->GetObjectClass(instance);
    // 获取methodId
    //函数签名 括号内表示参数 括号外表示返回值
    jmethodID methodId = env->GetMethodID(clz,"setName","(Ljava/lang/String;)V");
    // 调用方法
    jstring str = env->NewStringUTF("zhang");
    env->CallVoidMethod(instance,methodId,str);
}
```

# C/C++访问Java静态字段

1. 获取jclass对象
2. 获取静态字段ID
3. 设置静态字段的值

```cpp
/**
 * c/c++ 访问java静态字段
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaStringStaticField(JNIEnv *env, jobject instance) {
    // 获取jclass对象
    jclass clz = env->GetObjectClass(instance);
    jfieldID fieldId = env->GetStaticFieldID(clz,"sex","Ljava/lang/String;");
    //获取字段的值
    jstring jstr = (jstring)env->GetStaticObjectField(clz,fieldId);

    const char *sex = env->GetStringUTFChars(jstr,JNI_FALSE);
    LOGI("sex = %s\n",sex);
    jstring newStr = env->NewStringUTF("女");
    //修改字段
    env->SetStaticObjectField(clz,fieldId,newStr);
}
```

# C/C++访问Java静态方法

1. 获取jclass对象
2. 获取静态方法ID
3. 调用静态方法

```cpp
/**
 * c/c++访问静态方法
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaStaticMethod(JNIEnv *env, jobject instance) {

    jclass clz = env->GetObjectClass(instance);
    //获取静态方法id
    jmethodID methodId = env->GetStaticMethodID(clz,"getName","()Ljava/lang/String;");
    jstring obj = (jstring)env->CallStaticObjectMethod(clz,methodId);
    const char* value = env->GetStringUTFChars(obj,JNI_FALSE);
    LOGI("name = %s\n",value);
}
```
# 静态native方法访问Java静态字段

1. 获取静态字段id
2. 设置静态字段值

```cpp
/**
 * 静态native方法 第二个参数是jclass而不是jobject类型
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_native2JavaStringField(JNIEnv *env, jclass clz) {

    jfieldID fieldID = env->GetStaticFieldID(clz,"sex","Ljava/lang/String;");
    jstring jstr = (jstring) env->GetStaticObjectField(clz, fieldID);
    LOGI("sex = %s\n",env->GetStringUTFChars(jstr,JNI_FALSE));
    jstring newStr = env->NewStringUTF("女");
    //修改静态常量字段
    env->SetStaticObjectField(clz,fieldID,newStr);
}
```

# 示例代码

## Java代码

```java
package com.michael.ndk.write;

/**
 * C访问Java字段和方法
 * Created by liuguoquan on 2017/1/3.
 */

public class WriteJava {
  //Java成员变量
  public String name = "liu";
  private int age = 20;
  public static String sex = "男";

  public void setName(String name) {
    this.name = name;
  }

  public static String getName() {
    return "刘涤生";
  }
  /**
   * C/C++修改java String 类型字段本地方法
   */
  public native void c2JavaStringField();
  public native void c2JavaIntField();
  public native void c2JavaStringStaticField();
  /**
   *  C/C++ 访问Java方法
   */
  public native void c2JavaMethod();
  public native void c2JavaStaticMethod();

  /**
   * 静态native方法访问字段
   */
  public static native void native2JavaStringField();

  static {
    System.loadLibrary("native-lib");
  }
}
```

## C/C++代码

```cpp
#include <jni.h>
#include <string>

#include <android/log.h>
//logcat日志
#define  LOG_TAG    "NDK"
#define  LOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)
#define  LOGE(...)  __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)
/* Header for class com_example_demo_JniUtil */

extern "C"
/*C语言访问java String类型字段*/
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaStringField(JNIEnv *env, jobject instance) {
    // 1.获取jclass
    jclass clz = env->GetObjectClass(instance);
    // 2.获取字段ID
    jfieldID fieldId = env->GetFieldID(clz,"name","Ljava/lang/String;");
    //3.获取字段的值
    jstring str = (jstring)env->GetObjectField(instance,fieldId);
    //4.将jstring类型转换成字符指针
    const char *cstr = env->GetStringUTFChars(str,JNI_FALSE);
    LOGI("%s",cstr);
    //拼接字符
    const char *newstr = "lee";
    //5.创建新jstring
    jstring new_str = env->NewStringUTF(newstr);
    //6.将jstring类型的变量值 设置到java字段中
    env->SetObjectField(instance,fieldId,new_str);
}

extern "C"
/**
 * C/C++访问修改java int字段
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaIntField(JNIEnv *env, jobject instance) {

    // jclass
    jclass clz = env->GetObjectClass(instance);
    // 字段id
    jfieldID fieldId = env->GetFieldID(clz,"age","I");
    //得到字段值
    jint age = env->GetIntField(instance,fieldId);
    LOGI("age = %d",age);
    env->SetIntField(instance,fieldId,age + 1);
}

extern "C"
/**
 * C/C++ 访问java方法
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaMethod(JNIEnv *env, jobject instance) {

    // 获取jclass
    jclass clz = env->GetObjectClass(instance);
    // 获取methodId
    //函数签名 括号内表示参数 括号外表示返回值
    jmethodID methodId = env->GetMethodID(clz,"setName","(Ljava/lang/String;)V");
    // 调用方法
    jstring str = env->NewStringUTF("zhang");
    env->CallVoidMethod(instance,methodId,str);
}

extern "C"
/**
 * c/c++ 访问java静态字段
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaStringStaticField(JNIEnv *env, jobject instance) {
    // 获取jclass对象
    jclass clz = env->GetObjectClass(instance);
    jfieldID fieldId = env->GetStaticFieldID(clz,"sex","Ljava/lang/String;");
    //获取字段的值
    jstring jstr = (jstring)env->GetStaticObjectField(clz,fieldId);

    const char *sex = env->GetStringUTFChars(jstr,JNI_FALSE);
    LOGI("sex = %s\n",sex);
    jstring newStr = env->NewStringUTF("女");
    //修改字段
    env->SetStaticObjectField(clz,fieldId,newStr);
}

extern "C"
/**
 * c/c++访问静态方法
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_c2JavaStaticMethod(JNIEnv *env, jobject instance) {

    jclass clz = env->GetObjectClass(instance);
    //获取静态方法id
    jmethodID methodId = env->GetStaticMethodID(clz,"getName","()Ljava/lang/String;");
    jstring obj = (jstring)env->CallStaticObjectMethod(clz,methodId);
    const char* value = env->GetStringUTFChars(obj,JNI_FALSE);
    LOGI("name = %s\n",value);
}

extern "C"
/**
 * 静态native方法 第二个参数书jclass而不是jobject类型
 */
JNIEXPORT void JNICALL
Java_com_michael_ndk_write_WriteJava_native2JavaStringField(JNIEnv *env, jclass clz) {

    jfieldID fieldID = env->GetStaticFieldID(clz,"sex","Ljava/lang/String;");
    jstring jstr = (jstring) env->GetStaticObjectField(clz, fieldID);
    LOGI("sex = %s\n",env->GetStringUTFChars(jstr,JNI_FALSE));
    jstring newStr = env->NewStringUTF("女");
    //修改静态常量字段
    env->SetStaticObjectField(clz,fieldID,newStr);
}
```

# Q&A

# 为什么要得到jclass呢 ？

因为 ，我们要获取字段ID，在JNI中，获取java字段与方法都需要签名。而签名是在类加载的时候完成，所以在获取字段ID的时候需要传入jclass。

# 为什么传入了字段名称，还需要签名呢 ？

因为java支持重载 ， 一个方法名称可以有多个不同实现 ， 根据传入的参数不同 ，所以C语言调用函数为了区分不同的方法， 而对每个方法做了签名 ， 而字段则可用来标识类型。

>在.class的文件目录下 ，使用`javap -s -p className`   就可以列举出 ， 所有的字段与方法签名

[JNI开发系列③C语言调用Java字段与方法](http://www.jianshu.com/p/9cc6e3f0ead7)
[JNI开发系列②.h头文件分析](http://www.jianshu.com/p/cba836f6a08c)
[JNI开发系列①JNI概念及开发流程](http://www.jianshu.com/p/68bca86a84ce)


