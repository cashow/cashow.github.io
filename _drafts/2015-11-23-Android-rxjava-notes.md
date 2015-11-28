---
layout: post
title: "Android - RxJava学习笔记"
date: 2015-11-23 23:36:13 +0800
tags: Android 学习笔记
---

以下是学习RxJava过程中所接触到的博客以及项目：  
[深入浅出RxJava(一：基础篇)](http://blog.csdn.net/lzyzsd/article/details/41833541>)  
[深入浅出RxJava(二：操作符)](http://blog.csdn.net/lzyzsd/article/details/44094895)  
[深入浅出RxJava三--响应式的好处](http://blog.csdn.net/lzyzsd/article/details/44891933)  
[深入浅出RxJava四-在Android中使用响应式编程](http://blog.csdn.net/lzyzsd/article/details/45033611)  
[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)  
[ReactiveX官网](http://reactivex.io/)  
[Github项目 : RxJava-Android-Samples](https://github.com/kaushikgopal/RxJava-Android-Samples)  
***
RxJava最核心的是Observables（被观察者，事件源）和Subscribers（观察者）。Observables发出一系列事件，Subscribers处理这些事件。  
###创建一个Observable
<pre class="mcode">
Observable&lt;String&gt; myObservable = Observable.create(
    new Observable.OnSubscribe&lt;String&gt;() {
        @Override
        public void call(Subscriber&lt;? super String&gt; sub) {
            sub.onNext("Hello, world!");
            sub.onCompleted();
        }
    }
);
</pre>
这里定义的myObservable是给所有的Subscriber发出一个Hello World字符串。  
###创建一个Subscriber
<pre class="mcode">
Subscriber&lt;String&gt; mySubscriber = new Subscriber&lt;String&gt;() {
    @Override
    public void onNext(String s) { System.out.println(s); }

    @Override
    public void onCompleted() { }

    @Override
    public void onError(Throwable e) { }
};
</pre>
这里的mySubscriber仅仅就是打印observable发出的字符串。  
如果只需要在onNext，onError或者onCompleted的时候做一些处理，可以使用Action0和Action1类。  
Action0 是 RxJava 的一个接口，它只有一个方法 call()，这个方法是无参无返回值的。  
Action1 也是一个接口，它同样只有一个方法 call(T param)，这个方法也无返回值，但有一个参数。  
<pre class="mcode">
Action1&lt;String&gt; onNextAction = new Action1&lt;String&gt;() {
    // onNext()
    @Override
    public void call(String s) {
        Log.d(tag, s);
    }
};
Action1&lt;Throwable&gt; onErrorAction = new Action1&lt;Throwable&gt;() {
    // onError()
    @Override
    public void call(Throwable throwable) {
        // Error handling
    }
};
Action0 onCompletedAction = new Action0() {
    // onCompleted()
    @Override
    public void call() {
        Log.d(tag, "completed");
    }
};

</pre>
###mySubscriber订阅myObservable
mySubscriber订阅myObservable后，Observable每发出一个事件，就会调用它的Subscriber的onNext()。如果出错，会调用Subscriber的onError()。当不会再有新的onNext()发出时，会调用onCompleted()方法。  
subscribe方法有一个重载版本，接受三个Action1类型的参数，分别对应OnNext，OnComplete， OnError函数。  
<pre class="mcode">
myObservable.subscribe(mySubscriber);

// 自动创建 Subscriber ，并使用 onNextAction 来定义 onNext()
observable.subscribe(onNextAction);
// 自动创建 Subscriber ，并使用 onNextAction 和 onErrorAction 来定义 onNext() 和 onError()
observable.subscribe(onNextAction, onErrorAction);
// 自动创建 Subscriber ，并使用 onNextAction、 onErrorAction 和 onCompletedAction 来定义 onNext()、 onError() 和 onCompleted()
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
</pre>
***
###just操作符
just将传入的数据依次发出。  
<pre class="mcode">
Observable observable = Observable.just("Hello", "Hi", "Aloha");
// 将会依次调用：
// onNext("Hello");
// onNext("Hi");
// onNext("Aloha");
// onCompleted();
</pre>
###map操作符
map操作符用来把Observable传来的数据转换成另一个数据。
<pre class="mcode">
Observable.just("Hello, world!")
	.map(new Func1&lt;String, String&gt;() {
	  @Override
	  public String call(String s) {
	      return s + " -Dan";
	  }
})
.subscribe(s -&gt; System.out.println(s));

//使用lambda表达式简化后：
Observable.just("Hello, world!")
    .map(s -> s + " -Dan")
    .subscribe(s -> System.out.println(s));
</pre>
###from操作符
from操作符接收一个集合作为输入，每次输出一个元素给subscriber
<pre class="mcode">
String[] words = {"Hello", "Hi", "Aloha"};
Observable observable = Observable.from(words);
// 将会依次调用：
// onNext("Hello");
// onNext("Hi");
// onNext("Aloha");
// onCompleted();
</pre>
###flatMap操作符
flatMap将Observable的数据转换成一个或多个Observable。  
<pre class="mcode">
// 假设有个函数根据输入的字符串返回一个url列表：
Observable&lt;List&lt;String&gt;&gt; query(String text){
	// ...
} 
// 现在需要查询"Hello, world!"字符串并且显示结果

query("Hello, world!")
    .flatMap(new Func1&lt;List&lt;String&gt;, Observable&lt;String&gt;&gt;() {
        @Override
        public Observable&lt;String&gt; call(List&lt;String&gt; urls) {
            return Observable.from(urls);
        }
    })
    .subscribe(url -&gt; System.out.println(url));

//使用lambda表达式简化后：
query("Hello, world!")
    .flatMap(urls -> Observable.from(urls))
    .subscribe(url -> System.out.println(url));
</pre>
###filter操作符
filter把输入的数据进行过滤，然后输出符合条件的数据。  
<pre class="mcode">
//过滤title为null的数据
query("Hello, world!")
	.flatMap(urls -&gt; Observable.from(urls))	
	.flatMap(url -&gt; getTitle(url))
	.filter(title -&gt; title != null)
	.subscribe(title -&gt; System.out.println(title));
</pre>
###take操作符
take指定最多输出多少个结果。
<pre class="mcode">
query("Hello, world!")
    .flatMap(urls -&gt; Observable.from(urls))
    .flatMap(url -&gt; getTitle(url))
    .filter(title -&gt; title != null)
    .take(5)
    .subscribe(title -&gt; System.out.println(title));
</pre>
###doOnNext操作符
doOnNext允许我们在每次输出一个元素之前做一些额外的事情，比如这里的保存标题。
<pre class="mcode">
query("Hello, world!")
    .flatMap(urls -&gt; Observable.from(urls))
    .flatMap(url -&gt; getTitle(url))
    .filter(title -&gt; title != null)
    .take(5)
    .doOnNext(title -&gt; saveTitle(title))
    .subscribe(title -&gt; System.out.println(title));
</pre>
###buffer操作符
周期性地把Observable的数据合并成列表，并在一定时间后将列表传给Observer
<pre class="mcode">
// 每2秒更新一次在这期间view的点击次数
RxView.clickEvents(_tapBtn)
      .map(new Func1&lt;ViewClickEvent, Integer&gt;() {
          @Override
          public Integer call(ViewClickEvent onClickEvent) {
              _log("GOT A TAP");
              return 1;
          }
      })
      .buffer(2, TimeUnit.SECONDS)
      .observeOn(AndroidSchedulers.mainThread())
      .subscribe(new Observer&lt;List&lt;Integer&gt;&gt;() {

          @Override
          public void onCompleted() {
              // fyi: you'll never reach here
          }

          @Override
          public void onError(Throwable e) {
              _log("Dang error! check your logs");
          }

          @Override
          public void onNext(List&lt;Integer&gt; integers) {
              Timber.d("--------- onNext");
              if (integers.size() &gt; 0) {
                  _log(String.format("%d taps", integers.size()));
              }
          }
      });
</pre>
###debounce操作符
在一次事件发生后的一段时间内没有其他的操作，则发出这次事件
<pre class="mcode">
// 在textChangeEvents发生后的400ms内的没有收到其他的textChangeEvents事件，则发出这次事件
RxTextView.textChangeEvents(_inputSearchText)
          .debounce(400, TimeUnit.MILLISECONDS)
          .observeOn(AndroidSchedulers.mainThread())
          .subscribe(_getSearchObserver());
</pre>
###throttleFirst操作符
在每次事件触发后的一定时间间隔内丢弃新的事件。常用作去抖动过滤，例如按钮的点击监听器：  
<pre class="mcode">
RxView.clickEvents(button)
    .throttleFirst(500, TimeUnit.MILLISECONDS) // 设置防抖间隔为 500ms
    .subscribe(subscriber);
</pre>
***
###错误处理
代码中的potentialException() 和 anotherPotentialException()有可能会抛出异常。每一个Observerable对象在终结的时候都会调用onCompleted()或者onError()方法，所以以下代码会打印出”Completed!”或者”Ouch!”。
<pre class="mcode">
Observable.just("Hello, world!")
    .map(s -&gt; potentialException(s))
    .map(s -&gt; anotherPotentialException(s))
    .subscribe(new Subscriber&lt;String&gt;() {
        @Override
        public void onNext(String s) { System.out.println(s); }

        @Override
        public void onCompleted() { System.out.println("Completed!"); }

        @Override
        public void onError(Throwable e) { System.out.println("Ouch!"); }
    });
</pre>
这种模式有以下几个优点：  
1.只要有异常发生onError()一定会被调用  
这极大的简化了错误处理。只需要在一个地方处理错误即可以。  
2.操作符不需要处理异常  
将异常处理交给订阅者来做，Observerable的操作符调用链中一旦有一个抛出了异常，就会直接执行onError()方法。  
3.你能够知道什么时候订阅者已经接收了全部的数据。  
知道什么时候任务结束能够帮助简化代码的流程。（虽然有可能Observable对象永远不会结束） 
###订阅关系Subscription
调用Observable.subscribe()会返回一个Subscription对象，这个对象代表了被观察者和订阅者之间的联系。  
<pre class="mcode">
Subscription subscription = Observable.just("Hello, World!")
    .subscribe(s -> System.out.println(s));
</pre>
你可以在后面使用这个Subscription对象来操作被观察者和订阅者之间的联系。
<pre class="mcode">
subscription.unsubscribe();
System.out.println("Unsubscribed=" + subscription.isUnsubscribed());
// Outputs "Unsubscribed=true"
</pre>
RxJava在处理unsubscribing的时候，会停止整个调用链。如果你使用了一串很复杂的操作符，调用unsubscribe将会在他当前执行的地方终止，不需要做任何额外的工作！  
一般在unsubscribe()调用前，可以使用isUnsubscribed()先判断一下状态。
###取消所有的订阅
使用CompositeSubscription来持有所有的Subscriptions，然后在onDestroy()或者onDestroyView()里取消所有的订阅。
<pre class="mcode">
private CompositeSubscription mCompositeSubscription
    = new CompositeSubscription();

private void doSomething() {
    mCompositeSubscription.add(
        AndroidObservable.bindActivity(this, Observable.just("Hello, World!"))
        .subscribe(s -> System.out.println(s)));
}

@Override
protected void onDestroy() {
    super.onDestroy();

    mCompositeSubscription.unsubscribe();
}
</pre>
***
###利用compose进行链式调用
假设在程序中有多个 Observable ，并且他们都需要应用一组相同的 lift() 变换：
<pre class="mcode">
observable1
    .lift1()
    .lift2()
    .lift3()
    .lift4()
    .subscribe(subscriber1);
observable2
    .lift1()
    .lift2()
    .lift3()
    .lift4()
    .subscribe(subscriber2);
observable3
    .lift1()
    .lift2()
    .lift3()
    .lift4()
    .subscribe(subscriber3);
observable4
    .lift1()
    .lift2()
    .lift3()
    .lift4()
    .subscribe(subscriber1);
</pre>
使用 compose() 方法，Observable 可以利用传入的 Transformer 对象的 call 方法直接对自身进行处理。
<pre class="mcode">
public class LiftAllTransformer implements Observable.Transformer<Integer, String> {
    @Override
    public Observable<String> call(Observable<Integer> observable) {
        return observable
            .lift1()
            .lift2()
            .lift3()
            .lift4();
    }
}
...
Transformer liftAll = new LiftAllTransformer();
observable1.compose(liftAll).subscribe(subscriber1);
observable2.compose(liftAll).subscribe(subscriber2);
observable3.compose(liftAll).subscribe(subscriber3);
observable4.compose(liftAll).subscribe(subscriber4);
</pre>
***
###Scheduler调度器
在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。  
RxJava通过Scheduler来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 Scheduler：  
<pre class="mcode">
// 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
Schedulers.immediate()

// 总是启用新线程，并在新线程执行操作。
Schedulers.newThread()

// I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。  
Schedulers.io()

// 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。  
Schedulers.computation()

// Android专用，指定操作在Android主线程运行。
AndroidSchedulers.mainThread()
</pre>

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。  
####subscribeOn(): 
指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。  
####observeOn(): 
指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。  
代码示例：  
<pre class="mcode">
Observable.just(1, 2, 3, 4)
    .subscribeOn(Schedulers.io()) // 指定subscribe()发生在IO线程
    .observeOn(AndroidSchedulers.mainThread()) // 指定Subscriber的回调发生在主线程
    .subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer number) {
            Log.d(tag, "number:" + number);
        }
    });
</pre>
observeOn() 指定的是它之后的操作所在的线程。因此如果有多次切换线程的需求，只要在每个想要切换线程的位置调用一次 observeOn() 即可。  
<pre class="mcode">
Observable.just(1, 2, 3, 4) // IO 线程，由 subscribeOn() 指定
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.newThread())
    .map(mapOperator) // 新线程，由 observeOn() 指定
    .observeOn(Schedulers.io())
    .map(mapOperator2) // IO 线程，由 observeOn() 指定
    .observeOn(AndroidSchedulers.mainThread) 
    .subscribe(subscriber);  // Android 主线程，由 observeOn() 指定
</pre>
不过，不同于 observeOn() ， subscribeOn() 的位置放在哪里都可以，但它是只能调用一次的。