---
layout: post
title: "Android - 利用Eclipse Memory Analyzer(MAT)检测内存泄露问题"
date: 2015-11-22 14:37:59 +0800
tags: Android OutOfMemory 内存泄露 EclipseMemoryAnalyzer 原创
---

Out of memory是android开发过程中常见的问题。在应用出现内存泄露问题时，任何一段需要占用内存的代码都有可能导致应用崩溃，这个时候友盟后台错误分析里给出的stacktrace并没有什么卵用。通过LeakCanary或者Eclipse Memory Analyzer（简称MAT），可以较方便地定位内存泄露的源头。  

<div class="alert alert-success" role="alert">
为了复现用户使用过程中出现的 OutOfMemory 问题，我做了一个记录内存信息的库，感兴趣的同学可以去看我的这个项目：<a href="https://github.com/cashow/CashowMemoryMonitor">点击打开链接</a>
</div>

***

### Eclipse Memory Analyzer
Eclipse Memory Analyzer是一款用来分析Java内存的工具。通过Eclipse Memory Analyzer，可以很方便地看出哪些资源占用了app较大的内存，也可以推断出哪里发生了内存泄露。  
Eclipse Memory Analyzer官网：<https://eclipse.org/mat/>  
下载链接：<https://eclipse.org/mat/downloads.php>  
安装MAT的时候可以选择在eclipse里安装插件，或者直接下载安装不基于eclipse的独立版本，即下载链接里的Stand-alone Eclipse RCP Applications。  

***

### 测试代码
以下是android测试代码，在开启AsyncTask后旋转屏幕，可以造成内存泄露。
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

### 获取app的内存占用情况
安装好MAT后，接下来需要去获取app的内存占用情况。通过DDMS可以导出内存快照（heap dump），导出后的文件后缀是hprof，这个文件里记录了java对象和类在heap中的占用情况。  
导出的步骤如下：  
1.点击工具栏上的“Android Device Monitor”图标，打开DDMS。  
![mat_android_device_monitor](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_android_device_monitor.png)
2.在DDMS左侧选中应用包名，点击上方垃圾桶样式的“Cause GC”按钮，强制触发垃圾回收。  
![mat_ddms_cause_gc](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_ddms_cause_gc.png)
3.点击“Cause GC”按钮左边的“Dump HPROF file”按钮，弹出保存文件的对话框。
![mat_ddms_dump_hprof_file](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_ddms_dump_hprof_file.png)
![mat_ddms_save_hprof_file](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_ddms_save_hprof_file.png)
4.导出的hprof文件并不能直接使用MAT读取，还需要通过Android SDK提供的转换工具进行格式转换。这个转换工具叫hprof-conv，在sdk的platform-tools文件夹里。使用方法：  
<pre>
hprof-conv from.hprof to.hprof
</pre>
转换后的文件就可以使用MAT打开了。  
为了简化导出dump heap的步骤，可以考虑使用shell脚本简化第3、4步。记得要把adb和hprof-conv（这两个工具都在sdk的platform-tools文件夹里）加进系统环境变量里。  
<pre class="mcode">
#!/bin/bash
# 获取包名为com.example.outofmemorydetect的进程id
pid=`adb shell ps | grep com.example.outofmemorydetect | grep -v leakcanary | awk '{ print $2 }'`
# 调用dumpheap生成hprof文件，放在手机的sdcard文件夹里
adb shell am dumpheap $pid /data/local/tmp/dumpheap.hprof
sleep 1
# 将手机里的hprof文件推送到电脑上
adb pull /data/local/tmp/dumpheap.hprof temp.hprof
# 创建hprof文件夹
mkdir -p hprof
# 转换hprof文件，生成的dumpheap.hprof放在hprof文件夹里
hprof-conv temp.hprof hprof/dumpheap.hprof
# 删除临时文件
rm temp.hprof
adb shell rm /data/local/tmp/dumpheap.hprof
</pre>
执行上面的脚本后，可以在hprof文件夹里找到dumpheap.hprof文件，这个文件就是我们要分析的文件。  

***

### 打开hprof文件
打开MAT，通过菜单栏的 File - Open File.. 打开刚刚生成的dumpheap.hprof，弹出如下对话框：  
![mat_getting_started](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_getting_started.png)
第一个选项是自动分析可能存在内存泄露的对象。  
第二个选项是分析可疑对象，如字符串是否定义重了，空的collection、finalizer以及弱引用等。  
第三个选项是打开上次的报告。  
三个选项都不需要选中，把”Show this dialog ...“前面的勾去掉，点击cancel。  
点击右上角的Restore按钮：
![mat_restore](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_restore.png)
可以看到Overview界面，如下图所示：  
![mat_overview](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_overview.png)
这个界面包括了MAT对hprof文件的简要分析，鼠标移动到饼状图上可以在MAT的左侧看到详细的信息。  
分析饼状图后可以得出结论：我们的app总共使用了16.6MB的内存，其中8.3MB的内存被 android.content.res.Resources 占用，6.3MB的内存被 android.graphics.Bitmap 占用。

***

### 查看activity泄露情况
点击“Actions”下方的“Histogram”，可以查看内存里各个对象的数目以及占用的内存大小。
![mat_histogram](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_histogram.png)
第一列是类名，第二列是类的实例的数目，第三列是占用的内存大小。  
这里的数据并没有完全显示，点击列表下方的“Total: 36 of 2,473 ...”可以加载下一页的数据。  
可以在列表顶部的输入框里输入正则表达式过滤不需要的结果，比如输入包名可以查看我们自定义的类的内存占用情况。
![mat_histogram_filtered](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_histogram_filtered.png)
可以很明显地看到，MainActivity在内存里有5个实例，可我们在测试过程中没有开启新的activity，只是旋转了几次屏幕。由此可推断，旋转屏幕的过程中MainActivity没有被回收，即MainActivity发生了内存泄露。

***

### 定位内存泄露原因
在Histogram页面可以看到有3个MainActivity相关的信息，选中一条后右键，选择“Merge Shortest Paths to GC Roots” - “exclude weak references”，可看到MainActivity被引用的路径。  
以下是MainActivity$2对应的引用路径：
![mat_shortest_path](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_shortest_path.png)
可以看到，MainActivity里面还有3个不能被系统回收的AsyncTask。这几个AsyncTask可能就是MainActivity不能被系统回收的原因。  
为了确认AsyncTask就是导致内存泄露的罪魁祸首，可以把 startAsyncTask() 注释掉后再生成一次hprof文件。  
![mat_histogram_filtered_2](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_histogram_filtered_2.png)
现在MainActivity在内存里只有一个实例，整个世界清净了。

***

### 找到占用内存较大的bitmap
点击“Histogram”下面的“Domanitor Tree”按钮，进入到如下界面：  
![mat_domanitor_tree](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_domanitor_tree.png)
这个页面列出了占用内存较大的对象。  
注意第二行的 android.graphics.Bitmap，总共占用了37.50%的内存（总内存是16.6MB）。  
现在问题来了：这个bitmap是在什么地方出现的？占用这么多的内存是正常现象吗？  
为了解答以上问题，我们需要把这个bitmap导出成图片文件。  

#### 第一步：获取bitmap的宽度和高度
选中“android.graphics.Bitmap”栏，可以在左侧看到bitmap的相关信息：  
![mat_bitmap_info](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_bitmap_info.png)
记录下mWidth和mHeight的值。这两个值表示了bitmap的宽度和长度，比如现在要分析的这个bitmap是 1280 x 1280 的图片。  

#### 第二步：导出成data文件  
选中“android.graphics.Bitmap”下方的 “byte[6553600]” 栏，右键 - Copy - Save Value To File，将bitmap文件保存成后缀名为data的文件。  
![mat_bitmap_byte](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_bitmap_byte.png)
![mat_save_bitmap](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/mat_save_bitmap.png)

#### 第三步：使用GIMP打开data文件  
GIMP是一个开源的图像处理软件，包含大部分图象处理所需的功能，可以算是Linux下的PhotoShop。  
官网：<https://www.gimp.org/>  
下载链接：<https://www.gimp.org/downloads/>  
下载好GIMP后打开，将data文件拖到这个对话框内：  
![gimp_gnu](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/gimp_gnu.png)
出现以下对话框：  
![gimp_preview](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/gimp_preview.png)
图像类型选择RGB Alpha，宽度和高度填上第一步获取到的图片宽度和高度，预览图会显示图片的一小部分
![gimp_preview_2](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/gimp_preview_2.png)
点击open后可看到原图
![gimp_ori](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/gimp_ori.png)
这张图片就是 R.drawable.test 。  
现在找到了占用内存较大的图片，接下来就是考虑怎么减少图片占用的内存。一个常见的方法是将bitmap缩放后再放入ImageView里面。具体优化方法可以查看android官方文档的教程 "Displaying Bitmaps Efficiently" ：  

<https://developer.android.com/training/displaying-bitmaps/index.html>  

***


### 相关链接  
Android - 利用LeakCanary检测内存泄露：  

<http://cashow.github.io/android-detect-out-of-memory-with-leakcanary.html>
