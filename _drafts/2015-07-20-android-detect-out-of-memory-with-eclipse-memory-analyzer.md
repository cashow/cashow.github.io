---
layout: post
title: "Android - 利用Eclipse Memory Analyzer(MAT)检测内存泄露"
date: 2015-10-27 00:02:14 +0800
tags: Android oom 内存泄露
---

Out of memory是android开发过程中常见的问题。在应用出现内存泄露问题时，任何一段需要占用内存的代码都有可能导致应用崩溃，这个时候友盟后台错误分析里给出的stacktrace并没有什么卵用。通过LeakCanary或者Eclipse Memory Analyzer（简称MAT），可以较方便地定位内存泄露的源头。  
***
###LeakCanary
LeakCanary是[Square](https://github.com/square)公司开发的一个用于检测OOM问题的开源库。  
Github地址：<https://github.com/square/leakcanary>  
要在项目中使用LeakCanary，需要在build.gradle文件里加上  
<pre class="mcode">
dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3.1'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3.1'
}
</pre>
在application初始化时加上
<pre class="mcode">
LeakCanary.install(this);
</pre>
是的，只需要这么简单地两步，你就可以开始检测自己app的out of memory问题了！  
***
###LeakCanary使用示例：  
以下是LeakCanary官方的demo
Activity的代码：  
<pre class="mcode">
public class MainActivity extends ActionBarActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        View button = findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startAsyncTask();
            }
        });
    }

    private void startAsyncTask() {
        // This async task is an anonymous class and therefore has a hidden reference to the outer
        // class MainActivity. If the activity gets destroyed before the task finishes (e.g. rotation),
        // the activity instance will leak.
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                // Do some slow work in background
                SystemClock.sleep(20000);
                return null;
            }
        }.execute();
    }
}
</pre>
Application的代码：  
<pre class="mcode">
public class MyApplication extends Application {

    @Override public void onCreate() {
        super.onCreate();
        enabledStrictMode();
        LeakCanary.install(this);
    }

    private void enabledStrictMode() {
        if (SDK_INT >= GINGERBREAD) {
            StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                    .detectAll()
                    .penaltyLog()
                    .penaltyDeath()
                    .build());
        }
    }
}
</pre>