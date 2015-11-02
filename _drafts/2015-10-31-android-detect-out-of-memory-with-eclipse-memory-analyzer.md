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
###获取app的内存占用情况
安装好MAT后，接下来需要去获取app的内存占用情况。通过DDMS可以导出内存快照（heap dump），导出后的文件后缀是hprof，这个文件里记录了java对象和类在heap中的占用情况。  
导出后的hprof文件并不能直接使用MAT读取，需要通过Android SDK提供的转换工具hprof-conv进行格式转换。hprof-conv工具在sdk的platform-tools文件夹里。转换后的文件就可以使用MAT打开了。  
通过DDMS导出dump heap的步骤较繁琐，在此不再介绍。以下是导出hprof文件的shell脚本，记得要把adb和hprof-conv（这两个工具都在sdk的platform-tools文件夹里）加进系统环境变量里。  
<pre class="mcode">
#!/bin/bash
# 获取包名为com.example.outofmemorydetect的进程id
pid=`adb shell ps | grep com.example.outofmemorydetect | grep -v leakcanary | awk '{ print $2 }'`
# 调用dumpheap生成hprof文件，放在手机的sdcard文件夹里
adb shell am dumpheap $pid /sdcard/dumpheap.hprof
sleep 1
# 将手机里的hprof文件推送到电脑上
adb pull /sdcard/dumpheap.hprof temp.hprof
# 转换hprof文件
hprof-conv temp.hprof dumpheap.hprof
# 删除临时文件
rm temp.hprof
</pre>
