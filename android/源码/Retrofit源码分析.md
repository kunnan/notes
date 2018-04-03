**Retrofit源码分析**

[TOC]

# 1 Retrofit 的本质流程

一般从网络通信过程如下图：

![](http://upload-images.jianshu.io/upload_images/944365-830bc90df2e1d1fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实Retrofit的本质和上面是一样的套路，只是Retrofit通过使用大量的设计模式进行功能模块的解耦，使得上面的过程进行得更加简单 & 流畅。

![](http://upload-images.jianshu.io/upload_images/944365-72f373fbbb960b69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Retrofit 具体过程解释如下：

1. 通过解析 网络请求接口的注解 配置 网络请求参数
2. 通过 动态代理 生成 网络请求对象
3. 通过 网络请求适配器 将 网络请求对象 进行平台适配，平台包括：Android、Rxjava、Guava和java8
4. 通过 网络请求执行器 发送网络请求
5. 通过 数据转换器 解析服务器返回的数据
6. 通过 回调执行器 切换线程（子线程 ->>主线程）
7. 用户在主线程处理返回结果

![](http://upload-images.jianshu.io/upload_images/944365-5f4b1f44be83e554.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2 源码分析

Retrofit的使用步骤：

1. 创建 Retrofit 实例
2. 创建 网络请求接口实例 并 配置网络请求参数 
3. 发送网络请求
4. 处理服务器返回的数据

## 2.1 创建 Retrofit 实例

### 2.1.1 使用步骤

```java
 Retrofit retrofit = new Retrofit.Builder()
                                 .baseUrl("http://fanyi.youdao.com/")
                                 .addConverterFactory(GsonConverterFactory.create())
                                 .build();
```

### 2.1.2 源码分析

Retrofit 实例是使用建造者模式通过 Builder 类创建的。

接下来，分5个步骤对创建 Retrofit 实例进行分析：

![](http://upload-images.jianshu.io/upload_images/944365-3b9c7000667ddf89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 步骤1

![](http://upload-images.jianshu.io/upload_images/944365-566343b54bb3b5f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Retrofit 类：

```java
public final class Retrofit {

  //网络请求配置对象
  //存储网络请求相关的配置，如网络请求的方法、数据转换器、网络请求适配器、网络请求工厂、url地址等。
  private final Map<Method, ServiceMethod<?, ?>> serviceMethodCache = new ConcurrentHashMap<>();

  //网络请求工厂，Retrofit 默认是 okhttp
  final okhttp3.Call.Factory callFactory;
  
  //网络请求的url
  final HttpUrl baseUrl;
  //数据转换适配器集合
  final List<Converter.Factory> converterFactories;
  //网络请求适配器集合
  final List<CallAdapter.Factory> adapterFactories;
  //回调方法执行器
  final @Nullable Executor callbackExecutor;
  //是否提前对业务接口中的注解进行验证转换的标志位
  final boolean validateEagerly;

  Retrofit(okhttp3.Call.Factory callFactory, HttpUrl baseUrl,
      List<Converter.Factory> converterFactories, List<CallAdapter.Factory> adapterFactories,
      @Nullable Executor callbackExecutor, boolean validateEagerly) {
    this.callFactory = callFactory;
    this.baseUrl = baseUrl;
    this.converterFactories = unmodifiableList(converterFactories); // Defensive copy at call site.
    this.adapterFactories = unmodifiableList(adapterFactories); // Defensive copy at call site.
    this.callbackExecutor = callbackExecutor;
    this.validateEagerly = validateEagerly;
  }
  
}
```

**CallAdapter 详细介绍**

* 定义：网络请求执行器（Call）的适配器

>1. Call 在 Retrofit 里默认是 OkHttpCall
>2. 在 Retrofit 中提供了四种 CallAdapterFactory：ExecutorCallAdapterFactory（默认）、GuavaCallAdapterFactory、Java8CallAdapterFactory、RxJavaCallAdapterFactory。

* 作用：将默认的网络请求执行器（OkHttpCall）转换成适合被不同平台来调用的网络请求执行器形式。

#### 步骤2

![](http://upload-images.jianshu.io/upload_images/944365-21940f0bc0d92d8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Builder 类：

```java
  public static final class Builder {
    private final Platform platform;
    private @Nullable okhttp3.Call.Factory callFactory;
    private HttpUrl baseUrl;
    private final List<Converter.Factory> converterFactories = new ArrayList<>();
    private final List<CallAdapter.Factory> adapterFactories = new ArrayList<>();
    private @Nullable Executor callbackExecutor;
    private boolean validateEagerly;

    //1.Builder的无参构造函数
    public Builder() {
      this(Platform.get());
    }

    Builder(Retrofit retrofit) {
      platform = Platform.get();
      callFactory = retrofit.callFactory;
      baseUrl = retrofit.baseUrl;
      converterFactories.addAll(retrofit.converterFactories);
      adapterFactories.addAll(retrofit.adapterFactories);
      // Remove the default, platform-aware call adapter added by build().
      adapterFactories.remove(adapterFactories.size() - 1);
      callbackExecutor = retrofit.callbackExecutor;
      validateEagerly = retrofit.validateEagerly;
    }
    
    ···
 }
 
 //2.Platform类
 class Platform {
  private static final Platform PLATFORM = findPlatform();

  static Platform get() {
    return PLATFORM;
  }

//3. 获取平台类
  private static Platform findPlatform() {
    try {
      Class.forName("android.os.Build");
      if (Build.VERSION.SDK_INT != 0) {
        return new Android();
      }
    } catch (ClassNotFoundException ignored) {
    }
    try {
      Class.forName("java.util.Optional");
      return new Java8();
    } catch (ClassNotFoundException ignored) {
    }
    return new Platform();
  }
}

 //4. 用于接收服务器数据后进行线程切换在主线程显示结果
  static class Android extends Platform {
    @Override public Executor defaultCallbackExecutor() {
     //返回默认的回调方法执行器
     //作用：子线程切换主线程，并在主线程中执行回调方法。
      return new MainThreadExecutor();
    }

    @Override CallAdapter.Factory defaultCallAdapterFactory(@Nullable Executor callbackExecutor) {
      if (callbackExecutor == null) throw new AssertionError();
      //创建默认的网络请求适配器工厂
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }

    static class MainThreadExecutor implements Executor {
    
    //获取Android主线程绑定的Handler
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
       //切换到Android主线程切换数据
        handler.post(r);
      }
    }
    //线程切换的流程
    1.回调ExecutorCallAdapterFactory生成一个ExecutorCallbackCall对象
    2.通过调用ExecutorCallbackCall.equeue(callback)从而调用MainThreadExecutor的execute()，通过handler切换到主线程。
  }
  
  
  Builder(Platform platform) {
  this.platform = platform;
  // 放入一个内置的数据转换器工厂BuiltInConverters(
  converterFactories.add(new BuiltInConverters());
}
```

Builer 设置了默认的：

* 平台类型对象：Android
* 网络请求器：CallAdapterFactory
* 数据转换器：converterFactory
* 回调执行器：callbackExecutor

#### 步骤3

![](http://upload-images.jianshu.io/upload_images/944365-3278e5d69629c84e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```java
public static final class Builder {

 ···
 public Builder baseUrl(String baseUrl) {
  checkNotNull(baseUrl, "baseUrl == null");
  HttpUrl httpUrl = HttpUrl.parse(baseUrl);
  if (httpUrl == null) {
    throw new IllegalArgumentException("Illegal URL: " + baseUrl);
  }
  return baseUrl(httpUrl);
}

// 检测最后一个碎片来检查URL参数是不是以"/"结尾
public Builder baseUrl(HttpUrl baseUrl) {
  checkNotNull(baseUrl, "baseUrl == null");
  List<String> pathSegments = baseUrl.pathSegments();
  if (!"".equals(pathSegments.get(pathSegments.size() - 1))) {
    throw new IllegalArgumentException("baseUrl must end in /: " + baseUrl);
  }
  this.baseUrl = baseUrl;
  return this;
}
···

}
```

#### 步骤4

![](http://upload-images.jianshu.io/upload_images/944365-4fa550eb89257774.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先看 GsonConvertFactory.create()：

```java
public final class GsonConverterFactory extends Converter.Factory {

  public static GsonConverterFactory create() {
    //创建一个 Gson 对象
    return create(new Gson());
  }

  
  public static GsonConverterFactory create(Gson gson) {
    if (gson == null) throw new NullPointerException("gson == null");
    //创建一个含有Gson对象实例的GsonConverterFactory对象
    return new GsonConverterFactory(gson);
  }

  private final Gson gson;

  private GsonConverterFactory(Gson gson) {
    this.gson = gson;
  }
}
```
GsonConverterFactory.create() 是创建了一个包含 Gson 对象实例的 GsonConverterFactory 对象，并返回给 addConverterFactory()，接下来看看 addConverterFactory() 方法。

```java
//将上面创建的GsonConverterFactory添加到converterFactories的数组中
public Builder addConverterFactory(Converter.Factory factory) {
  converterFactories.add(checkNotNull(factory, "factory == null"));
  return this;
}
```

>步骤4用于创建一个包含 Gson 对象实例的 GsonConverterFactory 并放入到数据转换器工厂 converterFactories 里。
>1. Retrofit 默认使用 Gson 解析；
>2. 若使用其他解析方式，可以通过自定义数据解析器来实现；

#### 步骤5

![](http://upload-images.jianshu.io/upload_images/944365-b3173bddeada3f07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面来看 build()方法

```java
public Retrofit build() {
  if (baseUrl == null) {
    throw new IllegalStateException("Base URL required.");
  }
  
  //配置网络请求执行器
  okhttp3.Call.Factory callFactory = this.callFactory;
  //默认使用 OkHttp 网络请求
  if (callFactory == null) {
    callFactory = new OkHttpClient();
  }

  //配置回调方法执行器
  Executor callbackExecutor = this.callbackExecutor;
  if (callbackExecutor == null) {
    callbackExecutor = platform.defaultCallbackExecutor();
  }

  // 配置网络请求适配器工厂
  List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
  adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

  // 配置数据转换器工厂
  List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);
    
  //根据
  return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
      callbackExecutor, validateEagerly);
}
```

## 2.2 创建网络请求接口实例

### 2.2.1 使用步骤

1. 定义接收网络数据的类
2. 定义网络请求的接口类
3. 创建网络请求的接口类实例
4. 生成最后的网络请求对象

```java
<-- 步骤1：定义接收网络数据的类 -->
<-- JavaBean.java -->
public class JavaBean {
  .. // 这里就不介绍了
  }

<-- 步骤2：定义网络请求的接口类 -->
<-- AccessApi.java -->
public interface AccessApi {
    // 注解GET：采用Get方法发送网络请求
    // Retrofit把网络请求的URL分成了2部分：1部分baseurl放在创建Retrofit对象时设置；另一部分在网络请求接口设置（即这里）
    // 如果接口里的URL是一个完整的网址，那么放在创建Retrofit对象时设置的部分可以不设置
    @GET("openapi.do?keyfrom=Yanzhikai&key=2032414398&type=data&doctype=json&version=1.1&q=car")
    // 接受网络请求数据的方法
    Call<JavaBean> getCall();
    // 返回类型为Call<*>，*是解析得到的数据类型，即JavaBean
}

<-- 步骤3：在MainActivity创建接口类实例  -->
AccessApi NetService = retrofit.create(AccessApi.class);

<-- 步骤4：对发送请求的url进行封装，即生成最终的网络请求对象  --> 
        Call<JavaBean> call = NetService.getCall();
```

### 2.2.2 源码分析

>Retrofit 是通过代理模式使用 create() 方法创建网络请求接口的实例（通过网络请求接口里设置的注解进行网络请求参数的配置）。

下面主要分析步骤3和步骤4

```java
<-- 步骤3：在MainActivity创建接口类实例  -->
AccessApi NetService = retrofit.create(AccessApi.class);

<-- 步骤4：对发送请求的url进行封装，即生成最终的网络请求对象  --> 
        Call<JavaBean> call = NetService.getCall();
```

#### 步骤3

下面来看 Retrofit 的 create 方法。

```java
  public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
     //判断是否需要提前验证
     //具体方法作用：
     // 1. 给接口中每个方法的注解进行解析并得到一个ServiceMethod对象
     // 2. 以 Method 为键将该对象存入 LinkedHashMap 集合中
      eagerlyValidateMethods(service);
      //如果不进行提前验证则进行动态解析对应方法，得到一个ServiceMethod对象，最后存入到 LinkedHashMap 集合中
    }
    
    //创建网络请求的动态代理对象，即通过动态代理创建网络请求接口实例
    //动态代理是为了拿到网络请求接口实例上的所有注解
    //调用interface接口的方法实际上是通过调用InvocationHandler的invoke方法来完成指定的功能
    return (T) Proxy.newProxyInstance(
    service.getClassLoader(), //动态生成接口的实现类
    new Class<?>[] { service }, //动态创建实例
        new InvocationHandler() { //将代理类的实现交给InvocationHandler类作为具体的实现
          private final Platform platform = Platform.get();
            
            //执行真正的逻辑
          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            
            ···
            
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
  }
  
    private void eagerlyValidateMethods(Class<?> service) {
    Platform platform = Platform.get();
    for (Method method : service.getDeclaredMethods()) {
      if (!platform.isDefaultMethod(method)) {
        loadServiceMethod(method);
      }
    }
  }

  ServiceMethod<?, ?> loadServiceMethod(Method method) {
    ServiceMethod<?, ?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder<>(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```

下面具体看下 invoke() 方法实现

```java
new InvocationHandler() {
  private final Platform platform = Platform.get();
    
    //执行真正的逻辑
  @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
      throws Throwable {
    
    ···
    //读取网络请求接口里的方法，并根据配置好的属性配置serviceMethod对象
    ServiceMethod<Object, Object> serviceMethod =
        (ServiceMethod<Object, Object>) loadServiceMethod(method);
    //根据配置好的serviceMethod对象创建okHttpCall对象
    OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
    
    //调用okHttp，并根据okHttpCall返回RxJava的Observe对象或者返回Call
    return serviceMethod.callAdapter.adapt(okHttpCall);
  }
}
```

#### ServiceMethod

下面分析 loadServiceMethod(method) 方法

```java
//一个ServiceMethod对象对应于网络请求接口里的一个方法。

  ServiceMethod<?, ?> loadServiceMethod(Method method) {
  //判断缓存中是否有该对象
    ServiceMethod<?, ?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
      result = serviceMethodCache.get(method);
      if (result == null) {
        //没有缓存则创建一个新的ServiceMethod实例
        result = new ServiceMethod.Builder<>(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```

下面分析通过3个步骤 ServiceMethod 实例的创建过程

![](http://upload-images.jianshu.io/upload_images/944365-3b99234427f4717a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### ServiceMethod 构造函数

```java
final class ServiceMethod<R, T> {
  // Upper and lower characters, digits, underscores, and hyphens, starting with a character.
  static final String PARAM = "[a-zA-Z][a-zA-Z0-9_-]*";
  static final Pattern PARAM_URL_REGEX = Pattern.compile("\\{(" + PARAM + ")\\}");
  static final Pattern PARAM_NAME_REGEX = Pattern.compile(PARAM);

  final okhttp3.Call.Factory callFactory; //网络请求工厂
  final CallAdapter<R, T> callAdapter; //网络请求适配器

  private final HttpUrl baseUrl; //网络请求地址
  private final Converter<ResponseBody, R> responseConverter; //数据转换器
  private final String httpMethod; //网络请求的http方法
  private final String relativeUrl; //网络请求相对地址
  private final Headers headers; //请求头 键值对
  private final MediaType contentType; //网络请求的http报文body类型
  private final boolean hasBody;
  private final boolean isFormEncoded;
  private final boolean isMultipart;
  //解析接口定义时每个方法的参数，并在构造 HTTP 请求时设置参数
  private final ParameterHandler<?>[] parameterHandlers;

  ServiceMethod(Builder<R, T> builder) {
    this.callFactory = builder.retrofit.callFactory();
    this.callAdapter = builder.callAdapter;
    this.baseUrl = builder.retrofit.baseUrl();
    this.responseConverter = builder.responseConverter;
    this.httpMethod = builder.httpMethod;
    this.relativeUrl = builder.relativeUrl;
    this.headers = builder.headers;
    this.contentType = builder.contentType;
    this.hasBody = builder.hasBody;
    this.isFormEncoded = builder.isFormEncoded;
    this.isMultipart = builder.isMultipart;
    this.parameterHandlers = builder.parameterHandlers;
  }
}
```

##### ServiceMethod 的 Builder()

```java
Builder(Retrofit retrofit, Method method) {
  this.retrofit = retrofit;
  this.method = method;
  //获取网络请求方法中的注解
  this.methodAnnotations = method.getAnnotations();
  //获取网络请求接口方法的参数类型
  this.parameterTypes = method.getGenericParameterTypes();
  //获取网络请求接口方法里的注解内容
  this.parameterAnnotationsArray = method.getParameterAnnotations();
}
```

##### ServiceMethod 的 build()

```java
    public ServiceMethod build() {
        //从Retrofit对象中获取网络请求适配器
      callAdapter = createCallAdapter();
      //获取该网络请求适配器返回的数据类型
      responseType = callAdapter.responseType();
      if (responseType == Response.class || responseType == okhttp3.Response.class) {
        throw methodError("'"
            + Utils.getRawType(responseType).getName()
            + "' is not a valid response body type. Did you mean ResponseBody?");
      }
      //获取数据转换器
      responseConverter = createResponseConverter();
      
      //解析网络请求接口中的方法的注解
      //主要是解析获取HTTP请求的方法
      for (Annotation annotation : methodAnnotations) {
        parseMethodAnnotation(annotation);
      }

      int parameterCount = parameterAnnotationsArray.length;
      parameterHandlers = new ParameterHandler<?>[parameterCount];
      for (int p = 0; p < parameterCount; p++) {
        Type parameterType = parameterTypes[p];
        Annotation[] parameterAnnotations = parameterAnnotationsArray[p];
        //对方法参数中的注解进行解析
        //包括 Body、PartMap、Part、FieldMap
        parameterHandlers[p] = parseParameter(p, parameterType, parameterAnnotations);
      }

      return new ServiceMethod<>(this);
    }

    private CallAdapter<T, R> createCallAdapter() {
     //获取网络请求接口里方法的返回值类型
      Type returnType = method.getGenericReturnType();
     //获取请求接口方法的注解
      Annotation[] annotations = method.getAnnotations();
      try {
        //从retrofit对象中获取对应的网络请求适配器
        return (CallAdapter<T, R>) retrofit.callAdapter(returnType, annotations);
      } catch (RuntimeException e) {
      }
    }
    
    //retrofit 的 callAdapter 方法
    public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
    return nextCallAdapter(null, returnType, annotations);
  }

  public CallAdapter<?, ?> nextCallAdapter(@Nullable CallAdapter.Factory skipPast, Type returnType,
      Annotation[] annotations) {
    int start = adapterFactories.indexOf(skipPast) + 1;
    for (int i = start, count = adapterFactories.size(); i < count; i++) {
    //根据returnType 获取对应的网络请求适配器
      CallAdapter<?, ?> adapter = adapterFactories.get(i).get(returnType, annotations, this);
      if (adapter != null) {
        return adapter;
      }
    }
    
    throw new IllegalArgumentException(builder.toString());

    }
    
        private Converter<ResponseBody, T> createResponseConverter() {
      Annotation[] annotations = method.getAnnotations();
      try {
        return retrofit.responseBodyConverter(responseType, annotations);
      } catch (RuntimeException e) {
      }
    }
    
    //retrofit 的 responseBodyConverter 方法
      public <T> Converter<ResponseBody, T> responseBodyConverter(Type type, Annotation[] annotations) {
    return nextResponseBodyConverter(null, type, annotations);
  }

  public <T> Converter<ResponseBody, T> nextResponseBodyConverter(
      @Nullable Converter.Factory skipPast, Type type, Annotation[] annotations) {

    int start = converterFactories.indexOf(skipPast) + 1;
    for (int i = start, count = converterFactories.size(); i < count; i++) {
        //获取数据转换器
      Converter<ResponseBody, ?> converter =
          converterFactories.get(i).responseBodyConverter(type, annotations, this);
      if (converter != null) {
        //noinspection unchecked
        return (Converter<ResponseBody, T>) converter;
      }
    }

    throw new IllegalArgumentException(builder.toString());
  }
  }
  
  
  //retrofit2.converter.gson.GsonConverterFactory
    @Override
  public Converter<ResponseBody, ?> responseBodyConverter(Type type, Annotation[] annotations,
      Retrofit retrofit) {
    TypeAdapter<?> adapter = gson.getAdapter(TypeToken.get(type));
    return new GsonResponseBodyConverter<>(gson, adapter);
  }
  
  //Gson数据转换
  final class GsonResponseBodyConverter<T> implements Converter<ResponseBody, T> {
  private final Gson gson;
  private final TypeAdapter<T> adapter;

  GsonResponseBodyConverter(Gson gson, TypeAdapter<T> adapter) {
    this.gson = gson;
    this.adapter = adapter;
  }

  @Override public T convert(ResponseBody value) throws IOException {
    JsonReader jsonReader = gson.newJsonReader(value.charStream());
    try {
      return adapter.read(jsonReader);
    } finally {
      value.close();
    }
  }
}

}
```


##### OkHttpCall 对象

根据 `loadServiceMethod(method)` 配置好的 `ServiceMethod` 对象和输入的参数创建OkHttpCall 对象。

`OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args)`

```java
final class OkHttpCall<T> implements Call<T> {
  
  private final ServiceMethod<T, ?> serviceMethod; //含所有网络请求参数信息的对象
  private final @Nullable Object[] args; // 网络请求接口的参数
  private volatile boolean canceled;
  private @Nullable okhttp3.Call rawCall; //实际进行网络访问的类
  private @Nullable Throwable creationFailure; //
  private boolean executed;

 //构造函数
  OkHttpCall(ServiceMethod<T, ?> serviceMethod, @Nullable Object[] args) {
    this.serviceMethod = serviceMethod;
    this.args = args;
  }
}
```

##### 返回具体的网络请求器

将上面创建的 OkHttpCall 对象传给第一步创建的 ServiceMethod 对象中对应的网络请求适配工厂的 adapt() 中。

```java
return serviceMethod.callAdapter.adapt(okHttpCall);
```
>返回类型对象：Android 默认的是 Call<>，如果设置了 RxJavaCallAdapterFactory，返回的则是 Observable<>

```java
  static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      //回调方法执行器用于线性切换
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override public void enqueue(final Callback<T> callback) {
      checkNotNull(callback, "callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }
  }
```

#### 步骤4 创建最终的网络请求对象

```java
Call<JavaBean> call = NetService.getCall();
```

* `NetService`对象实际上是动态代理对象`Proxy.newProxyInstance()`，并不是真正的网球请求接口创建的对象
* 当`NetService`对象调用`getCall()`方法时会被动态代理对象`Proxy.newProxyInstance()`拦截，然后调用自身的`InvocationHandler # invoke()`。
* `invoke(Object proxy, Method method, Object... args)`会传入三个参数（代理对象、方法、方法的参数）
* 接下来利用 Java 反射获取到`getCall()`的注解信息，配合 args 参数创建 ServiceMethod 对象。
* 最终创建并返回一个`OkHttpCall`类型的 Call 对象。

>1. OkHttpCall 类是 OkHttp 的包装类。
>2. 创建了 OkHttpCall 类型的 Call 对象还不能发送网络请求，需要创建 Request 对象才能发送网络请求。

**总结**

Retrofit采用了外观模式统一调用创建网络请求接口实例和网络请求参数配置的方法，具体细节是：

* 动态创建网络请求接口的实例（代理模式 - 动态代理）
* 创建 serviceMethod 对象（建造者模式 & 单例模式（缓存机制））
* 对 serviceMethod 对象进行网络请求参数配置：通过解析网络请求接口方法的参数、返回值和注解类型，从Retrofit对象中获取对应的网络请求的url地址、网络请求执行器、网络请求适配器 & 数据转换器。（策略模式）
* 对 serviceMethod 对象加入线程切换的操作，便于接收数据后通过Handler从子线程切换到主线程从而对返回数据结果进行处理（装饰模式）
* 最终创建并返回一个OkHttpCall类型的网络请求对象

## 2.3 执行网络请求

Retrofit 默认使用 OkHttp，即 OkHttpCall 类，但可以自定义设置自己需要的 Call 类。

OkHttpCall 提供了两种网络请求方式。

1. 同步请求：`OkHttpCall.execute()`。
2. 异步请求：`OkHttpCall.enqueue()`

### 2.3.1 同步请求

#### 发送请求过程

1. 对网络请求接口方法中的每个参数利用对应的`ParameterHandler`进行解析，再根据`ServiceMethod`对象创建一个`OkHttp`的`Request`对象。
2. 使用`OkHttp`的 Request 发送网络请求。
3. 对返回的数据使用设置的数据转换器解析返回的数据，最终得到一个 `Response<T>` 对象。

#### 具体使用

```java
Response<JavaBean> response = call.execute(); 
```

#### 源码分析

```java
  @Override public Response<T> execute() throws IOException {
    okhttp3.Call call;

    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      if (creationFailure != null) {
        if (creationFailure instanceof IOException) {
          throw (IOException) creationFailure;
        } else {
          throw (RuntimeException) creationFailure;
        }
      }

      call = rawCall;
      if (call == null) {
        try {
         //创建一个OkHttp的Request对象
          call = rawCall = createRawCall();
        } catch (IOException | RuntimeException e) {
          creationFailure = e;
          throw e;
        }
      }
    }

    if (canceled) {
      call.cancel();
    }
    //2.调用OkHttp的execute方法发送网络请求（同步）
    //3.解析网络请求返回的数据
    return parseResponse(call.execute());
  }

  private okhttp3.Call createRawCall() throws IOException {
    //返回Request对象
    Request request = serviceMethod.toRequest(args);
    //创建一个okhttp3.Call对象
    okhttp3.Call call = serviceMethod.callFactory.newCall(request);
    if (call == null) {
      throw new NullPointerException("Call.Factory returned null.");
    }
    return call;
  }

  Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
    ResponseBody rawBody = rawResponse.body();

    // Remove the body's source (the only stateful object) so we can pass the response along.
    rawResponse = rawResponse.newBuilder()
        .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
        .build();

    int code = rawResponse.code();
    if (code < 200 || code >= 300) {
      try {
        // Buffer the entire body to avoid future I/O.
        ResponseBody bufferedBody = Utils.buffer(rawBody);
        return Response.error(bufferedBody, rawResponse);
      } finally {
        rawBody.close();
      }
    }

    if (code == 204 || code == 205) {
      rawBody.close();
      return Response.success(null, rawResponse);
    }

    ExceptionCatchingRequestBody catchingBody = new ExceptionCatchingRequestBody(rawBody);
    try {
      T body = serviceMethod.toResponse(catchingBody);
      return Response.success(body, rawResponse);
    } catch (RuntimeException e) {
      // If the underlying source threw an exception, propagate that rather than indicating it was
      // a runtime exception.
      catchingBody.throwIfCaught();
      throw e;
    }
  }

```

特别注意：

* ServiceMethod几乎保存了一个网络请求所需要的数据
* 发送网络请求时，OkHttpCall需要从ServiceMethod中获得一个Request对象
* 解析数据时，还需要通过ServiceMethod使用Converter（数据转换器）转换成Java对象进行数据解析

>为了提高效率，Retrofit还会对解析过的请求ServiceMethod进行缓存，存放在Map<Method, ServiceMethod> serviceMethodCache = new LinkedHashMap<>();对象中，即第二步提到的单例模式

![](https://upload-images.jianshu.io/upload_images/944365-98f84c6cf564936d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### 2.3.2 异步请求

#### 发送请求过程

* 步骤1：对网络请求接口的方法中的每个参数利用对应ParameterHandler进行解析，再根据ServiceMethod对象创建一个OkHttp的Request对象
* 步骤2：使用OkHttp的Request发送网络请求；
* 步骤3：对返回的数据使用之前设置的数据转换器（GsonConverterFactory）解析返回的数据，最终得到一个Response<T>对象
* 步骤4：进行线程切换从而在主线程处理返回的数据结果

异步请求的过程跟同步请求类似，唯一不同之处在于：异步请求会将回调方法交给回调执行器在指定的线程中执行。**指定的线程此处是指主线程（UI线程）。**

#### 具体使用

```java
call.enqueue(new Callback<JavaBean>() {
            @Override
            public void onResponse(Call<JavaBean> call, Response<JavaBean> response) {
                System.out.println(response.isSuccessful());
                if (response.isSuccessful()) {
                    response.body().show();
                }
                else {
                    try {
                        System.out.println(response.errorBody().string());
                    } catch (IOException e) {
                        e.printStackTrace();
                    } ;
                }
            }
```

* 从上面分析有：call是一个静态代理
* 使用静态代理的作用是：在okhttpCall发送网络请求的前后进行额外操作

>这里的额外操作是：线程切换，即将子线程切换到主线程，从而在主线程对返回的数据结果进行处理

#### 源码分析

```java
 //ExecutorCallbackCall<T>类
     @Override public void enqueue(final Callback<T> callback) {
      checkNotNull(callback, "callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }
    
    <-- 分析1：delegate.enqueue（）解析 -->
@Override 
public void enqueue(final Callback<T> callback) {
   
    okhttp3.Call call;
    Throwable failure;

// 步骤1：创建OkHttp的Request对象，再封装成OkHttp.call
     // delegate代理在网络请求前的动作：创建OkHttp的Request对象，再封装成OkHttp.call
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already executed.");
      executed = true;

      call = rawCall;
      failure = creationFailure;
      if (call == null && failure == null) {
        try {
         
          call = rawCall = createRawCall(); 
          // 创建OkHttp的Request对象，再封装成OkHttp.call
         // 方法同发送同步请求，此处不作过多描述  
        } catch (Throwable t) {
          failure = creationFailure = t;
        }
      }

// 步骤2：发送网络请求
    // delegate是OkHttpcall的静态代理
    // delegate静态代理最终还是调用Okhttp.enqueue进行网络请求
    call.enqueue(new okhttp3.Callback() {
      @Override 
        public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse)
          throws IOException {
        Response<T> response;
        try {
        
          // 步骤3：解析返回数据
          response = parseResponse(rawResponse);
        } catch (Throwable e) {
          callFailure(e);
          return;
        }
        callSuccess(response);
      }

      @Override 
         public void onFailure(okhttp3.Call call, IOException e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callFailure(Throwable e) {
        try {
          callback.onFailure(OkHttpCall.this, e);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }

      private void callSuccess(Response<T> response) {
        try {
          callback.onResponse(OkHttpCall.this, response);
        } catch (Throwable t) {
          t.printStackTrace();
        }
      }
    });
  }

// 请回去上面分析1的起点

<-- 分析2：异步请求后的线程切换-->
// 线程切换是通过一开始创建Retrofit对象时Platform在检测到运行环境是Android时进行创建的：（之前已分析过）
// 采用适配器模式
static class Android extends Platform {

    // 创建默认的回调执行器工厂
    // 如果不将RxJava和Retrofit一起使用，一般都是使用该默认的CallAdapter.Factory
    // 后面会对RxJava和Retrofit一起使用的情况进行分析
    @Override
      CallAdapter.Factory defaultCallAdapterFactory(Executor callbackExecutor) {
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }

    @Override 
      public Executor defaultCallbackExecutor() {
      // 返回一个默认的回调方法执行器
      // 该执行器负责在主线程（UI线程）中执行回调方法
      return new MainThreadExecutor();
    }

    // 获取主线程Handler
    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());


      @Override 
      public void execute(Runnable r) {
        // Retrofit获取了主线程的handler
        // 然后在UI线程执行网络请求回调后的数据显示等操作。
        handler.post(r);
      }
    }

// 切换线程的流程：
// 1. 回调ExecutorCallAdapterFactory生成了一个ExecutorCallbackCall对象
// 2. 通过调用ExecutorCallbackCall.enqueue(CallBack)从而调用MainThreadExecutor的execute()通过handler切换到主线程处理返回结果（如显示在Activity等等）
  }
```

# 3 总结

Retrofit 本质上是一个 RESTful 的HTTP 网络请求框架的封装，即通过 大量的设计模式 封装了 OkHttp ，使得简洁易用。具体过程如下：

* Retrofit 将 Http请求 抽象 成 Java接口
* 在接口里用 注解 描述和配置 网络请求参数
* 用动态代理 的方式，动态将网络请求接口的注解 解析 成HTTP请求
* 最后执行HTTP请求

![](https://upload-images.jianshu.io/upload_images/944365-56df9f9ed647f7da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

# 参考文章

* [Android：手把手带你 深入读懂 Retrofit 2.0 源码)](https://www.jianshu.com/p/0c055ad46b6c)
* [Android：Retrofit 与 RxJava联合使用大合集](http://blog.csdn.net/carson_ho/article/details/79125101)
* [Carson_Ho的博客](http://blog.csdn.net/carson_ho)

