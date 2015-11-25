---
layout: post
title: "Android - RxJava学习笔记"
date: 2015-11-23 23:36:13 +0800
tags: Android 学习笔记
---

相关博客链接：  
[深入浅出RxJava(一：基础篇)](http://blog.csdn.net/lzyzsd/article/details/41833541>)  
[深入浅出RxJava(二：操作符)](http://blog.csdn.net/lzyzsd/article/details/44094895)  
[深入浅出RxJava三--响应式的好处](http://blog.csdn.net/lzyzsd/article/details/44891933)  
[深入浅出RxJava四-在Android中使用响应式编程](http://blog.csdn.net/lzyzsd/article/details/45033611) 
***
RxJava最核心的两个东西是Observables（被观察者，事件源）和Subscribers（观察者）。Observables发出一系列事件，Subscribers处理这些事件。  
Observable每发出一个事件，就会调用它的Subscriber的onNext方法。如果出错，会调用Subscriber的onError()，否则调用Subscriber的onComplete()。 
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
###mySubscriber订阅myObservable
<pre class="mcode">
myObservable.subscribe(mySubscriber);
</pre>
一旦mySubscriber订阅了myObservable，myObservable就会调用mySubscriber对象的onNext，mySubscriber就会打印出“Hello World！”。如果没有出错，调用mySubscriber的onComplete方法，否则调用onError方法。  
###简化Observable的创建
Observable.just用来创建只发出一个事件就结束的Observable对象。  
<pre class="mcode">
Observable<String> myObservable = Observable.just("Hello, world!");
</pre>
###简化Subscriber的创建
如果只需要在onNext的时候做一些处理，可以使用Action1类。  
<pre class="mcode">
Action1&lt;String&gt; onNextAction = new Action1&lt;String&gt;() {
    @Override
    public void call(String s) {
        System.out.println(s);
    }
};
</pre>
###简化mySubscriber订阅myObservable
subscribe方法有一个重载版本，接受三个Action1类型的参数，分别对应OnNext，OnComplete， OnError函数。  
<pre class="mcode">
myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction);
</pre>
这里我们并不关心onError和onComplete，所以只需要第一个参数就可以。  
<pre class="mcode">
myObservable.subscribe(onNextAction);
</pre>
上述代码可改写成：  
<pre class="mcode">
Observable.just("Hello, world!")
.subscribe(new Action1&lt;String&gt;() {
    @Override
    public void call(String s) {
          System.out.println(s);
    }
});

//使用lambda表达式简化后：
Observable.just("Hello, world!")
    .subscribe(s -> System.out.println(s));
</pre>
###map操作符
map操作符用来把Observable传来的数据转换成另一个数据。map操作符返回一个Observable对象，这样就可以实现链式调用。
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
Observable.from("url1", "url2", "url3")
    .subscribe(url -> System.out.println(url));
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
// 以下是统计2秒内view的点击次数

RxView.clickEvents(_tapBtn)
      .map(new Func1&lt;ViewClickEvent, Integer&gt;() {
          @Override
          public Integer call(ViewClickEvent onClickEvent) {
              Timber.d("--------- GOT A TAP");
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
              Timber.d("----- onCompleted");
          }

          @Override
          public void onError(Throwable e) {
              Timber.e(e, "--------- Woops on error!");
              _log("Dang error! check your logs");
          }

          @Override
          public void onNext(List&lt;Integer&gt; integers) {
              Timber.d("--------- onNext");
              if (integers.size() &gt; 0) {
                  _log(String.format("%d taps", integers.size()));
              } else {
                  Timber.d("--------- No taps received ");
              }
          }
      });
</pre>
###debounce操作符
在Observable输出一个元素后，忽略一段时间之内的所有数据。
<pre class="mcode">
// 在textChangeEvents发生后，忽略400ms内的所有textChangeEvents

RxTextView.textChangeEvents(_inputSearchText)
          .debounce(400, TimeUnit.MILLISECONDS)// default Scheduler is Computation
          .observeOn(AndroidSchedulers.mainThread())
          .subscribe(_getSearchObserver());
</pre>

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
###调度器
subscribeOn()可以指定观察者代码运行的线程，observerOn()可以指定订阅者运行的线程：  
<pre class="mcode">
myObservableServices.retrieveImage(url)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(bitmap -> myImageView.setImageBitmap(bitmap));
</pre>
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