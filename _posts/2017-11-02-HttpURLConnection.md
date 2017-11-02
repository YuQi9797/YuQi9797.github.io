---
layout: post
title: "HttpURLConnection"
date: 2017-11-02 10:42
description: "Android中使用HTTP协议访问网络"
tag: Android
---

本次，我们学习官方建议使用的HttpURLConnection的用法。

首先我们需要获取到HttpURLConnection的实例，一般只需要new出一个URL对象，并传入目标的网络地址，然后调用一下openConnection()方法即可，如下所示：
```
URL url = new URL("https://www.baidu.com");
HttpURLConnection connection = (HttpURLConnection) uri.openConnection();
```
得到了HttpURLConnection实例后，我们需要设置一下HTTP请求所使用的方法。常用的方法有GET和POST两种。

GET表示希望从服务器那里获取数据，POST表示希望提交数据给服务器。
```
connection.setRequestMethod("GET");
```
我们还可以可以进行一些自由的定制，如设置连接超时、读取超时的毫秒数，以及服务器希望得到的一些消息头等。
```
connection.setConnectTimeout(8000);//设置连接超时
connection.setReadTimeout(8000);//设置读取超时
```
接着我们需要调用getInputStream()方法，获取到服务器返回的输入流。然后对输入流进行解读。
```
InputStream in = connection.getInputStream();
```
最后调用disconnect()方法，将这个HTTP连接关闭掉。
```
connection.disconnect();
```

☆在运行程序之前，别忘了声明一下网络权限
```
<uses-permission android:name = "android.permission.INTERNET"/>
```
