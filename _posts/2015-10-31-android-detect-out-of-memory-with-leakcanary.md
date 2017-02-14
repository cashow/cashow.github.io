---
layout: post
title: "Android - 利用LeakCanary检测内存泄露"
date: 2015-10-31 22:39:59 +0800
tags: Android OutOfMemory 内存泄露 LeakCanary
---

Out of memory是android开发过程中常见的问题。在应用出现内存泄露问题时，任何一段需要占用内存的代码都有可能导致应用崩溃，这个时候友盟后台错误分析里给出的stacktrace并没有什么卵用。通过LeakCanary或者Eclipse Memory Analyzer（简称MAT），可以较方便地定位内存泄露的源头。  


***

### LeakCanary
LeakCanary是[Square](https://github.com/square)公司开发的一个用于检测OOM(out of memory的缩写)问题的开源库。  
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
是的，只需要这么简单的两步，你就可以开始检测自己app的out of memory问题了！  

***

### LeakCanary使用示例：  
以下是LeakCanary官方的demo，主要是在activity里开启一个会导致内存泄露的AsyncTask，在application里进行LeakCanary的初始化并打开[严格模式](http://developer.android.com/reference/android/os/StrictMode.html)。  
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
        new AsyncTask&lt;Void, Void, Void&gt;() {
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
    @Override 
    public void onCreate() {
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
运行以上代码后，点击按钮并旋转屏幕，这时由于AsyncTask还在执行，导致activity不能正常销毁，这就导致了内存泄露。  
稍等一会，通知栏就会出现MainActivity已经泄露的提示，如图所示：  
![leakcanary_notification](http://7xjvhq.com1.z0.glb.clouddn.com/leakcanary_notification.png)
点进去后，可以看到LeakCanary分析出来的内存泄露的原因。点击右边的加号可以看到更详细的信息。  
![leakcanary_leak_detail_info](http://7xjvhq.com1.z0.glb.clouddn.com/leakcanary_leak_detail_info.png)
在Logcat里也会有相应的信息：  
![leakcanary_leak_detail_info](http://7xjvhq.com1.z0.glb.clouddn.com/leakcanary_logcat.png)
由此可看出，MainActivity里面的AsyncTask导致了泄露。  

***

### 监测自定义的对象
在调用LeakCanary.install()时，LeakCanary会自动安装ActivityRefWatcher，在每个activity调用onDestroy()后通过判断activity的引用是否还在，确认activity是否已经泄露。  
另外，LeakCanary.install()还会返回一个已经配置好的RefWatcher。通过RefWatcher可以监测理应被系统回收的对象的状态。比如可以通过RefWatcher检测fragment是否泄露：  
<pre class='mcode'>
public abstract class BaseFragment extends Fragment {
    @Override 
    public void onDestroy() {
        super.onDestroy();
        RefWatcher refWatcher = MyApplication.getRefWatcher(getActivity());
        refWatcher.watch(this);
    }
}
</pre>
***

### 不监听特定的对象  
如果有些对象你希望LeakCanary忽略掉，可以创建自己的ExcludedRefs：  
<pre class='mcode'>
protected RefWatcher installLeakCanary() {
    ExcludedRefs excludedRefs = AndroidExcludedRefs.createAppDefaults()
        .instanceField("com.example.ExampleClass", "exampleField")
        .build();
    return LeakCanary.install(this, DisplayLeakService.class, excludedRefs);
}
</pre>
***

### 不监听特定的activity  
ActivityRefWatcher默认是监听所有的activity。重新实现LeakCanary的[install](https://github.com/square/leakcanary/blob/master/leakcanary-android/src/main/java/com/squareup/leakcanary/LeakCanary.java)方法可以自定义需要监听的activity。
<pre class='mcode'>
protected RefWatcher installLeakCanary() {
    if (LeakCanary.isInAnalyzerProcess(this)) {
        return RefWatcher.DISABLED;
    } else {
        ExcludedRefs excludedRefs =
                AndroidExcludedRefs.createAppDefaults().build();
        LeakCanary.enableDisplayLeakActivity(this);
        ServiceHeapDumpListener heapDumpListener =
                new ServiceHeapDumpListener(this, DisplayLeakService.class);
        final RefWatcher refWatcher =
              LeakCanary.androidWatcher(this, heapDumpListener, excludedRefs);
        registerActivityLifecycleCallbacks(
          new Application.ActivityLifecycleCallbacks() {
            @Override
            public void onActivityDestroyed(Activity activity) {
                // 在这里处理不需要监听的activity
                if (activity instanceof MainActivity) {
                    return;
                }
                refWatcher.watch(activity);
            }

            @Override
            public void onActivityCreated(Activity activity, 
                Bundle savedInstanceState) {
            }

            @Override
            public void onActivityStarted(Activity activity) {
            }

            @Override
            public void onActivityResumed(Activity activity) {
            }

            @Override
            public void onActivityPaused(Activity activity) {
            }

            @Override
            public void onActivityStopped(Activity activity) {
            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, 
                Bundle outState) {
            }

        });
        return refWatcher;
    }
}
</pre>
***

### 混淆  
如果在debug版本使用了混淆，需要在混淆文件加上以下的混淆配置：
<pre class="mcode">
# LeakCanary
-keep class org.eclipse.mat.** { *; }
-keep class com.squareup.leakcanary.** { *; }
</pre>
***

### 相关链接
Android - 利用Eclipse Memory Analyzer(MAT)检测内存泄露问题：  
<http://cashow.github.io/android-detect-out-of-memory-with-eclipse-memory-analyzer.html>