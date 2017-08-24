---
title: Mac编译Android源码
date: 2017-08-14 21:01:06
tags: Android系统
categories: [Android,Framwork]
---

# 编译Android源码

本机的系统环境如下：

* macOs Sierra 10.12.5
* JDK 1.8.0_131
* Xcode 8.3.3
* 本地源码路径 /Volumes/android/aosp
* 源码版本 android-7.1.2_r1**

## 配置系统环境

* Mac创建大小写分区映像 120G
* 安装JDK
* 安装 xcode 命令行工具
* 安装macports，并配置到系统环境 `export PATH=/opt/local/bin:$PATH`
* 通过 macports 安装 make , git 以及 GPG

```
$ POSIXLY_CORRECT=1 sudo port install gmake libsdl git gnupg
```
<!-- more -->

## 下载源码

* 安装Repo `curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo`
* 将`~/bin`目录添加到PATH路径
* `chmod a+x ~/bin/repo`打开可执行权限
* 在创建的源码目录初始化repo仓库`$ repo init -u https://android.googlesource.com/platform/manifest -b android-7.1.2_r1`
* 同步源码到本地`repo sync`，最好是在晚上下载

## 编译源码

```
//清空 out 输出目录中的所有文件
make clobber
// 初始化我们的编译环境
source build/envsetup.sh
//选择我们需要编译的目标
lunch

//编译源码 指定线程数量
make -j8

//模拟器启动
emulator
```

## 花费时间

* 下载源码时间大概10个小时左右
* 编译源码时间大约2小时左右 8线程编译

## could not find jdk tools.jar

* 现象

```
$ make -j16
build/core/config.mk:600: *** Error: could not find jdk tools.jar at /System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/../lib/tools.jar, please check if your JDK was installed correctly.  Stop.
```

* 解决方案

主要是因为环境变量没有设置ANDROID_JAVA_HOME 导致的，添加对应的环境变量即可，输入下面的命令：

```
export ANDROID_JAVA_HOME=$(/usr/libexec/java_home -v 1.8)
```

## ‘syscall’ is deprecated

* 错误现象

```
system/core/libcutils/threads.c:38:10: error: 'syscall' is deprecated: first deprecated in OS X 10.12 - syscall(2) is unsupported; please switch to a supported interface. For SYS_kdebug_trace use kdebug_signpost(). [-Werror,-Wdeprecated-declarations]
return syscall(SYS_thread_selfid);
```

* 解决方案

这是因为最新的macOS的调整引起的，个人下载的最新版的源码也没有解决这个问题，最终采用的方法是使用更早版本的macOS的SDK，下载地址参考前面准备工作中的说明。解压(也会用时比较久)zip包以后将MacOSX10.11.sdk拷贝到/Applications/XCode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs。为了避免下次升级的时候再被删除，建议拷贝到自定义目录，然后建立软链接。例如我放在~/lib目录下，再给它创建一个软链接：

```
$ sudo ln -s ~/lib/MacOSX10.11.sdk /Applications/XCode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk
```

然后确保AOSP源码下build/core/combo/mac_version.mk文件中
`mac_sdk_versions_supported := 10.9 10.10 10.11`
后面不要写10.12。

# 导入源码到Android Studio

## 生成 IDE 相关的项目文件

```
make idegen && development/tools/idegen/idegen.sh
```
命令必须在bash下运行，zsh下不能运行，bash 和 zsh 切换命令如下

```
切换bash
chsh -s /bin/bash
切换zsh
chsh -s /bin/zsh
```
在源码根目录下生成android.ipr和android.iml。

## 导入源码

启动Android Studio，然后选择打开一个已存在的Android Studio工程，选择源码根目录的android.ipr，经过漫长(我首次导入用了20分钟)的加载过程以后，Android 源码就已经成功的加载到了Android Studio中。

# 参考资料

* [macOS（Sierra 10.12）上Android源码（AOSP）的下载、编译与导入到Android Studio](http://blog.bihe0832.com/macOS-AOSP.html)

* [Android FrameWork学习（一）Android 7.0系统源码下载\编译](http://www.jianshu.com/p/6af0bb7c1e70)

* [Android FrameWork学习（二）Android系统源码调试](http://www.jianshu.com/p/4ab864caefb2)

* [Mac 10.12 编译 Android 源码](http://www.jianshu.com/p/1513fc9e1a74)


