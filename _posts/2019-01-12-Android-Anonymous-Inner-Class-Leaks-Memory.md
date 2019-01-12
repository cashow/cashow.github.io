---
layout: post
title: "Android - 匿名内部类导致内存泄露的处理办法"
date: 2019-01-12 21:47:46 +0800
tags: Android 内存泄露
---

在使用各种 listener 时，稍有不注意就会导致内存泄露，因此在使用延时返回的回调时，需要格外小心。

---

## demo

先看一个的 demo：MainActivity 点击按钮后，调用 LongTimeOperation 开启一个耗时 10 秒的线程，并在执行完成后调用回调 onLongTimeCallback()。

MainActivity 中开启耗时操作的代码：

<pre class='mcode'>
private void startLongTimeOperation() {
  LongTimeOperation longTimeOperation = new LongTimeOperation();
  LongTimeOperation.LongTimeCallback longTimeCallback = new LongTimeOperation.LongTimeCallback() {
    @Override
    public void onLongTimeCallback() {
      text.setText("onLongTimeCallback");
    }
  };
  longTimeOperation.setLongTimeCallback(longTimeCallback);
  longTimeOperation.start();
}
</pre>

LongTimeOperation 中耗时操作的代码：

<pre class='mcode'>
public void start() {
  new Thread(() -> {
    try {
      Thread.sleep(10000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    mMainHandler.post(() -> {
      if (longTimeCallback != null) {
        longTimeCallback.onLongTimeCallback();
      }
    });
  }).start();
}
</pre>

在以上的代码中，由于 longTimeCallback 是匿名内部类，持有了外部的引用，在这个例子里持有了 MainActivity 的引用。

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_1.png)

因此，在 MainActivity onDestory() 后，由于 longTimeOperation 持有 longTimeCallback 的引用，而 longTimeCallback 持有了 MainActivity 的引用，导致 GC 的时候无法回收 MainActivity 实例，造成了内存泄露。

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_2.png)

---

## 改进方案一：将回调中所有对象放入 WeakReference 中

为了避免匿名内部类引用 Activity 的问题，可以把回调里的操作单独提出来，在这个例子中就是回调中需要用到的 TextView。

改动过后，longTimeOperation 引用的是 weakLongTimeCallback，weakLongTimeCallback 中的 TextView 是在 WeakReference 中。因此当 MainActivity 被回收时，textView 也能被回收，避免了内存泄露。

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_3.png)

<pre class='mcode'>
private void startLongTimeOperation() {
  LongTimeOperation longTimeOperation = new LongTimeOperation();
  longTimeOperation.setLongTimeCallback(new WeakLongTimeCallback(text));
  longTimeOperation.start();
}

static class WeakLongTimeCallback implements LongTimeOperation.LongTimeCallback {
  private WeakReference&lt;TextView&gt; textViewWeakReference;

  public WeakLongTimeCallback(TextView textView) {
    this.textViewWeakReference = new WeakReference&lt;&gt;(textView);
  }

  @Override
  public void onLongTimeCallback() {
    if (textViewWeakReference != null && textViewWeakReference.get() != null) {
      TextView textView = textViewWeakReference.get();
      textView.setText("onLongTimeCallback");
    }
  }
}
</pre>

### 缺点一

这个改进方案的缺点是，对于现有代码的改动量大。如果回调中使用到了多个 View，需要声明多个 WeakReference。

<pre class='mcode'>
static class WeakLongTimeCallback implements LongTimeOperation.LongTimeCallback {
  private WeakReference&lt;TextView&gt; textViewWeakReference1;
  private WeakReference&lt;TextView&gt; textViewWeakReference2;
  private WeakReference&lt;TextView&gt; textViewWeakReference3;
  // ...
}
</pre>

### 缺点二

所有 WeakReference 对象的生命周期需要和 Activity 绑定，否则当 WeakReference 的对象不再有强引用时，会在 GC 时被回收掉。

demo 如下：

<pre class='mcode'>
private void startLongTimeOperation() {
  TestObject testObject = new TestObject();
  testObject.setCreateTime(System.currentTimeMillis());

  LongTimeOperation longTimeOperation = new LongTimeOperation();
  longTimeOperation.setLongTimeCallback(new WeakLongTimeCallback(text, testObject));
  longTimeOperation.start();
}

static class WeakLongTimeCallback implements LongTimeOperation.LongTimeCallback {
  private WeakReference&lt;TextView&gt; textViewWeakReference;
  private WeakReference&lt;TestObject&gt; testObjectWeakReference;

  public WeakLongTimeCallback(TextView textView, TestObject testObject) {
    this.textViewWeakReference = new WeakReference&lt;&gt;(textView);
    this.testObjectWeakReference = new WeakReference&lt;&gt;(testObject);
  }

  @Override
  public void onLongTimeCallback() {
    if (textViewWeakReference != null && textViewWeakReference.get() != null) {
      TextView textView = textViewWeakReference.get();
      if (testObjectWeakReference != null && testObjectWeakReference.get() != null) {
        textView.setText("onLongTimeCallback " + testObjectWeakReference.get().getCreateTime());
      } else {
        textView.setText("onLongTimeCallback testObject is not alive");
      }
    }
  }
}
</pre>

在上面的例子中，TestObject 也传进了 WeakLongTimeCallback，然而 TestObject 的实例并没有在其他地方有引用，因此一旦 GC，TestObject 的实例就会被回收掉。

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_4.png)

在 Android Studio 中强制 GC 的方法：

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_5.png)

打开应用，点击按钮后强制 GC，可看到 TextView 输出了 "onLongTimeCallback testObject is not alive"。

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_6.png)

---

## 改进方案二：将当前对象放入 WeakReference 中

可以让 MainActivity 实现 LongTimeOperation.LongTimeCallback 接口，并在 WeakLongTimeCallback 中持有 MainActivity 的弱引用。

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_7.png)

这个改动的好处是，WeakLongTimeCallback 只需要存 MainActivity 对象的引用，

### 缺点

这个改动同样不支持在调用 LongTimeOperation 时传入 TestObject。

<pre class='mcode'>
public class Activity3 extends AppCompatActivity implements LongTimeOperation.LongTimeCallback {
  // ...

  private void startLongTimeOperation() {
    LongTimeOperation longTimeOperation = new LongTimeOperation();
    longTimeOperation.setLongTimeCallback(new WeakLongTimeCallback(this));
    longTimeOperation.start();
  }

  @Override
  public void onLongTimeCallback() {
    text.setText("onLongTimeCallback");
  }

  static class WeakLongTimeCallback implements LongTimeOperation.LongTimeCallback {
    private WeakReference&lt;Activity3&gt; activity3WeakReference;

    public WeakLongTimeCallback(Activity3 activity3) {
      this.activity3WeakReference = new WeakReference&lt;&gt;(activity3);
    }

    @Override
    public void onLongTimeCallback() {
      if (activity3WeakReference != null && activity3WeakReference.get() != null) {
        activity3WeakReference.get();
      }
    }
  }
}
</pre>

---

## 改进方案三：Activity 销毁时清除回调

在 MainActivity 中存 LongTimeOperation 实例，并在 onDestroy() 里清除 longTimeOperation 的回调。

这个方案的改动量小，但是也需要 Activity 中存下所有耗时操作的实例，并在 onDestroy() 里清除这些操作的回调。

<pre class='mcode'>
public class Activity4 extends AppCompatActivity {
  // ...
 
  private LongTimeOperation longTimeOperation;

  private void startLongTimeOperation() {
    longTimeOperation = new LongTimeOperation();
    LongTimeOperation.LongTimeCallback longTimeCallback = new LongTimeOperation.LongTimeCallback() {
      @Override
      public void onLongTimeCallback() {
        text.setText("onLongTimeCallback");
      }
    };
    longTimeOperation.setLongTimeCallback(longTimeCallback);
    longTimeOperation.start();
  }

  @Override
  protected void onDestroy() {
    super.onDestroy();    
    if (longTimeOperation != null) {
      longTimeOperation.setLongTimeCallback(null);
    }
  }
}
</pre>

如果引入了 [lifecycle](http://cashow.github.io/Android-Lifecycle-Learning-Note.html) 库，也可以在 LongTimeOperation 初始化时传入 FragmentActivity，并在 destroy() 中清除 longTimeCallback。这样的话 MainActivity 除了需要在 LongTimeOperation 初始化时传入 Activity 进去，没有其他任何的改动。

<pre class='mcode'>
public LongTimeOperation(FragmentActivity activity) {
  mMainHandler = new Handler(Looper.myLooper());

  activity.getLifecycle().addObserver(new LifecycleObserver() {
    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    void destroy() {
      longTimeCallback = null;
    }
  });
}
</pre>

### 缺点

对于第三方库提供的接口，可能只是暴露了一个返回值是 void 的接口，因此拿不到持有回调的实例

---

## 思考题：能否在 LongTimeCallback 里存匿名内部类？

demo 如下：

<pre class='mcode'>
private void startLongTimeOperation() {
  LongTimeOperation longTimeOperation = new LongTimeOperation(this);
  LongTimeOperation.LongTimeCallback longTimeCallback = new WeakLongTimeCallback(new LongTimeOperation.LongTimeCallback() {
    @Override
    public void onLongTimeCallback() {
      text.setText("onLongTimeCallback");
    }
  });
  longTimeOperation.setLongTimeCallback(longTimeCallback);
  longTimeOperation.start();
}

static class WeakLongTimeCallback implements LongTimeOperation.LongTimeCallback {
  private WeakReference&lt;LongTimeOperation.LongTimeCallback&gt; callbackWeakReference;

  public WeakLongTimeCallback(LongTimeOperation.LongTimeCallback callback) {
    this.callbackWeakReference = new WeakReference&lt;&gt;(callback);
  }

  @Override
  public void onLongTimeCallback() {
    if (callbackWeakReference != null && callbackWeakReference.get() != null) {
      callbackWeakReference.get().onLongTimeCallback();
    }
  }
}
</pre>

答案是：并不可以！

这样改动后，引用关系如下：

![a](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/Anonymous-Inner-Class-Leaks_Memory_7.png)

从这个引用关系来看，longTimeCallback 是和 longTimeOperation 解耦的。然而，MainActivity 并没有持有 longTimeCallback 的引用，因此在 GC 时，WeakReference 中的 longTimeCallback 会被回收，导致回调不会被触发。

要避免这种情况，可以把 longTimeCallback 定义成全局变量，这样的话 longTimeCallback 的生命周期就和 MainActivity 一样了。

<pre class='mcode'>
public class Activity7 extends AppCompatActivity {
  // ...
  
  private LongTimeOperation.LongTimeCallback longTimeCallback;

  private void startLongTimeOperation() {
    LongTimeOperation longTimeOperation = new LongTimeOperation();
    longTimeCallback = new LongTimeOperation.LongTimeCallback() {
      @Override
      public void onLongTimeCallback() {
        text.setText("onLongTimeCallback");
      }
    };
    longTimeOperation.setLongTimeCallback(new WeakLongTimeCallback(longTimeCallback));
    longTimeOperation.start();
  }

  static class WeakLongTimeCallback implements LongTimeOperation.LongTimeCallback {
    private WeakReference&lt;LongTimeOperation.LongTimeCallback&gt; callbackWeakReference;

    public WeakLongTimeCallback(LongTimeOperation.LongTimeCallback callback) {
      this.callbackWeakReference = new WeakReference&lt;&gt;(callback);
    }

    @Override
    public void onLongTimeCallback() {
      if (callbackWeakReference != null && callbackWeakReference.get() != null) {
        callbackWeakReference.get().onLongTimeCallback();
      }
    }
  }
}
</pre>