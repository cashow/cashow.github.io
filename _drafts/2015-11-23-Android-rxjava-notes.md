---
layout: post
title: "Android - RxJava学习笔记"
date: 2015-11-23 23:36:13 +0800
tags: Android 学习笔记
---

相关博客链接：  
<http://blog.csdn.net/lzyzsd/article/details/41833541>  
<http://blog.csdn.net/lzyzsd/article/details/44094895>  
<http://blog.csdn.net/lzyzsd/article/details/44891933>  
<http://blog.csdn.net/lzyzsd/article/details/45033611> 
***
RxJava最核心的两个东西是Observables（被观察者，事件源）和Subscribers（观察者）。Observables发出一系列事件，Subscribers处理这些事件。  
一个Observable可以发出零个或者多个事件，直到结束或者出错。每发出一个事件，就会调用它的Subscriber的onNext方法，最后调用Subscriber.onNext()或者Subscriber.onError()结束。 
###创建一个Observable对象
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
这里定义的Observable对象仅仅发出一个Hello World字符串。  
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
这里subscriber仅仅就是打印observable发出的字符串。  
###mySubscriber订阅myObservable
<pre class="mcode">
myObservable.subscribe(mySubscriber);
</pre>
一旦mySubscriber订阅了myObservable，myObservable就会调用mySubscriber对象的onNext和onComplete方法，mySubscriber就会打印出Hello World！  
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
subscribe方法有一个重载版本，接受三个Action1类型的参数，分别对应OnNext，OnComplete， OnError函数。  
<pre class="mcode">
myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction);
</pre>
这里我们并不关心onError和onComplete，所以只需要第一个参数就可以。  
<pre class="mcode">
myObservable.subscribe(onNextAction);
</pre>