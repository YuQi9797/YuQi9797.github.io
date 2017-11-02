---
layout: post
title: "HttpURLConnection(Demo)"
date: 2017-11-02 11:00
description: "Android中使用HTTP协议访问网络"
tag: Android
---
前面学会了HttpURLConnection的相关知识，我们简单来做一个小案例。ヾ(*´▽‘*)ﾉ

首先修改activity_main.xml中的代码：

这里放置了一个Button和一个TextView，当我们点击Button发送HTTP请求，将从服务器获取到的数据显示到TextView中，由于数据可能会比较长，我们使用ScrollView控件，可以以滚动的形式查看屏幕外的那部分内容。
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/send_request"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send Request"/>

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:id="@+id/response_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
    </ScrollView>

</LinearLayout>
```

接着我们修改MainActivity中的代码：
```
package com.qiyu.network;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class MainActivity extends AppCompatActivity {

    TextView responseText;
    Button sendRequest;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        sendRequest = (Button) findViewById(R.id.send_request);
        responseText = (TextView) findViewById(R.id.response_text);
        sendRequest.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                sendRequestWithHttpURLConnection();
            }
        });
    }

    private void sendRequestWithHttpURLConnection() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                try {
                    URL url = new URL("https://www.baidu.com");
                    connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setConnectTimeout(8000);
                    connection.setReadTimeout(8000);
                    InputStream in = connection.getInputStream();

                    reader = new BufferedReader(new InputStreamReader(in));
                    StringBuilder response = new StringBuilder();
                    String line;
                    while ((line = reader.readLine()) != null) {
                        response.append(line);
                    }
                    showResponse(response.toString());
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    if (reader != null) {
                        try {
                            reader.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    if (connection != null) {
                        connection.disconnect();
                    }
                }
            }
        }).start();
    }

    private void showResponse(final String response) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                responseText.setText(response);
            }
        });
    }
}
```
可以看到我们在点击事件中调用了sendRequestWithHttpURLConnection()方法，在方法中开启了一个子线程，然后在子线程中使用HttpURLConnection发出一条HTTP请求，请求的目标地址是百度的首页。接着利用BufferedReader对服务器返回的流进行读取，将结果传入到showResponse()方法中。

在showResponse()方法中，我们调用了一个runOnUiThread()方法，将返回的数据显示在界面上。

★★★ Android不允许在子线程中进行UI操作，所以我们需要通过runOnUiThread()方法将线程切换到主线程，然后再更新UI元素。

附上效果图：
<div>
  <img src="/images/image/HttpURLConnection.png" width="231" height="372"/>
</div>

代码地址：https://github.com/Shanks07/androidSources/tree/master/Network
