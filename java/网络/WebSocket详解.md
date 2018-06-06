[TOC]

**WebSocket 详解**

# 简介

>WebSocket protocol 是HTML5一种新的协议。它实现了浏览器与服务器全双工通信(full-duplex)。一开始的握手需要借助HTTP请求完成。

# WebSocket 握手

**1. 客户端、服务器建立TCP连接，三次握手。这是通信的基础，传输控制层，若失败后续都不执行。**

**2. TCP连接成功后，浏览器通过HTTP协议向服务器传送WebSocket支持的版本号等信息。（开始前的HTTP握手）**

```javascript
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Version: 13
Origin: http://example.com
```

`Upgrade: websocket Connection: Upgrade`是 WebSocket 的核心，告诉服务器客户端发起的是 WebSocket协议而不是 Http。

`Sec-WebSocket-Key`是 WebSocket 客户端发送的一个 base64 编码的密文，要求服务端必须返回一个对应加密的“Sec-WebSocket-Accept”应答，否则客户端会抛出“Error during WebSocket handshake”错误，并关闭连接。

**3. 服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据。**
 
 ```javascript
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
 ```
`Upgrade: websocket Connection: Upgrade`告诉客户端，服务器已经成功切换协议了。

`Sec-WebSocket-Accept`的值是服务端采用与客户端一致的密钥计算出来后返回客户端的。

`HTTP/1.1 101 Switching Protocols`表示服务端接受 WebSocket 协议的客户端连接，经过这样的请求-响应处理后，客户端服务端的 WebSocket 连接握手成功, 后续就可以进行 TCP 通讯了。

**4. 当收到了连接成功的消息后，通过TCP通道进行传输通信。**


# OkHttp 进行 WebSocket通信 

```java
OkHttpClient.Builder builder = new OkHttpClient.Builder();
builder.connectTimeout(3, TimeUnit.SECONDS);
builder.writeTimeout(3, TimeUnit.SECONDS);
builder.readTimeout(3, TimeUnit.SECONDS);
mOkHttpClient = builder.build();
// ws://echo.websocket.org
mRequest =
  new Request.Builder()
      .url(Constants.WSS)
      .build();
mWebSocket = mOkHttpClient.newWebSocket(mRequest, new WebSocketListener());
mOkHttpClient.dispatcher().executorService().shutdown();
```

