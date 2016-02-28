---
layout: post
title: "《Head first设计模式》学习笔记 - 单件模式"
date: 2016-02-28 15:51:22 +0800
tags: 设计模式 学习笔记 单件模式
excerpt: <p>单件模式确保一个类只有一个实例，并提供一个全局访问点。</p>
---

<div class="alert alert-success" role="alert">单件模式确保一个类只有一个实例，并提供一个全局访问点。</div>

有一些对象我们只需要一个，比方说：线程池、缓存、对话框、处理器偏好设置和注册表的对象等等。事实上，这类对象只能有一个实例，如果制造出多个实例，就会导致许多问题产生，例如：程序的行为异常、资源使用过量，或者是不一致的结果。  

### 使用静态变量
如何确保这些类只存在一个实例？利用java的静态变量可以做到，但使用静态变量有个缺点：如果将对象赋值给一个全局变量，那么你必须在程序一开始就创建好对象。万一这个对象非常耗费资源，而程序在这次的执行过程中又一直没用到它，就形成了浪费。

### 经典的单件实现
以下是经典的单件实现：
<pre class="mcode">
public class Singleton {
    // 利用一个静态变量来记录Singleton的唯一实例。
    private static Singleton uniqueInstance;

    // 把构造器声明为私有的，只有Singleton类内才可以调用构造器。
    private Singleton() {
    }

    // 用getInstance()方法实例化对象，并返回这个实例。
    public static Singleton getInstance() {
        // 如果uniqueInstance是空的，表示还没有创建实例。
        if (uniqueInstance == null) {
            // 如果uniqueInstance是空的，我们就利用私有的构造器产生一个Singleton实例并
            // 把它赋值给uniqueInstance静态变量中。请注意，如果我们不需要这个实例，它就
            // 永远不会产生。这就是“延迟实例化”
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
</pre>
getInstance()是静态的，这意味着它是一个类方法，所以可以在代码的任何地方使用Singleton.getInstance()访问它。这和访问全局变量一样简单，只是多了个优点：单件可以延迟实例化。

### 处理多线程
假如有两个线程同时调用Singleton.getInstance()，而这时uniqueInstance还没有初始化，那么有可能会出现调用Singleton.getInstance()方法返回不同的实例。  
只要把getInstance()变成同步方法，多线程灾难几乎就可以轻易地解决了：  
<pre class="mcode">
public class Singleton {
    private static Singleton uniqueInstance;

    private Singleton() {
    }

    // 通过增加synchronized关键字到getInstance()方法中，我们
    // 迫使每个线程在进入这个方法之前，要先等候别的线程离开该方法。
    // 也就是说，不会有两个线程可以同时进入这个方法。
    public static synchronized Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
</pre>
上面的代码可以解决多线程的问题，但是同步会降低性能，这不又是另一个问题吗？  
只有第一次执行此方法时，才真正需要同步，一旦设置好uniqueInstance变量，就不需要同步这个方法了。之后每次调用这个方法，同步都是一种累赘。

### 改善多线程
为了要符合大多数java应用程序，很明显的，我们需要确保单件模式能在多线程的状况下正常工作。但是似乎同步getInstance()的做法将拖垮性能，该怎么办你？  
可以有一些选择：  

#### 1. 如果getInstance()的性能对应用程序不是很关键，就什么都别做
没错，如果你的应用程序可以接受getInstance()造成的额外负担，就忘了这件事吧。同步getInstance()的方法既简单又有效。但是你必须知道，同步一个方法可能造成程序效率下降100倍。因此如果将getInstance()的程序使用在频繁运行的地方，你可能就要重新考虑了。

#### 2. 使用“急切”创建实例，而不用延迟实例化的做法
如果应用程序总是创建并使用单件实例，或者在创建和运行时方面的负担不太繁重，你可能想要急切创建此单件，如下所示：
<pre class="mcode">
public class Singleton {
    // 在静态初始化器中创建单件。这段代码保证了线程安全
    private static Singleton uniqueInstance = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        // 已经有实例了，直接使用它
        return uniqueInstance;
    }
}
</pre>
利用这个做法，我们依赖JVM在加载这个类时马上创建此唯一的单件实例。JVM保证在任何线程访问uniqueInstance静态变量之前，一定先创建此实例。

#### 3. 用“双重检查加锁”，在getInstance()中减少使用同步
利用双重检查加锁，首先检查是否实例已经创建了，如果尚未创建，才进行同步。这样一来，只有第一次会同步，这正是我们想要的。
<pre class="mcode">
public class Singleton {
    // volatile关键词确保，当uniqueInstance变量被初始化成Singleton实例时，
    // 多个线程正确地处理uniqueInstance变量
    private volatile static Singleton uniqueInstance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        // 检查实例，如果不存在，就进入同步区块。
        if (uniqueInstance == null) {
            // 注意，只有第一次才彻底执行这里的代码
            synchronized (Singleton.class){
                // 进入区块后，再检查一次。如果仍是null，才创建实例
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
</pre>
如果性能是你关心的重点，那么这个做法可以帮你大大地减少getInstance()的时间耗费。