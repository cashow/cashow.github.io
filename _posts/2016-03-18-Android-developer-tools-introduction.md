---
layout: post
title: "Android - 我使用过的第三方库以及踩过的坑"
date: 2016-03-18 00:07:42 +0800
tags: Android Android开发工具 学习笔记 原创
---

做android开发3年了，接触到了不少第三方库，每一个新接触到的工具都能避免重复造轮子，加快开发进度。然而在使用这些工具的过程中，我也踩过不少的坑。因此特意写篇博客简单介绍下各个库的功能，以及怎么避免踩坑。  

***

### weiboSDK

####  官网  
<https://github.com/sinaweibosdk/weibo_android_sdk>  

####  简介  
新浪微博的sdk。分享微博以及微博登陆需要用到。  

####  踩过的坑  
1.旧版本的问题：导入sdk后，在gradle编译时报错，提示“Multiple dex files define Lcom/sina/weibo/sdk/BuildConfig”。报错原因是weibosdkcore.jar里面有BuildConfig.class，这个文件本不应该出现在这里的。解决办法是将weibosdkcore.jar解开，删掉weibosdkcore.jar后重新生成jar。  
2.旧版本的问题：友盟上看到有报错：“Service Intent must be explicit: Intent { act=com.sina.weibo.remotessoservice }”，原因是android 5.0开始，service必须显性声明。如果有人遇到这种问题将微博sdk升级到最新版本就可以了。

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
这样的代码会导致内存泄露，原因是代码中使用了匿名内部类，非静态内部类（包括匿名内部类）会持有一个隐式的外部类引用，在这里是ImageLoadingListener引用了外部的activity。如果在图片没有完全加载的情况下退出activity，由于匿名内部类还持有对activity的引用，导致activity不能正确地销毁，这就造成了内存泄露。以下两种办法可以避免内存泄露：  
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
在安卓手机上显示和输入emoji表情的库。  

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

### android-gpuimage

#### 官网
<https://github.com/CyberAgent/android-gpuimage>

#### 简介  
实现图片滤镜效果的库。

***

### butterknife

#### 官网
<https://github.com/JakeWharton/butterknife>

#### 简介  
简化findViewById，通过这个库可以明显减少代码行数，使代码更简洁易读，样例如下：  
<pre class="mcode">
class ExampleActivity extends Activity {
  @Bind(R.id.title)
  TextView title;
  @Bind(R.id.subtitle)
  TextView subtitle;
  @Bind(R.id.footer)
  TextView footer;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
  }
}
</pre>  
另外，Android Studio有个插件叫Android Butterknife Zelezny，支持一键从布局文件中生成对应的控件声明和Bind注解，非常方便。

***

### android-async-http

#### 官网
<https://github.com/loopj/android-async-http>

#### 简介  
网络请求的库  

***

### gson

#### 官网
<https://github.com/google/gson>

#### 简介  
支持json字符串和java对象相互转换的库。类似的库有阿里的[fastjson](https://github.com/alibaba/fastjson)和国外团队的[jackson](https://github.com/FasterXML/jackson)，其中[fastjson](https://github.com/alibaba/fastjson)的官方wiki里有几个json解析库的比较：[各种JSON库的比较](https://github.com/alibaba/fastjson/wiki/%E5%90%84%E7%A7%8DJSON%E5%BA%93%E7%9A%84%E6%AF%94%E8%BE%83)，仅供参考。  

***

### leakcanary

#### 官网
<https://github.com/square/leakcanary>

#### 简介  
用来检测内存泄露的库，相关的讲解可见：[Android - 利用LeakCanary检测内存泄露](http://cashow.github.io/android-detect-out-of-memory-with-leakcanary.html)  

***

### ActiveAndroid

#### 官网
<https://github.com/pardom/ActiveAndroid>

#### 简介  
简化数据库操作的库，样例如下：
<pre class="mcode">
// java的Category类对应数据库的Categories表
@Table(name = "Categories")
public class Category extends Model {
    @Column(name = "Name")
    public String name;
}

// 将数据插入到数据库
Category restaurants = new Category();
restaurants.name = "Restaurants";
restaurants.save();

// 在数据库查询数据
public static List&lt;Item&gt; getAll(Category category) {
    return new Select()
        .from(Item.class)
        .where("Category = ?", category.getId())
        .orderBy("Name ASC")
        .execute();
}
</pre>  

#### 踩过的坑  
通过Select()获取到的数据假如只在内存里修改了变量值，不保存到数据库里，在其他地方调用Select()时如果也查询到了刚刚获取的数据，那么被修改的变量值会被重置，如下所示：
<pre class="mcode">
private List&lt;TestObject&gt; testList;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    test();
}

private void test(){
    testList = new Select().from(TestObject.class).execute();
    print("log 0");
    for (int i = 0; i &lt; testList.size(); i++) {
        TestObject object = testList.get(i);
        object.name += " hello";
    }
    print("log 1");
    int count = new Select().from(TestObject.class).execute().size();
    print("log 2");
}

private void print(String text){
    for (int i = 0; i &lt; testList.size(); i++) {
        Log.d("test", text + " : " + testList.get(i).name);
    }
}

// 打印出来的日志是：
// log 0 : name1
// log 0 : name2
// log 0 : name3
// log 1 : name1 hello
// log 1 : name2 hello
// log 1 : name3 hello
// log 2 : name1
// log 2 : name2
// log 2 : name3
</pre>  
目前能想到的解决办法就是在修改变量值后立即将数据存入到数据库里。

***

### jpush

#### 官网
<http://docs.jpush.io/>

#### 简介  
极光推送的sdk，可用来推送系统消息和通知。  

***

### umeng

#### 官网
<https://www.umeng.com/>

#### 简介  
友盟统计的sdk。友盟会在app出现崩溃的时候自动上传崩溃日志到后台，通过友盟的错误分析功能可以看到崩溃发生的地方以及崩溃日志。通过分析这些日志可以了解到哪部分代码有问题。如果代码被混淆过，还可以上传mapping文件到友盟后台，这样看到的崩溃日志就是没有混淆过的。  

***

### QrCodeScan

#### 官网
<https://github.com/SkillCollege/QrCodeScan>

#### 简介  
扫描二维码的库，结合了ZXing和ZBar的优点，使用ZXing来控制摄像头取得图像，使用ZBar来解析扫描到的数据。  

#### 踩过的坑
有时候虽然图片在扫描框内但还是扫不出来，手机离远点就能扫出来。不确定是这个库的问题还是我自己没有处理好。

***

### AMap

#### 官网
<http://lbs.amap.com/>

#### 简介  
高德定位和高德地图的sdk。  

#### 踩过的坑
在刚开始做android开发的时候，年少无知的我竟然使用android系统自带的api进行gps定位，虽然我测试用的小米手机定位没有什么问题，但总有用户反应定位不准或者不能定位，我也说不清楚是国内各式各样ROM的锅还是android系统定位的api本身在国内就不好使，于是我改用高德的sdk进行定位，再也没有用户来抱怨了，在此对高德地图致以最诚挚的感谢！至于为什么不用百度地图的sdk，呵呵，我对[百度](https://www.zhihu.com/question/33267404)没有好感！！！

***

### retrolambda

#### 官网
<https://github.com/evant/gradle-retrolambda>  

#### 简介  
使android代码支持lambda表达式  

#### 踩过的坑  
如果有资源文件只在lambda表达式里被调用，那么lint会把这个文件标记成unused，因此在使用lint工具查找未被使用的文件时需要确认下这个资源文件是否真正使用过。相关issue：[Unused resources when used in Lambda expressions](https://github.com/evant/gradle-retrolambda/issues/69>)

***

### RxJava 和 RxAndroid

#### 官网
<https://github.com/ReactiveX/RxJava>  

<https://github.com/ReactiveX/RxAndroid>   

#### 简介
相关资料可见：[Android - RxJava学习笔记](http://cashow.github.io/Android-rxjava-notes.html)  

#### 踩过的坑  
混淆后RxJava不能正常使用，加入了第三方库[RxJavaProGuardRules](https://github.com/artem-zinnatullin/RxJavaProGuardRules)后才能正常使用。相关issue： <https://github.com/ReactiveX/RxAndroid/issues/219>  

***

### MaterialDialog  

#### 官网
<https://github.com/drakeet/MaterialDialog>

#### 简介
显示Material Design的对话框。

#### 踩过的坑  
旧版本在设置了setCanceledOnTouchOutside(false)后，按返回键还是可以取消掉这个对话框。新版本已经修复了这个问题。

***

### Fresco

#### 官网
<https://github.com/facebook/fresco>

#### 简介  
facebook出的加载图片的库。fresco的最大特点在于，图片不在Java Heap上分配内存。对于图片较多的app，尤其是社交类app，使用fresco可以大大地降低app的内存消耗。fresco支持直接在xml文件里设置图片的圆角和加载时的占位图。fresco还支持图片加载失败后点击自动重新加载。更多的功能可参见官方文档。目前我正在把项目中大部分加载图片的代码改用fresco实现。

#### 踩过的坑  
1.加入fresco后在64位机型上会崩溃，解决办法：<https://github.com/facebook/fresco/issues/554>。关于so文件的说明可见：<http://www.jianshu.com/p/cb05698a1968>  
2.在listview里，控件设置了BaseControllerListener之后，如果父控件被设置成了GONE还是会调用onFinalImageSet。解决办法是将DraweeController置为空。

***

以上就是我所用过的第三方库。为了避免在使用新的第三方库时不小心踩进坑里，建议大家先认真看看各个库在github上的issues。一般比较常见的bug都会有相应的issue，所以如果在使用第三方库时先了解这些库目前存在的问题，会更好地帮助你决定是否要引入这个库。
