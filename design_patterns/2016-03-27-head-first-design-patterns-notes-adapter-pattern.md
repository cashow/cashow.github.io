---
layout: post
title: "《Head first设计模式》学习笔记 - 适配器模式"
date: 2016-03-27 15:40:06 +0800
tags: 设计模式
excerpt: <p>适配器模式将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。</p>
---

<div class="alert alert-success" role="alert">适配器模式将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。</div>

### 我们周围的适配器

如果你需要在欧洲国家使用美国制造的笔记本电脑，你可能需要使用一个交流电的适配器。  
你知道适配器的作用：它位于美式插头和欧式插座的中间，它的工作是将欧式插座转换成美式插座，好让美式插头可以插进这个插座得到电力。或者也可以这么认为：适配器改变了插座的接口，以符合美式笔记本电脑的需求。  
好了，这是真实世界的适配器，那面向对象适配器又是什么呢？其实，OO适配器和真实世界的适配器扮演着同样的角色：将一个接口转换成另一个接口，以符合客户的期望。  

### 面向对象适配器
假设已有一个软件系统，你希望它能和一个新的产商类库搭配使用，但是这个新产商所设计出来的接口，不同于旧产商的接口：  
![adapter_0](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/adapter_0.png)  
你不想改变现有的代码，解决这个问题（而且你也不能改变产商的代码）。所以该怎么做？你可以写一个类，将新产商接口转换成你所期望的接口。
![adapter_1](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/adapter_1.png)  
这个适配器工作起来就如同一个中间人，它将客户所发出的请求转换成产商类能理解的请求。这个适配器实现了你的类所期望的接口，而且这个适配器也能和产商的接口沟通。

### 火鸡转换器
我们先来看看一个简化的鸭子和火鸡的接口和类：  
<pre class="mcode">
// 鸭子基类
public interface Duck {
    public void quack();
    public void fly();
}

// 绿头鸭是鸭子的子类，实现了鸭子的呱呱叫和飞行的能力
public class MallardDuck implements Duck{
    @Override
    public void quack() {
        System.out.println("Quack");
    }

    @Override
    public void fly() {
        System.out.println("I'm flying");
    }
}

// 火鸡基类
public interface Turkey {
    // 火鸡不会呱呱叫，只会咯咯叫
    public void gobble();
    // 火鸡会飞，虽然飞不远
    public void fly();
}

// 野生火鸡
public class WildTurkey implements Turkey{
    @Override
    public void gobble() {
        System.out.println("Gobble gobble");
    }

    @Override
    public void fly() {
        System.out.println("I'm flying a short distance");
    }
}
</pre>
现在，假设你缺鸭子对象，想用一些火鸡对象来冒充。显而易见，因为火鸡的接口不同，所以我们不能公然拿来用。那么，写个适配器吧：  
<pre class="mcode">
// 首先，你需要实现想转换成的类型接口，也就是你的客户期望看到的接口
public class TurkeyAdapter implements Duck {
    Turkey turkey;

    // 接着，需要取得要适配的对象引用，这里我们引用构造器取得这个引用
    public TurkeyAdapter(Turkey turkey) {
        this.turkey = turkey;
    }

    // 现在我们需要实现接口中所有的方法。quack()在类之间的转换很简单，
    // 只要调用gobble()接可以了
    @Override
    public void quack() {
        turkey.gobble();
    }

    // 固然两个接口都具备了fly()方法，火鸡的飞行距离很短，不像鸭子可以长途飞行。
    // 要让鸭子的飞行和火鸡的飞行能够对应，必须连续五次调用火鸡的fly()来完成
    @Override
    public void fly() {
        for (int i = 0; i < 5; i++) {
            turkey.fly();
        }
    }
}
</pre>

### 适配器模式解析
客户使用适配器的过程如下：  
1. 客户通过目标接口调用适配器的方法对适配器发出请求；  
2. 适配器使用被适配者接口把请求转换成被适配者的一个或多个调用接口；  
3. 客户接收到调用的结果，但并未察觉这一切是适配器在起转换作用。  

请注意，客户和被适配者是解耦的，一个不知道另一个。

现在，我们知道，这个模式可以通过创建适配器进行接口转换，让不兼容的接口变成兼容。这可以让客户从实现的接口解耦。如果在一段时间之后，我们想要改变接口，适配器可以将改变的部分封装起来，客户就不必为了应对不同的接口而每次跟着修改。

这个适配器模式充满着良好的OO设计原则：使用对象组合，以修改的接口包装被适配者。这种做法还有额外的优点，那就是，被适配者的任何子类，都可以搭配着适配器使用。

也请留意，这个模式是如何把客户和接口绑定起来，而不是和实现绑定起来的。我们可以使用数个适配器，每一个都负责转换不同组的后台类。或者，也可以加上新的实现，只要它们遵守目标接口就可以。
