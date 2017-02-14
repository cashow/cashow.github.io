---
layout: post
title: "Android组件 - 利用LocalBroadcastManager在应用内发送广播"
date: 2015-07-05 19:45:14 +0800
tags: Android Android组件 Broadcast
---

LocalBroadcastManager是android support v4包里提供的一个组件，用来在应用内发送广播。与普通的Broadcast相比，LocalBroadcast不需要发送全局广播，效率更高，而且发送的数据不能被其他应用接受，其他应用也不能发送这些广播到我们的app，因此不需要担心数据的安全性问题。  
LocalBroadcast的用法和Broadcast基本一致，只需要在Broadcast相关的代码前加上LocalBroadcastManager.getInstance(context)就行了。  

#### 发送广播：  
<pre class="mcode">
LocalBroadcastManager.getInstance(context).sendBroadcast(intent);  
</pre>

#### 注册广播监听器：  
<pre class="mcode">
LocalBroadcastManager.getInstance(context).registerReceiver(receiver, filter);
</pre>

#### 注销广播监听器：
<pre class="mcode">
LocalBroadcastManager.getInstance(context).unregisterReceiver(receiver);
</pre>

***
具体实现代码如下：  
<pre class="mcode">
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.support.v4.content.LocalBroadcastManager;
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;


public class MainActivity extends ActionBarActivity {
    private Button button;
    private TextView text;

    private MyReceiver receiver;
    private IntentFilter filter;

    private Context context;

    private static final String MY_BROADCAST_TAG = "com.example.localbroadcasttest";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        context = getApplicationContext();

        receiver = new MyReceiver();
        filter = new IntentFilter();
        filter.addAction(MY_BROADCAST_TAG);

        button = (Button) findViewById(R.id.button);
        text = (TextView) findViewById(R.id.text);

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent();
                intent.setAction(MY_BROADCAST_TAG);
                intent.putExtra("message", "Hello~");
                LocalBroadcastManager.getInstance(context).sendBroadcast(intent);
            }
        });
    }

    @Override
    protected void onResume() {
        super.onResume();
        LocalBroadcastManager.getInstance(context).registerReceiver(receiver, filter);
    }

    @Override
    protected void onPause() {
        super.onPause();
        LocalBroadcastManager.getInstance(context).unregisterReceiver(receiver);
    }

    class MyReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(Context arg0, Intent arg1) {
            String message = arg1.getStringExtra("message");
            text.setText("接受到广播：\ntext = " + message);
        }
    }
}
</pre>

***
官方文档：  
<https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html>