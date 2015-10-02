---
layout: post
title: "Android - 利用LeakCanary和Eclipse Memory Analyzer检测内存泄露"
date: 2015-07-20 23:49:32 +0800
tags: Android oom 内存泄露
---

Out of memory是android开发过程中常见的问题。在应用出现内存泄露问题时，任何一段需要占用内存的代码都有可能导致应用崩溃，这个时候友盟后台错误分析里给出的stacktrace并没有什么卵用。通过LeakCanary或者Eclipse Memory Analyzer（简称mat），可以较方便地定位内存泄露的源头。  
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