---
layout: post
title: "Android - 利用Eclipse Memory Analyzer(MAT)检测内存泄露"
date: 2015-10-31 22:42:48 +0800
tags: Android OutOfMemory 内存泄露 EclipseMemoryAnalyzer
---

Out of memory是android开发过程中常见的问题。在应用出现内存泄露问题时，任何一段需要占用内存的代码都有可能导致应用崩溃，这个时候友盟后台错误分析里给出的stacktrace并没有什么卵用。通过LeakCanary或者Eclipse Memory Analyzer（简称MAT），可以较方便地定位内存泄露的源头。  
***
###Eclipse Memory Analyzer
Eclipse Memory Analyzer是一个用来分析Java内存的工具。通过Eclipse Memory Analyzer，可以很方便地看出哪些资源占用了app较大的内存，也可以检测出哪里发生了内存泄露。  
Eclipse Memory Analyzer官网：<https://eclipse.org/mat/>  
下载链接：<https://eclipse.org/mat/downloads.php>  
安装MAT的时候可以选择在eclipse里安装插件，或者直接下载安装不基于eclipse的独立版本，即下载链接里的Stand-alone Eclipse RCP Applications。  
***
###测试代码
以下是android测试代码，在开启AsyncTask后立即旋转屏幕，可以造成内存泄露。
<pre class="mcode">
public class MainActivity extends ActionBarActivity {
    private Button button;
    private ImageView image;
    private Bitmap bitmap;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        button = (Button) findViewById(R.id.button);
        image = (ImageView) findViewById(R.id.image);

        Drawable drawable = getResources().getDrawable(R.drawable.test);
        bitmap = ((BitmapDrawable)drawable).getBitmap();
        image.setImageBitmap(bitmap);

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
***
###获取app的内存占用情况
安装好MAT后，接下来需要去获取app的内存占用情况。通过DDMS可以导出内存快照（heap dump），导出后的文件后缀是hprof，这个文件里记录了java对象和类在heap中的占用情况。  
然而，导出的hprof文件并不能直接使用MAT读取，还需要通过Android SDK提供的转换工具hprof-conv进行格式转换。hprof-conv工具在sdk的platform-tools文件夹里。转换后的文件就可以使用MAT打开了。  
为了简化导出dump heap的步骤，可以考虑使用shell脚本自动导出hprof文件。代码如下。记得要把adb和hprof-conv（这两个工具都在sdk的platform-tools文件夹里）加进系统环境变量里。  
<pre class="mcode">
#!/bin/bash
# 获取包名为com.example.outofmemorydetect的进程id
pid=`adb shell ps | grep com.example.outofmemorydetect | grep -v leakcanary | awk '{ print $2 }'`
# 调用dumpheap生成hprof文件，放在手机的sdcard文件夹里
adb shell am dumpheap $pid /sdcard/dumpheap.hprof
sleep 1
# 将手机里的hprof文件推送到电脑上
adb pull /sdcard/dumpheap.hprof temp.hprof
# 创建hprof文件夹
mkdir -p hprof
# 转换hprof文件，生成的dumpheap.hprof放在hprof文件夹里
hprof-conv temp.hprof hprof/dumpheap.hprof
# 删除临时文件
rm temp.hprof
</pre>
执行上面的脚本后，可以在hprof文件夹里找到dumpheap.hprof文件，这个文件就是我们要分析的文件。  
***
###分析hprof文件
打开MAT，通过菜单栏的 File - Open File.. 打开刚刚生成的dumpheap.hprof，弹出如下对话框：  
![mat_getting_started](http://7xjvhq.com1.z0.glb.clouddn.com/mat_getting_started.png)
第一个选项是自动分析可能存在内存泄露的对象。  
第二个选项是分析可疑对象，如字符串是否定义重了，空的collection、finalizer以及弱引用等。  
第三个选项是打开上次的报告。  
三个选项都不需要选中，把”Show this dialog ...“前面的勾去掉，点击cancel。  
点击右上角的Restore按钮：
![mat_restore](http://7xjvhq.com1.z0.glb.clouddn.com/mat_restore.png)
可以看到Overview界面，如下图所示：  
![mat_overview](http://7xjvhq.com1.z0.glb.clouddn.com/mat_overview.png)