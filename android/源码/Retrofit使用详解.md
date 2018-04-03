**Retrofit使用详解**

[TOC]

# 定义

Retrofit 是一个 Restful 设计风格的 HTTP 网络请求框架的封装，网络请求的工作本质上是 OkHttp 完成，而 Retrofit 仅负责网络请求接口的封装。

![](https://upload-images.jianshu.io/upload_images/944365-b5194f1d16673589.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

# 地址

[retrofit 2.3.0](https://github.com/square/retrofit)

# 功能

* 基于 OkHttp & 遵循 Restful API 设计风格
* 通过注解配置网络请求参数
* 支持同步 & 异步网络请求
* 支持多种数据格式的解析 & 序列化格式（json、xml、protobuf）
* 支持 RxJava

# 网络请求库对比

![](https://upload-images.jianshu.io/upload_images/944365-58819416dfd2767a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

# 使用介绍

Retrofit 的使用步骤：

1. 添加Retrofit库的依赖
2. 创建 接收服务器返回数据 的类
3. 创建 用于描述网络请求 的接口
4. 创建 Retrofit 实例
5. 创建 网络请求接口实例 并 配置网络请求参数
6. 发送网络请求（异步 / 同步）请求数据

## 1. 添加Retrofit库的依赖

build.gradle

```
compile 'com.squareup.retrofit2:retrofit:2.3.0'
```

AndroidManifest.xml 添加网络权限

```
<uses-permission android:name="android.permission.INTERNET"/>
```

ProGuard

```
# Retain generic type information for use by reflection by converters and adapters.
-keepattributes Signature
# Retain service method parameters.
-keepclassmembernames,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}
# Ignore annotation used for build tooling.
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
```

## 2. 创建 接收服务器返回数据 的类

```java
public class ApiResult<T> {
    public int status;
    public String msg;
    public T data;
}
```

## 3. 创建 用于描述网络请求 的接口

Retrofit将 Http请求抽象成Java接口：采用注解描述和配置网络请求参数

```java
public interface LoginApi {

  /**
   * 登录
   *
   */
  @POST("auth/login") Observable<ApiResult<User>> login(@Body LoginBody body);
}
```

### 注解类型

![](https://upload-images.jianshu.io/upload_images/944365-ee747d1e331ed5a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### 注解说明

网络请求方法

![](https://upload-images.jianshu.io/upload_images/944365-e97379b8e0942459.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

标记

![](https://upload-images.jianshu.io/upload_images/944365-a6f1fc997c23a2e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

* @FormUrlEncoded 表示发送form-encoded的数据，每个键值对需要用@Filed来注解键名，随后的对象需要提供值。
* @Multipart 表示发送form-encoded的数据（适用于 有文件 上传的场景），每个键值对需要用@Part来注解键名，随后的对象需要提供值。

网络请求参数

![](https://upload-images.jianshu.io/upload_images/944365-c547f2344eef630b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

## 4. 创建 Retrofit 实例

```java
Retrofit retrofit = new Retrofit.Builder().baseUrl(Constants.HOST) // 设置网络请求的Url地址
  .addConverterFactory(GsonConverterFactory.create()) //设置数据解析器
  .addCallAdapterFactory(RxJava2CallAdapterFactory.create()) //设置请求适配器
  .client(HttpClient.getDefaultHttpClient()) //设置网络请求框架OkHttp
  .build();
```

### 数据解析器


| 数据解析器 | Gradle依赖 |
| --- | --- |
| Gson | com.squareup.retrofit2:converter-gson:2.3.0 |
| Jackson | com.squareup.retrofit2:converter-jackson:2.3.0 |
| Simple XML | com.squareup.retrofit2:converter-simplexml:2.3.0 |
| Protobuf | com.squareup.retrofit2:converter-protobuf:2.3.0 |
| Moshi | com.squareup.retrofit2:converter-moshi:2.3.0 |
| Wire | com.squareup.retrofit2:converter-wire:2.3.0 |
| Scalars | com.squareup.retrofit2:converter-scalars:2.3.0 |

### 网络请求适配器

使用的是 Android 默认的 CallAdapter，则不需要添加网络请求适配器的依赖，否则则需要按照需求进行添加 Retrofit 提供的 CallAdapter。

| 网络请求适配器 | Gradle依赖 |
| --- | --- |
| guava | com.squareup.retrofit2:adapter-guava:2.3.0 |
| Java8 | com.squareup.retrofit2:adapter-java8:2.3.0 |
| rxjava2 | com.squareup.retrofit2:adapter-rxjava2:2.3.0 |

## 5. 创建 网络请求接口实例

```java
// 创建 网络请求接口 的实例
LoginApi request = retrofit.create(LoginApi.class)
//对 发送请求 进行封装
Call<Reception> call = request.getCall();
```

## 6. 发送网络请求（异步 / 同步）

```java
//发送网络请求(异步)
        call.enqueue(new Callback<Translation>() {
            //请求成功时回调
            @Override
            public void onResponse(Call<Translation> call, Response<Translation> response) {
                //请求处理,输出结果
                response.body().show();
            }

            //请求失败时候的回调
            @Override
            public void onFailure(Call<Translation> call, Throwable throwable) {
                System.out.println("连接失败");
            }
        });

// 发送网络请求（同步）
Response<Reception> response = call.execute();
```

## 7. 处理返回数据

```java
//发送网络请求(异步)
        call.enqueue(new Callback<Translation>() {
            //请求成功时回调
            @Override
            public void onResponse(Call<Translation> call, Response<Translation> response) {
                // 对返回数据进行处理
                response.body().show();
            }

            //请求失败时候的回调
            @Override
            public void onFailure(Call<Translation> call, Throwable throwable) {
                System.out.println("连接失败");
            }
        });

// 发送网络请求（同步）
  Response<Reception> response = call.execute();
  // 对返回数据进行处理
  response.body().show();
```

# Retrofit 的拓展使用

```java
<-- 主要在创建Retrofit对象中设置 -->
Retrofit retrofit = new Retrofit.Builder()
  .baseUrl(""http://fanyi.youdao.com/"")
  .addConverterFactory(ProtoConverterFactory.create()) // 支持Prototocobuff解析
  .addConverterFactory(GsonConverterFactory.create()) // 支持Gson解析
  .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 支持RxJava
  .build();
```

# 参考文章

* [Android Retrofit 2.0 的详细 使用攻略（含实例讲解)](https://www.jianshu.com/p/a3e162261ab6)
* [Android：Retrofit 与 RxJava联合使用大合集](http://blog.csdn.net/carson_ho/article/details/79125101)
* [Carson_Ho的博客](http://blog.csdn.net/carson_ho)


