---
layout: post
title: "Android - 我使用过的第三方库和开发工具以及踩过的坑"
date: 2015-12-22 22:11:39 +0800
tags: Android
---

做android开发2年半了，接触到了不少第三方库和开发工具，每一个新接触到的工具都能避免重复造轮子，加快开发进度。然后在使用这些工具的过程中，我也踩过不少的坑。因此特意写篇博客简单介绍下各个库的功能，以及怎么避免踩坑。  

***

### weibo

#### 官网  
<https://github.com/sinaweibosdk/weibo_android_sdk>  

#### 简介  
新浪微博的sdk。没什么好介绍的，基本上大家都用过。  

#### 踩过的坑  
1.导入sdk后，在gradle编译时报错，提示“Multiple dex files define Lcom/sina/weibo/sdk/BuildConfig”。报错原因是weibosdkcore.jar里面有BuildConfig.class，这个文件本不应该出现在这里的。解决办法是将weibosdkcore.jar解开，删掉weibosdkcore.jar后重新生成jar。刚去官网看了眼，微博sdk近期有更新，不知道这个问题修好没有。  
2.友盟上看到有报错：“Service Intent must be explicit: Intent { act=com.sina.weibo.remotessoservice }”，原因是android 5.0开始，service必须显性声明。网上搜到一个解决办法：<https://ticket.avosapps.com/tickets/4069>，不过我还没测试过，不确定有没有用。

***

### qiNiu

#### 官网  
<http://developer.qiniu.com/>  

#### 简介  
七牛的sdk。  

#### 踩过的坑  
auth

***

### universalImageLoader
加载图片的库  
官网：<https://github.com/nostra13/Android-Universal-Image-Loader>  

### pullToRefresh
下拉刷新  

### emojicon
显示emoji表情的库  
官网：<https://github.com/rockerhieu/emojicon>  

### taeSDK
淘宝的sdk  
官网：<http://baichuan.taobao.com/>  

### gpuimage
滤镜  
官网：<https://github.com/CyberAgent/android-gpuimage>  

### butterknife:7.0.1
简化findViewById  
官网：<https://github.com/JakeWharton/butterknife>  

### android-async-http:1.4.8
网络请求  
官网：<https://github.com/loopj/android-async-http>  

### gson:2.2.4
解析json  
官网：<https://github.com/google/gson>  

### leakcanary-android:1.3
内存泄露检测  
官网：<https://github.com/square/leakcanary>  

### ActiveAndroid
数据库操作  
官网：<https://github.com/pardom/ActiveAndroid>  

### jpush
极光推送  
官网：<http://docs.jpush.io/>  

### umeng
友盟统计  
官网：<https://www.umeng.com/>  

### zxing
扫描二维码  
官网：<https://github.com/zxing/zxing>  

### zbar
扫描二维码  
官网：<https://github.com/ZBar/ZBar>  

### AMap
高德  
官网：<http://lbs.amap.com/>  

### retrolambda:3.2.0
使android代码支持lambda表达式  
官网：<https://github.com/evant/gradle-retrolambda>    

### RxJava:1.0.16  
官网：<https://github.com/ReactiveX/RxJava>    

### RxAndroid:1.0.1
官网：<https://github.com/ReactiveX/RxAndroid>      