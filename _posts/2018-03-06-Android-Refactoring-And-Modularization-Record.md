---
layout: post
title: "Android - 项目重构及组件化的记录"
date: 2018-03-06 23:14:02 +0800
tags: Android 设计模式 组件化
---

近半年内，我对我们的项目进行过多次大大小小的重构，并在近期实现了组件化，在此想把一些我认为有意义的调整记录下来。

由于我负责的项目使用了 [Clean 架构](https://github.com/android10/Android-CleanArchitecture)，所以我就直接拿 Clean 架构 demo 的代码来做说明。不了解 Clean 架构也没关系，这篇文章不会涉及到 Clean 架构的具体内容，你可以在看的过程中把 Data 层和 Domain 层忽略掉。

这是这篇文章 demo 项目的链接：<https://github.com/cashow/ModularizationDemo>

<div class="alert alert-success" role="alert">这篇文章里提到过的项目调整都在 demo 项目里通过 pull request 的形式实现了，每个 pull request 对应一次项目调整的内容：<a href="https://github.com/cashow/ModularizationDemo/pulls?q=is%3Apr+is%3Aclosed">点击查看 pull requests</a></div>

---

### 初始状态

这个 demo 项目总共有 3 个页面：主页（MainActivity），用户列表页（UserListActivity），用户详情页（UserDetailsActivity）。

在主页有按钮点击后可以进到用户列表页，用户列表页点击用户栏可以进到用户详情页。

这就是这个 demo 所有的功能了，就这么简单。

<div class="alert alert-success" role="alert">这是项目初始状态的代码：<a href="https://github.com/cashow/ModularizationDemo/tree/157f3f4390f4e8c6f2ddfc21630f211872426c60">点击查看链接</a></div>

---

### 使用 MVP 改写复杂页面

每个项目都不可避免的会遇到一些复杂的页面，有时候我花大量时间完成一个页面要实现的功能后，会发现这个 Activity 或者 Fragment 已经有 1000+ 行数的代码了。使用 MVP 模式，将 Activity 的 Model、View、Presenter 抽离出来，能让 Activity 只保留 View 相关的逻辑（生命周期、点击事件等），从而变得特别简洁。

我在改造过程中遵循的规则是，View 层不保留任何的 Model。Presenter 通过异步操作获取到 Model 后，将 Model 保留到 Presenter 里。View 只保留最基本的 UI 相关的操作，例如点击事件、触摸事件和页面跳转等。

例如，在你调用完一个接口后要根据接口的返回值前往不同的页面，正常情况下这个逻辑会写在接口的回调里，如果你想找到当前页面会跳转到哪些页面去，你就需要去这个回调里找，而这个回调里可能还会掺杂着其他的处理逻辑。使用 MVP 后，View 会提供跳转到各个页面的接口，由于 Activity 要实现 View 定义的接口，所以 View 里所有和页面跳转相关的方法都会平铺到 Activity 类里，不再需要去各个方法和回调里找。经过这样的改造后，你可以很方便地看到这个 View 里有哪些跳转到其他页面的操作。

对 MVP 不了解的可以去看这几篇博客：

[Android MVP 详解](https://www.jianshu.com/p/9a6845b26856)  
[MVP模式在Android开发中的应用](http://blog.csdn.net/vector_yi/article/details/24719873)  
[Android中的MVP](https://rocko.xyz/2015/02/06/Android%E4%B8%AD%E7%9A%84MVP/)  
[浅谈 MVP in Android](http://blog.csdn.net/lmj623565791/article/details/46596109)

<div class="alert alert-success" role="alert">demo 项目 MVP 的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/1/files">点击查看链接</a></div>

---

### 将 java 文件夹按模块拆分

项目里原本是按代码类别去拆分 java 文件夹的，例如所有的 Activity 放到 activity 文件夹中，所有的 Listener 放到 listener 文件夹中，如图所示：

![modularization_demo_ori_java](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/modularization_demo_ori_java.png)

这么做有个缺点是，同一个模块的代码分散在各个文件夹下，当你要移除某个模块的功能，或者将某个模块的代码抽取出来放到另一个项目里，你需要去各个文件夹把这个模块相关的代码提取出来。

在经历过一次提取模块代码到新项目的惨痛经历后，我决定把 java 文件夹按模块整理，模块独有的文件放到模块的文件夹下，各个模块共用的文件放到 common 文件夹下，如果所示：

![modularization_demo_new_java](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/modularization_demo_new_java.png)

<div class="alert alert-success" role="alert">demo 项目拆分 java 文件夹的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/2/files">点击查看链接</a></div>

---

### 将 res 文件夹按模块拆分

在将 java 文件夹按模块拆分后，我把 res 文件夹也按模块拆分开了。

拆分前：

![modularization_demo_old_res](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/modularization_demo_old_res.png)

拆分后：

![modularization_demo_new_res](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/modularization_demo_new_res.png)

要实现这样的拆分，你需要修改 app/build.gradle 文件：

<pre>
def listSubFile = {
    def resFolder = 'src/main/res'
    def files = file(resFolder).listFiles()
    def folders = []
    files.each {
        item -> folders.add(item.absolutePath)
    }
    folders.add(file(resFolder))
    return folders
}
sourceSets {
    main {
        jniLibs.srcDirs = ['libs']
        res.srcDirs += listSubFile()
    }
}
</pre>

上面这段代码的作用是，将 res 文件夹下的所有子文件夹取出，并设为资源文件夹。通过这个配置，你就能在 res 文件夹下加各个模块的子文件夹。

需要注意的是，有时候在 res 文件夹添加新的子文件夹时，Android Studio 不会将新文件夹视为资源文件夹，这时你需要将上面的代码注释掉，Sync 一遍，取消注释后再 Sync 一遍即可。

<div class="alert alert-success" role="alert">demo 项目拆分 res 文件夹的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/3/files">点击查看链接</a></div>

---

### 将每个 Module 的引用库放到一个文件里进行管理

目前的项目分了 3 层：App 层、Data 层和 Domain 层，每一层的 build.gradle 都配置了自己的引用库。

这是 App 层的 build.gradle：

![app_dependencies](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/app_dependencies.png)

这是 Data 层的 build.gradle：

![data_dependencies](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/data_dependencies.png)

这是 Domain 层的 build.gradle：

![domain_dependencies](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/domain_dependencies.png)

可以看到，如果你要给 App 层添加或者删除引用库，你需要先定位到 app/build.gradle 文件，再进行修改，其他层同理。当你的项目里拆分了多个 Module 时，修改各个 Module 的引用库也是件麻烦的事。

为此，我将每个 Module 的引用库整理到了 root 文件夹下的 dependencies.gradle 文件里：

![all_dependencies](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/all_dependencies.png)

修改后的 app/build.gradle：

![app_new_dependencies](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/app_new_dependencies.png)

修改后的 data/build.gradle：

![data_new_dependencies](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/data_new_dependencies.png)

修改后的 domain/build.gradle：

![domain_new_dependencies](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/domain_new_dependencies.png)

以后如果要修改每个 Module 引用的库，只需要进入 dependencies.gradle 修改对应的引用库即可。

<div class="alert alert-success" role="alert">demo 项目整理引用库的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/4/files">点击查看链接</a></div>

---

### 组件化

接下来要做的就是组件化。组件化的目的是将各个模块间的代码彻底解耦，并实现各模块独立运行，缺少某个模块并不会影响其他模块的测试和开发。在这篇文章里，我们暂且将主页、用户列表页和用户详情页当做 3 个模块去处理。

这是组件化之前各个模块的依赖关系：

![ori_module](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/ori_module.jpg?imageView2/2/w/400)

可以看到，在实现组件化之前，App 层共有 main、userdetails、userlist 和 common 四个模块的代码，并且这四个模块的代码可以相互调用。

为了实现组件化，需要做的事有：

1.将各个模块的代码抽取出来，放到该模块对应的 Module 里。各个模块的 Module 互相不能有依赖关系（在这个例子里是 userdetails 和 userlist）；  
2.将共用的代码抽取出来，放到一个 Module 里，所有的模块依赖于这个 Module（在这个例子里是 common）；  
3.创建一个 Module 当做 App 的入口，在这个模块里做闪屏页、引导页等入口类型的功能，并实现 Application 的初始化操作。这个 Module 依赖所有模块的 Module（在这个例子里是 main，依赖于 userdetails 和 userlist）。

这是实现了组件化后各个模块的依赖关系：

![new_module](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/new_module.jpg?imageView2/2/w/400)

可以看到，在实现了组件化后，userdetails 和 userlist 之间没有依赖关系，因此不能相互调用代码，实现了模块间的解耦。

---

### 使用 ARouter 将不同模块的 Activity 解耦

在把各个模块的代码整合到不同的 Module 后，模块之间并没有依赖关系，不能直接通过 startActivity() 去启用其他模块的 Activity，为此我引入了 [ARouter](https://github.com/alibaba/ARouter)。

改动前，跳转到 UserListActivity 的代码：

![navigate_ori](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/navigate_ori.png)

引入 ARouter 后给 UserListActivity 加上 @Route 注解：

![user_list_activity](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/user_list_activity.png)

改动后，跳转到 UserListActivity 的代码：

![navigate_new](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/navigate_new.png)

<div class="alert alert-success" role="alert">demo 项目引入 ARouter 的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/5/files">点击查看链接</a></div>

---

### 将模块公用的代码整理到一个 Module 里

由于各模块准备分拆，我需要把不同模块之间共用的代码和资源文件整理出来。

这个过程是个比较痛苦的过程，因为有些模块共用的代码可能会和模块独有的代码放在一个文件里，有些代码则要往上查好几层依赖关系才能确定有没有被其他模块引用，不过当你看到一行行代码从你模块里移除掉，模块里的代码变得异常简洁时，你会觉得这一切都是值得的。

<div class="alert alert-success" role="alert">demo 项目整理共用代码的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/6/files">点击查看链接</a></div>

---

### 将每个模块的代码移入对应的 Module 中

现在该开始做模块分拆了。我需要把每个模块的代码和资源文件移入对应的 Module 中，这个过程中踩到了几个坑，其中最典型的坑就是 ButterKnife。

在 Module 里如果要使用 ButterKnife，你需要在 Module 的 build.gradle 里加上 `apply plugin: 'com.jakewharton.butterknife'`，并在代码里将 R.id.xxx 改成 R2.id.xxx。

在迁移过程中，我还遇到了 `Unable to find method 'com.android.build.gradle.api.BaseVariant.getOutputs()Ljava/util/List;'` 问题，并定位到了对应的 issue ：[点击查看链接](https://github.com/JakeWharton/butterknife/issues/1130)。按 issue 的建议使用 9.0.0-SNAPSHOT 版本后就可以正常编译了。

据说 DataBinding 的代码移入 Module 时也有坑，由于项目中没有用到，这个坑就不清楚了，有用到的可以自己去查下资料。

<div class="alert alert-success" role="alert">demo 项目拆分模块代码的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/7/files">点击查看链接</a></div>

---

### 每个模块独立运行

现在要实现的是，让每个模块可以独立运行，例如将 userdetails 模块当作独立的 Application 进行开发和测试。这样的话可以在开发过程中避免其他模块的错误代码影响到你负责模块的开发，并且由于不需要引入其他的模块，开发过程中项目的编译时间会缩短很多。

首先，是在 Module 的 gradle.properties 文件（如果没有就创建一个）下添加 `isRunAsApp=false` 代码，并在 build.gradle 文件下添加：

<pre>
if (isRunAsApp.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
</pre>

上面代码的作用是，当 gradle.properties 里的 isRunAsApp 等于 true 时，将这个 Module 当作 Application 运行，等于 false 时当作 Library 运行。

也就是说，如果我们要单独测试这个模块，将 isRunAsApp 设成 true 并在项目的 settings.gradle 把其他模块注释掉，这个模块就能编译成 apk 运行到手机上。

当然了，这么改肯定不够，还需要修改其他地方，比如需要在 AndroidManifest.xml 里指定 Application、权限和入口 Activity。由于这些信息只在 debug 时有用，编译整个项目时不需要这些配置，因此你需要创建一个 debug 专用的 AndroidManifest.xml 文件，并在 build.gradle 里指定测试专用的文件。

<pre>
sourceSets {
    main {
        if (isRunAsApp.toBoolean()) {
            manifest.srcFile 'src/mdebug/AndroidManifest.xml'
        }
    }
}
</pre>

上面代码的作用是，在 isRunAsApp 等于 true 时，使用 src/mdebug/AndroidManifest.xml 文件当做项目的 AndroidManifest.xml。在这个文件里，你可以指定测试用的 Application、权限和入口 Activity。

<div class="alert alert-success" role="alert">demo 项目实现各模块独立运行的改动记录：<a href="https://github.com/cashow/ModularizationDemo/pull/8/files">点击查看链接</a></div>

---

### demo 项目链接

<https://github.com/cashow/ModularizationDemo>
