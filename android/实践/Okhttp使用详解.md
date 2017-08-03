Okhttp是高性能的http库，支持同步、异步，而且实现了spdy、http2、websocket协议，api很简洁易用，和volley一样实现了http协议的缓存。Okhttp已经被Android官方采用，实现了几乎和Java.net.HttpURLConnection一样的功能。

# 下载地址

[官方介绍：http://square.github.io/okhttp/](http://square.github.io/okhttp/)
[Github源代码：https://github.com/square/okhttp](https://github.com/square/okhttp)

# 功能

-	一般的Get请求
-	一般的Post请求
-	基于Http的文件上传
-	文件下载
-	加载图片
-	支持请求回调，直接返回对象、对象集合
-	支持session的保持

# 使用

## Http Get

对于网络加载库，最常用的就是http get请求，比如获取一个网页的内容。

```Java
        //1.创建OkHttpClient对象
        OkHttpClient mOkHttpClient = new OkHttpClient();

        //2.创建一个Request
        final Request mRequest = new Request.Builder().url("https://www.baidu.com").build();

        //3.创建Call对象
        Call mCall = mOkHttpClient.newCall(mRequest);

        //4.请求加入调度
        mCall.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {

                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        mMsgTxt.setText("failure");
                    }
                });

            }

            @Override
            public void onResponse(Call call, final Response response) throws IOException {

                //字符串
                final String msg = response.body().string();
                //字节数组
                byte[] msgBytes = response.body().bytes();
                //流
                InputStream inputStream = response.body().byteStream();

                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        mMsgTxt.setText(msg);
                    }
                });


            }
        });

    }


```
以上就是发送一个Get请求的步骤

1. 首先构造一个Request对象，参数最少有个url，可以通过Request.Builder设置更多的参数，比如：header、method等。
2. 然后通过Request的对象去够着一个Call对象，类似于将你的请求封装成任务。
3. 最后，我们希望以异步的方式去执行请求，所以我们调用的是call.equeue，将call加入调度队列，然后等待任务执行完成，在Callback中即可得到结果。注意，回调方法都是运行在子线程中，如果需要操作控件，需要使用Handler切换到主线程。

上面是异步的方式执行get请求，当然也支持阻塞的方式，直接调用call.execute()方法返回一个Response。如下所示。

```Java
//阻塞调用
new Thread(new Runnable() {
	@Override
	public void run() {
		try {
			Response response = mCall.execute();
			System.out.println(response.body().string());
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}).start();
```

## Http post请求

```Java
Request request = buildMultipartFormRequest(
        url, new File[]{file}, new String[]{fileKey}, null);
FormEncodingBuilder builder = new FormEncodingBuilder();   
builder.add("username","liuguoquan");

Request request = new Request.Builder()
                   .url(url)
                .post(builder.build())
                .build();
 mOkHttpClient.newCall(request).enqueue(new Callback(){});
```
Post请求时，参数是包含在请求体中的，所以我们通过FormEncodingBuilder，添加多个String键值对，然后去构造RequestBody，最后完成Request的构造。

## 基于Http的文件上传

接下来构造一个RequestBody的Builder叫做MultipartBuilder。

```Java
File file = new File(Environment.getExternalStorageDirectory(), "balabala.mp4");

RequestBody fileBody = RequestBody.create(MediaType.parse("application/octet-stream"), file);

RequestBody requestBody = new MultipartBuilder()
     .type(MultipartBuilder.FORM)
     .addPart(Headers.of(
          "Content-Disposition", 
              "form-data; name=\"username\""), 
          RequestBody.create(null, "liu"))
     .addPart(Headers.of(
         "Content-Disposition", 
         "form-data; name=\"mFile\"; 
         filename=\"wjd.mp4\""), fileBody)
     .build();

Request request = new Request.Builder()
    .url("http://192.168.1.103:8080/okHttpServer/fileUpload")
    .post(requestBody)
    .build();

Call call = mOkHttpClient.newCall(request);
call.enqueue(new Callback()
{
    //...
});
```
上述代码向服务器传递了一个键值对username：liu和一个文件。通过MultipartBuilder的addPart方法可以添加键值对或者文件。

## 图片下载

图片下载和文件下载，这两个是通过回调的Response拿到byte[]然后解码成图片；文件下载就是拿到InputStream后做写文件操作。

参考文章：

[从原理角度解析Android （Java） http 文件上传](http://blog.csdn.net/lmj623565791/article/details/23781773)
[泡网：OkHttp使用教程](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html)
