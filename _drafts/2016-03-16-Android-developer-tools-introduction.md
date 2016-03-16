---
layout: post
title: "Android - 我使用过的第三方库和开发工具以及踩过的坑"
date: 2016-03-16 22:52:27 +0800
tags: Android Android开发工具
---

做android开发3年了，接触到了不少第三方库和开发工具，每一个新接触到的工具都能避免重复造轮子，加快开发进度。然而在使用这些工具的过程中，我也踩过不少的坑。因此特意写篇博客简单介绍下各个库的功能，以及怎么避免踩坑。  

***

### weiboSDK:3.1.2

####  官网  
<https://github.com/sinaweibosdk/weibo_android_sdk>  

####  简介  
新浪微博的sdk。分享微博以及微博登陆需要用到。  

####  踩过的坑  
1.旧版本的问题：导入sdk后，在gradle编译时报错，提示“Multiple dex files define Lcom/sina/weibo/sdk/BuildConfig”。报错原因是weibosdkcore.jar里面有BuildConfig.class，这个文件本不应该出现在这里的。解决办法是将weibosdkcore.jar解开，删掉weibosdkcore.jar后重新生成jar。  
2.旧版本的问题：友盟上看到有报错：“Service Intent must be explicit: Intent { act=com.sina.weibo.remotessoservice }”，原因是android 5.0开始，service必须显性声明。

***

### qiNiu

#### 官网  
<http://developer.qiniu.com/>  

#### 简介  
七牛的sdk，上传图片到七牛时需要用到。  
七牛是一个云存储的平台，可以用来当作app的图床。从七牛加载图片的时候可以设置自定义的图片样式，指定要获取的图片宽度、高度和缩放方式等等，七牛会将图片缩放后再返回给客户端。通过自定义的图片样式，可以保证控件从网络加载的图片不会太大，从而减少ImageView的内存消耗。  

***

### universalImageLoader

#### 官网 
<https://github.com/nostra13/Android-Universal-Image-Loader>

#### 简介  
加载图片的库  

#### 踩过的坑  
1.不能加载宽度或高度大于4000的图片。解决办法是先将图片下载下来，不使用universalImageLoader加载图片。  
2.这个坑其实算是java的坑。先看代码：  
<pre class="mcode">
ImageLoader.getInstance().displayImage(url, image, options, new ImageLoadingListener() {
    @Override
    public void onLoadingStarted(String imageUri, View view) {
    }

    @Override
    public void onLoadingFailed(String imageUri, View view, FailReason failReason) {
    }

    @Override
    public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
    }

    @Override
    public void onLoadingCancelled(String imageUri, View view) {
    }
});
</pre>
这样的代码会导致内存泄露，原因是代码中使用了匿名内部类，而非静态内部类（包括匿名内部类）会持有一个隐式的外部类引用，在这里是ImageLoadingListener引用了外部的activity。如果在图片没有完全加载的情况下退出activity，由于匿名内部类还持有对activity的引用，导致activity不能正确地销毁，这就造成了内存泄露。以下两种办法可以避免内存泄露：  
1.让activity实现ImageLoadingListener接口，在加载图片时使用activity作为图片加载的回调。 
<pre class="mcode">
public class MainActivity extends Activity implements ImageLoadingListener{
}

ImageLoader.getInstance().displayImage(url, image, options, this);
</pre>
2.写一个实现ImageLoadingListener的自定义类，在加载图片时使用实例化的自定义类。如果在回调里需要修改activity的部分控件，需要将这些控件加上WeakReference。
<pre class="mcode">
MyImageLoadingListener listener = new MyImageLoadingListener();
ImageLoader.getInstance().displayImage(url, image, options, listener);
</pre>

***

### emojicon

#### 官网 
<https://github.com/rockerhieu/emojicon>

#### 简介  
显示emoji表情的库。  

#### 踩过的坑  
1.emoji的字号如果和TextView的字号一样，显示出来的emoji会特别小，因此大部分EmojiconTextView都要重新设定emoji的字号。  
2.EmojiconTextView不能使用append()添加带有emoji的字段，因为EmojiconTextView只重写了TextView的setText()方法。解决办法是将EmojiconTextView代码里转换字符串的代码单独提取出来。  
<pre class="mcode">
public SpannableStringBuilder getTextBuilder(CharSequence text){
    SpannableStringBuilder builder = new SpannableStringBuilder((CharSequence)text);
    EmojiconHandler.addEmojis(this.getContext(), 
        builder, this.mEmojiconSize, this.mEmojiconAlignment, 
        this.mEmojiconTextSize, this.mTextStart, this.mTextLength, 
        this.mUseSystemDefault);
    return builder;
}
</pre>
3.EmojiconEditText的hint不能正常加上emoji。这个问题没有解决。

***

### taeSDK

#### 官网 
<http://baichuan.taobao.com/>

#### 简介  
淘宝的sdk。用来显示淘宝商品，下订单，和绑定淘宝账号。  

#### 踩过的坑  
1.调用sdk的页面需要重写onActivityResult，这个时候不能将自定义的requestCode设成1或者2，因为sdk需要用到。

***

### gpuimage

#### 官网 
<https://github.com/CyberAgent/android-gpuimage>

#### 简介  
滤镜 

***

### butterknife:7.0.1

#### 官网 
<https://github.com/JakeWharton/butterknife>

#### 简介  
简化findViewById  

***

### android-async-http:1.4.8

#### 官网 
<https://github.com/loopj/android-async-http>

#### 简介  
网络请求  

***

### gson:2.2.4

#### 官网 
<https://github.com/google/gson>

#### 简介  
解析json  

***

### leakcanary-android:1.3

#### 官网 
<https://github.com/square/leakcanary>

#### 简介  
内存泄露检测  

***

### ActiveAndroid

#### 官网 
<https://github.com/pardom/ActiveAndroid>

#### 简介  
数据库操作 

#### 踩过的坑  
1.select获取到的数据如果直接修改变量值，但不保存到数据库里，从其他地方调用select时修改后的值会被重置。 

***

### jpush

#### 官网 
<http://docs.jpush.io/>

#### 简介  
极光推送  

***

### umeng

#### 官网 
<https://www.umeng.com/>

#### 简介  
友盟统计  

***

### zxing

#### 官网 
<https://github.com/zxing/zxing>

#### 简介  
扫描二维码  

***

### zbar

#### 官网 
<https://github.com/ZBar/ZBar>

#### 简介  
扫描二维码  

***

### AMap

#### 官网 
<http://lbs.amap.com/>

#### 简介  
高德  

***

### retrolambda:3.2.0

#### 官网 
<https://github.com/evant/gradle-retrolambda>  

#### 简介  
使android代码支持lambda表达式  

#### 踩过的坑  
1.如果有drawable文件只在lambda表达式里出现过，那么lint会把这个文件标记成unused，因此在删除unused文件时需要检查是否真正使用过。相关issue： <https://github.com/evant/gradle-retrolambda/issues/69>

***

### RxJava:1.0.16  

#### 官网 
<https://github.com/ReactiveX/RxJava>  

***

### RxAndroid:1.0.1

#### 官网 
<https://github.com/ReactiveX/RxAndroid>    

#### 踩过的坑  
1.混淆后RxJava不能正常使用，加入了第三方库‘com.artemzin.rxjava:proguard-rules:1.0.16.2’之后才能正常使用。可能是gradle版本的问题。相关issue： <https://github.com/ReactiveX/RxAndroid/issues/219>  

***

### MaterialDialog:1.2.2  

#### 官网 
<https://github.com/drakeet/MaterialDialog>

#### 简介
显示Material Design的对话框。有修改。

***

### Fresco

#### 官网 
<https://github.com/facebook/fresco>

#### 简介  
加载图片的库

#### 踩过的坑  
1.加入fresco后在64位机型上会崩溃，解决办法：<https://github.com/facebook/fresco/issues/554>。关于so文件的说明：<http://www.jianshu.com/p/cb05698a1968>  
2.控件设置了BaseControllerListener之后，即使被设置成了GONE还是会调用onFinalImageSet。解决办法是将DraweeController置为空。  