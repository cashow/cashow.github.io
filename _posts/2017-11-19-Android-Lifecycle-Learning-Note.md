---
layout: post
title: "Android - lifecycle 库学习笔记"
date: 2017-11-19 14:33:33 +0800
tags: Android 学习笔记 lifecycle
---

lifecycle 库是2017年 Google I/O 开发者大会上发布的一个库，目的是让除了 Activity 和 Fragment 外的其他组件也能感知到生命周期。

------

### 引入

root目录的 build.gradle加上：

<pre>
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
</pre>

项目里的build.gradle加上：

<pre>
// 如果你引入了 lifecycle:extensions 或者 lifecycle:common-java8，
// 可以不引入 lifecycle:runtime
implementation "android.arch.lifecycle:runtime:1.0.3"

// 如果你引入了 lifecycle:common-java8 并且使用 DefaultLifecycleObserver，
// 可以不引入 lifecycle:compiler
annotationProcessor "android.arch.lifecycle:compiler:1.0.0"

// 如果需要支持 Java8 的组件，需要引入 lifecycle:common-java8
implementation "android.arch.lifecycle:common-java8:1.0.0"
</pre>

------

### 接收 lifecycle 的回调

<pre class="mcode">
// 如果你使用的是 Java 8, 那么只需实现 DefaultLifecycleObserver 接口。
// 为此你需要引入 lifecycle:common-java8 库
public class LifecycleObserverDemo implements DefaultLifecycleObserver {
    // 在这里注册回调
    public void setLifecycle(Lifecycle lifecycle) {
        lifecycle.addObserver(this);
    }

    @Override
    public void onCreate(@NonNull LifecycleOwner owner) {
    }

    @Override
    public void onStart(@NonNull LifecycleOwner owner) {
    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {
    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {
    }

    @Override
    public void onStop(@NonNull LifecycleOwner owner) {
    }

    @Override
    public void onDestroy(@NonNull LifecycleOwner owner) {
    }
}
// 如果你使用的是 Java 7, 需要实现 LifecycleObserver 接口，
// 并且通过注解将你的函数和生命周期的回调进行绑定.
public class LifecycleObserverDemo implements LifecycleObserver {
    // 在这里注册回调
    public void setLifecycle(Lifecycle lifecycle) {
        lifecycle.addObserver(this);
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    void create() {
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void start() {
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    void resume() {
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    void pause() {
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void stop() {
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    void destroy() {
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    void any() {
    }
}
</pre>

------

### 注册 lifecycle 的回调

在 Support 库 v26.1.0 及以后的版本，Fragments 和 Activities 会实现 LifecycleOwner 接口，只需要在 Activity 或者 Fragment 里加入以下代码即可注册 lifecycle 的回调：

<pre class="mcode">
lifecycleObserverDemo.setLifecycle(getLifecycle());
</pre>

如果你的 Support 库版本号低于 26.1.0，或者你的自定义类想要实现 LifecycleOwner，你需要使用到 LifecycleRegistry，并且自己处理每个生命周期。

<pre class="mcode">
public class MainActivity extends AppCompatActivity implements LifecycleOwner {
    private LifecycleObserverDemo lifecycleObserverDemo;
    private LifecycleRegistry mLifecycleRegistry;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mLifecycleRegistry = new LifecycleRegistry(this);
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);

        lifecycleObserverDemo = new LifecycleObserverDemo();
        lifecycleObserverDemo.setLifecycle(getLifecycle());
    }

    @Override
    public void onStart() {
        super.onStart();
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_START);
    }

    @Override
    protected void onResume() {
        super.onResume();
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_RESUME);
    }

    @Override
    protected void onPause() {
        super.onPause();
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_PAUSE);
    }

    @Override
    protected void onStop() {
        super.onStop();
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_DESTROY);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
}
</pre>

-----

### lifecycle 库的实现原理

在 Activity onCreate() 时，Activity 会自动添加一个 ReportFragment。ReportFragment 监听到了生命周期事件后，再传递给 LifecycleRegistry。向 Activity 注册一个无 UI 的 Fragment 用于将各种 Activity 回调分离出来是个常用的做法。

### 回调顺序

ON_CREATE, ON_START, ON_RESUME 发生在 LifecycleOwner 相关的生命周期结束后。

ON_PAUSE, ON_STOP, ON_DESTROY 发生在 LifecycleOwner 相关的生命周期被调用之前。

Lifecycle.State 还提供了一个 isAtLeast() 函数，用来判断当前的生命周期是否已经进行到了某个阶段。总共有以下几种值（以下值是递增的）：

<pre class="mcode">
// Activity 的 onDestroy() 已被调用
DESTROYED

// Lifecycle 已经初始化但没有接受到 onCreate() 的回调
INITIALIZED

// Activity 的 onCreate() 已被调用，但 onStop() 没被调用
CREATED

// Activity 的 onStart() 已被调用，但 onPause() 没被调用
STARTED

// Activity 的 onResume() 已被调用
RESUMED
</pre>

------

### 已知问题

由于 FragmentLifecycleCallbacks 只存在于 v4 包中，所以当 LifecycleRegistryOwner 是 android.app.Fragment 时，该 Fragment 相关生命周期发生变化也不会主动产生相关的 lifecycle 事件，你也接收不到任何的回调。

这个问题已经被提在官方的 issue 上了：

<https://issuetracker.google.com/issues/62160522>

这是 google 工作人员的回复：

Yeah, we don't currently target platform Activity / Fragments.  
是的，我们开发时并没有针对原生的 Activity 和 Fragments。

I'm keeping this open, but we are not looking to fix this in the nearest future.  
这个 issue 不会关闭，但是我们在近期也不打算修复这个问题。

PS for people who uses platform fragments: please consider the usage support library fragments. When you use platform fragments, it means that your code depends on multiple years old implementation of fragments, which is shipped with a device. Those implementation are usually very old and has a lot of bugs that are already fixed in support library.

PS：对于那些还在使用原生的 Fragments 的开发者，请考虑使用 support 库里的 Fragments。如果你使用原生的 Fragments，这也意味着你的代码依赖于多年来各种各样的 Fragments 的实现，这些 Fragments 的代码通常都比较陈旧，并且带有大量已在 support 库里修复好的bug。

------

### 相关链接

<https://developer.android.com/topic/libraries/architecture/lifecycle.html>

<https://juejin.im/entry/592556538d6d810058034d5f>

<http://chaosleong.github.io/2017/05/27/How-Lifecycle-aware-Components-actually-works/>

### 项目链接

<https://github.com/cashow/AndroidTricks/tree/master/LifeCycleDemo>
