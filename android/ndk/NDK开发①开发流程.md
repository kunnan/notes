**NDK开发①开发流程**

[TOC]

# Java编写native方法

新建JniUtil类如下:

```java
	public class JniUtil {
		
		/**
		 * 回调接口
		 * @author jimi098
		 *
		 */
		public interface IRecFrameListener {
			
			public void onRecFrameListener(String result);
		}
		
		private IRecFrameListener mListener;
		
		
		static {
			
			System.loadLibrary("test");
		}
		
		
		public void setOnRecFrameListener(IRecFrameListener listener) {
			
			mListener = listener;
		}
		
		/**
		 * 供jni调用的java方法
		 * @param result
		 */
		public void onRecFrame(int result) {
			mListener.onRecFrameListener(String.valueOf(result));
		}
		
		//java本地方法
		public native int init();
		public native int add(int a,int b);
		public native String getString(String input);
	
	}
```

# 生成.h头文件

编译JniUtil类，生成JniUtil.class文件，然后在cmd中切换目录至android工程的src文件夹路径，运行`javah -jni com.example.demo.JniUtil`命令即可生成.h头文件

```java
	#include <jni.h>
	#include <android/log.h>
	
	//logcat日志
	#define  LOG_TAG    "Test"
	#define  LOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)
	#define  LOGE(...)  __android_log_print(ANDROID_LOG_ERROR,LOG_TAG,__VA_ARGS__)
	/* Header for class com_example_demo_JniUtil */
	
	#ifndef _Included_com_example_demo_JniUtil
	#define _Included_com_example_demo_JniUtil
	#ifdef __cplusplus
	extern "C" {
	#endif
	/*
	 * Class:     com_example_demo_JniUtil
	 * Method:    init
	 * Signature: ()I
	 */
	JNIEXPORT jint JNICALL Java_com_example_demo_JniUtil_init
	  (JNIEnv *, jobject);
	
	/*
	 * Class:     com_example_demo_JniUtil
	 * Method:    add
	 * Signature: (II)I
	 */
	JNIEXPORT jint JNICALL Java_com_example_demo_JniUtil_add
	  (JNIEnv *, jobject, jint, jint);
	
	/*
	 * Class:     com_example_demo_JniUtil
	 * Method:    getString
	 * Signature: (Ljava/lang/String;)Ljava/lang/String;
	 */
	JNIEXPORT jstring JNICALL Java_com_example_demo_JniUtil_getString
	  (JNIEnv *, jobject, jstring);
	
	#ifdef __cplusplus
	}
	#endif
	#endif
```

# 实现jni

新建jni文件夹，实现本地java方法

```java
	#include<jniutil.h>
	
	jobject mObject;
	
	
	JNIEXPORT jint JNICALL Java_com_example_demo_JniUtil_init(JNIEnv *env,
			jobject obj) {
	
		//获取全局对象
		mObject = (*env)->NewGlobalRef(env, obj);
	
		jmethodID methodID;
		jclass mClass;
	
		//查找指定名称类
		mClass = (*env)->GetObjectClass(env, mObject);;
	
		//获取方法ID
		methodID = (*env)->GetMethodID(env,mClass,"onRecFrame","(I)V");
	
		//调用上层java方法
		(*env)->CallVoidMethod(env,obj,methodID,89);
	
		return 5;
	}
	
	/*
	 * Class:     com_example_demo_JniUtil
	 * Method:    add
	 * Signature: (II)I
	 */
	JNIEXPORT jint JNICALL Java_com_example_demo_JniUtil_add(JNIEnv *env,
			jobject obj, jint x, jint y) {
	
		return x + y;
	
	}
	
	/*
	 * Class:     com_example_demo_JniUtil
	 * Method:    getString
	 * Signature: (Ljava/lang/String;)Ljava/lang/String;
	 */
	JNIEXPORT jstring JNICALL Java_com_example_demo_JniUtil_getString(JNIEnv *env,
			jobject obj, jstring result) {
	
		return result;
	
	}
```

# 编写mk文件

## Android.mk

指定编译规则

```
	LOCAL_PATH := $(call my-dir)
	
	include $(CLEAR_VARS)
	
	LOCAL_MODULE    := test
	LOCAL_SRC_FILES := test.c
	
	LOCAL_LDLIBS    := -llog
	include $(BUILD_SHARED_LIBRARY)
```

## Application.mk

指定编译平台

```
	APP_ABI := armeabi armeabi-v7a
	APP_PLATFORM := android-22
	APP_STL:=gnustl_static
	APP_CPPFLAGS:=-frtti -fexceptions 
```

# 配置NDK执行编译

右击工程--》Properties--》Builders--》New--》选择Program--》Main--》Location：选择ndk-build.cmd命令路径--》Working Directory：选择工程路径--》Refresh--》Build Options，配置完成后即可自动编译，不需要安装cygwin linux模拟环境

# 应用jni

```
	public class MainActivity extends Activity implements OnClickListener, IRecFrameListener {
		
		private Button mButton;
		
		private JniUtil mJniUtil;
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        
	        mButton = (Button) findViewById(R.id.btn_click);
	        mButton.setOnClickListener(this);
	        mJniUtil = new JniUtil();
	        mJniUtil.setOnRecFrameListener(this);
	        
	    }
	
		@Override
		public void onClick(View v) {
			// TODO Auto-generated method stub
			System.out.println("init: " + mJniUtil.init());
			System.out.println("sum: "+ mJniUtil.add(1, 2));
			System.out.println("getString: " + mJniUtil.getString("wert"));
		}
	
	
		/**
		 * 回调，由jni方法返回数据
		 */
		@Override
		public void onRecFrameListener(String result) {
			// TODO Auto-generated method stub
			System.out.println(result);
			
		}
	}
```

# Android Studio 2.2 开发NDK

[NDK开发基础①使用Android Studio编写NDK](http://www.jianshu.com/p/f1b8b97d2ef8)

## Gradle配置NDK

```
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.1"
    defaultConfig {
        applicationId "com.michael.lplayer"
        minSdkVersion 15
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags ""
                arguments '-DANDROID_TOOLCHAIN=clang'
                abiFilters "armeabi-v7a"
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
    //加载外部so库
    sourceSets.main {
        jniLibs.srcDirs = ['libs']
        jni.srcDirs = []
    }
}
```

## CMakeLists.txt 配置

```
# Sets the minimum version of CMake required to build the native
# library. You should either keep the default value or only pass a
# value of 3.4.0 or lower.

cmake_minimum_required(VERSION 3.4.1)

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

set(distribution_DIR ${CMAKE_SOURCE_DIR}/../../../../libs)
include_directories(libs/include)

add_library( avutil-55
             SHARED
             IMPORTED )
set_target_properties( avutil-55
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavutil-55.so )

add_library( swresample-2
             SHARED
             IMPORTED )
set_target_properties( swresample-2
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libswresample-2.so )
add_library( avcodec-57
             SHARED
             IMPORTED )
set_target_properties( avcodec-57
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavcodec-57.so )
add_library( avfilter-6
             SHARED
             IMPORTED)
set_target_properties( avfilter-6
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavfilter-6.so )
add_library( swscale-4
             SHARED
             IMPORTED)
set_target_properties( swscale-4
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libswscale-4.so )
add_library( avformat-57
             SHARED
             IMPORTED)
set_target_properties( avformat-57
                       PROPERTIES IMPORTED_LOCATION
                       ../../../../libs/armeabi-v7a/libavformat-57.so )

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11")

add_library( ffmpeg
             SHARED
             src/main/cpp/ffmpeg_decode.c )

#target_include_directories(ffmpeg PRIVATE libs/include)

target_link_libraries( ffmpeg avutil-55 swresample-2  avcodec-57 avfilter-6 swscale-4 avformat-57
                       ${log-lib} )

```


