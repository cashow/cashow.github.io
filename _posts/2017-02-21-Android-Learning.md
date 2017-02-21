---
layout: post
title: "Android - 进阶学习资料整理"
date: 2017-02-21 23:06:23 +0800
tags: Android
---

在为公司新招了2个android工程师后，我开始转做后端开发，并把android项目全权交给他们俩去负责。为了让他们在开发过程中能高效地完成高质量的代码，我整理了一份android进阶的学习资料。虽说是进阶学习资料，但其实大部分都是性能优化方面的文章。

----------------------

### Android Training
链接：<https://developer.android.com/training/index.html>

在我面试android工程师的时候发现，很多人上android官网都是为了查api文档，很少有人注意到官网还有个Android Training板块。这个板块是官方的培训教程，为你讲解从android基础到性能优化的各个方面内容。虽说是培训教程，但我还是建议有一定android基础的同学再去看，因为这个教程并不是从0开始讲解，有些内容需要一定的android基础才能看懂。

假如你是通过网上的视频或者博客开始学习android基础知识，想要系统地补习一遍android知识，那我强烈建议你再从头到尾看一遍Android Training板块。在这里你可以看到常用功能的标准写法，比如加载fragment、执行后台任务、高效加载大图。将官网上各个功能的写法和你自己的代码进行对比，你可能会找到自己代码中缺失的一些判断条件，而这些缺失的代码可能会导致一些意想不到的问题。

如果你还在犹豫，请看下面的代码（这也是我的一道面试题），并指出以下代码有什么问题。假如你看不出来，请把Android Training板块再刷一遍。

<pre class='mcode'>
public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);

        if (findViewById(R.id.fragment_container) != null) {
            HeadlinesFragment firstFragment = new HeadlinesFragment();

            getSupportFragmentManager().beginTransaction()
                    .add(R.id.fragment_container, firstFragment).commit();
        }
    }
}
</pre>

ps：Android Training板块已经有人在做中文版了，想看中文版的同学请自行搜索。翻译的进度肯定比不上官网更新速度，所以有能力阅读英文的同学最好还是阅读原版。

----------------------------

### Android Developers Blog
链接：<https://android-developers.googleblog.com/>

这是android官方的技术博客，在这里可以看到android每个版本的更新说明。这个博客偶尔还会发布些优化方面的教程。建议经常关注下这个博客里android版本更新相关的内容，以避免新系统上线后出现自己家的app不兼容的问题。

----------------------------

### 胡凯的性能优化系列
先献上胡凯大神的博客：<http://hukai.me/>

以下是YouTube上android开发团队出的[Android Performance](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)系列教程。点开看看就知道，满满的都是干货。

[Android性能优化典范 - 第1季](http://hukai.me/android-performance-patterns/)

[Android性能优化典范 - 第2季](http://hukai.me/android-performance-patterns-season-2/)

[Android性能优化典范 - 第3季](http://hukai.me/android-performance-patterns-season-3/)

[Android性能优化典范 - 第4季](http://hukai.me/android-performance-patterns-season-4/)

[Android性能优化典范 - 第5季](http://hukai.me/android-performance-patterns-season-5/)

以下是android开发团队出的[Android Performance Patterns](https://www.udacity.com/course/ud825)系列教程。同样的，都是干货。

[Android性能优化之渲染篇](http://hukai.me/android-performance-render/)

[Android性能优化之运算篇](http://hukai.me/android-performance-compute/)

[Android性能优化之内存篇](http://hukai.me/android-performance-memory/)

[Android性能优化之电量篇](http://hukai.me/android-performance-battery/)

除了上述两个系列外，还有一些文章要重点推荐下：

[Android内存优化之OOM](http://hukai.me/android-performance-oom/)

[Android开发最佳实践](http://hukai.me/android-dev-patterns/)

[Android Training - 管理应用的内存](http://hukai.me/android-training-managing_your_app_memory/)

[Android Training - 提升布局文件的性能(Lesson 1 - 优化布局的层级)](http://hukai.me/android-training-improve-layouts-lesson-1/)

[Android Training - 提升布局文件的性能(Lesson 2 - 使用include标签重用Layout)](http://hukai.me/android-training-improve-layouts-lesson-2/)

[Android Training - 提升布局文件的性能(Lesson 3 - 使用viewStub按需载入视图)](http://hukai.me/android-training-improve-layouts-lesson-3/)

[Android Training - 提升布局文件的性能(Lesson 4 - 使用ViewHolder来提升ListView的性能)](http://hukai.me/android-training-improve-layouts-lesson-4/)

----------------------------

### Android Weekly
链接：<http://androidweekly.net/>

Android Weekly，即android开发技术周报，每周都会推荐一些精选的技术文章。如果你想了解现在android最流行的技术，找近期新出的性能优化教程，可以从这里入手。

Github上有个开源项目翻译了不少Android Weekly的文章，也顺带推荐一下：

<https://github.com/hehonghui/android-tech-frontier>

----------------------------

### 内存泄露分析
[Android内存溢出分析](http://blog.tisa007.com/tech/android_memory_management_and_solve_oom_problem.html)

[Wrangling Dalvik: Memory Management in Android (Part 1 of 2)](https://www.raizlabs.com/dev/2014/03/wrangling-dalvik-memory-management-in-android-part-1-of-2/)

[Hunting Your Leaks: Memory Management in Android (Part 2 of 2)](https://www.raizlabs.com/dev/2014/04/hunting-your-leaks-memory-management-in-android-part-2-of-2/)

----------------------------

### 一些零碎的文章
知乎上总结的开发者选项的功能：

<https://www.zhihu.com/question/29967530>

知乎上总结的android开发习惯：

<https://www.zhihu.com/question/27227425>


Android开发经验总结：

<http://zmywly8866.github.io/2015/12/12/android-develop-experencies.html>

Futurice公司总结的Android开发最佳实践：

<https://github.com/futurice/android-best-practices/blob/master/translations/Chinese/README.cn.md>

Romain Guy大神写的文章，教你不看代码检测性能问题：

<http://www.curious-creature.com/2012/12/01/android-performance-case-study/>

不看代码检测性能问题第二集：

<http://www.curious-creature.com/2015/03/25/android-performance-case-study-follow-up/>

加快app启动速度：

<http://www.curious-creature.com/2009/03/04/speed-up-your-android-ui/>

加快app响应速度：

<http://blog.udinic.com/2015/09/15/speed-up-your-app>

严格模式（Strict Mode）：

<https://developer.android.com/reference/android/os/StrictMode.html>

----------------------------

### 代码规范
以下是一些android和java的代码规范，仅供参考。

#### android官方的代码规范：
<https://source.android.com/source/code-style.html>

#### Google的java代码规范：
<https://google.github.io/styleguide/javaguide.html>

#### 阿里巴巴java开发手册：
<https://102.alibaba.com/newsInfo.htm?newsId=6>
