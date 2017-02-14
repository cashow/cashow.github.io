---
layout: post
title: "《Head first设计模式》学习笔记 - 策略模式"
date: 2015-11-29 18:19:10 +0800
tags: 设计模式 学习笔记 策略模式
excerpt: <p>策略模式定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。</p>
---

<div class="alert alert-success" role="alert">策略模式定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。</div>
假设有一个模拟鸭子的游戏，游戏中会出现各种鸭子，一边游泳戏水，一边呱呱叫。这个游戏的内部设计了一个鸭子超类Duck，并让各种鸭子继承此超类。  
<pre class="mcode">
public abstract class Duck {
    public void quack(){
        // 所有的鸭子都会呱呱叫，由Duck类负责实现
    }
    public void swim(){
        // 所有的鸭子都会游泳，由Duck类负责实现
    }
    public abstract void display();  // 每个鸭子的子类负责实现自己的display
}
</pre>
现在要增加一个功能，让鸭子能飞。


### 实现方法1：在Duck类里加上fly()方法
只需要在Duck类中加上fly()方法，所有的鸭子都会继承fly()。  
但是问题来了，并非所有的子类都会飞，比如橡皮鸭子就不能飞。在超类中加上fly()，就会导致所有的子类都具备fly()，连那些不该具备fly()的子类也无法免除。  
<p class="text-danger">对代码所做的局部修改，影响层面可不只是局部！</p>

### 实现方法2：把fly()从超类中取出来
我们可以把fly()从超类中取出来，放入一个“Flyable接口”中。这么一来，只有会飞的鸭子才能实现此接口。同样的方式，也可以用来设计一个“Quackable接口”，因为不是所有的鸭子都会叫。  
这也是个超笨的主意，这样一来重复的代码会变多。如果有48个Duck的子类都要稍微修改一下飞行的行为，需要修改48个Duck的子类的代码。  
虽然Flyable与Quackable可以解决“一部分”问题（不会再有会飞的橡皮鸭），但是却造成代码无法复用。  
<p class="text-danger">设计原则：找出应用中可能出现变化之处，把它们独立出来，不要和那些不需要变化的代码混在一起。</p>

### 分开变化和不会变化的部分
就我们目前所知，除了fly()和quack()的问题之外，Duck类还算一切正常。现在，为了要分开“变化和不会变化的部分”，我们准备建立两组类（完全远离Duck类），一个是“fly”相关的，一个是“quack”相关的，每一组类将实现各自的动作。

### 设计鸭子的行为
如何设计那组实现飞行和呱呱叫的行为的类呢？  
我们希望一切能有弹性，毕竟，正是因为一开始鸭子行为没有弹性，才让我们走上现在这条路。我们应该在鸭子类中包含设定行为的方法，这样可以实现在“运行时”动态地“改变”鸭子的飞行行为。  
<p class="text-danger">设计原则：针对接口编程，而不是针对实现编程。</p>
<pre class="mcode">
// 针对实现编程
// 声明变量“d”为Dog类型，会造成我们必须针对具体实现编码
Dog d = new Dog();
d.bark();

// 针对接口/超类型编程
// 我们知道该对象是狗，但是我们现在利用animal进行多态的调用
Animal animal = new Dog();
animal.makeSound();

// 更棒的是，子类实例化的动作不再需要在代码中硬编码，
// 例如new Dog()，而是“在运行时才指定具体实现的对象”
// 我们不知道实际的子类型是“什么”，我们只关心它知道如何正确地进行makeSound()的动作就够了
a = getAnimal();
a.makeSound();
</pre>
我们利用接口代表行为，比方说，FlyBehavior与QuackBehavior，而行为的每个实现都将实现其中一个接口。  
所以这次鸭子类不会负责实现Flying与Quacking接口，反而是由我们制造一组其他类专门实现FlyBehavior与QuackBehavior，这就成为“行为”类。  
这样的做法迥异于以往，以前的做法是：行为来自Duck超类的具体实现，或是继承某个接口并由子类自行实现。这两种做法都是依赖于“实现”，我们被“实现”绑得死死的，没办法更改行为。  
在我们的新设计中，鸭子的子类将使用接口（FlyBehavior与QuackBehavior）所表示的行为，所以实际的“实现”不会被绑死在鸭子的子类中。  

### 实现鸭子的行为
在此，我们有两个接口，FlyBehavior与QuackBehavior，还有对应的子类，负责实现具体的行为：  
FlyWithWings继承自FlyBehavior，实现鸭子飞行；  
FlyNoWay继承自FlyBehavior，实现不会飞的鸭子的行为；  
Quack继承自QuackBehavior，实现鸭子呱呱叫；  
Squeak继承自QuackBehavior，实现橡皮鸭子吱吱叫；  
MuteQuack继承自QuackBehavior，实现不会叫的鸭子的行为。  
这样的设计，可以让飞行和呱呱叫的动作被其他的对象复用，因为这些行为已经与鸭子类无关了。这么一来，有了继承的“复用”好处，却没有继承所带来的包袱。  
而我们可以新增一些行为，不会影响到既有的行为类，也不会影响“使用”到飞行行为的鸭子类。  

### 整合鸭子的行为
在Duck类中加入两个实例变量，分别为“flyBehavior”与“quackBehavior”，声明为接口类型，每个鸭子对象都会动态地设置这些变量以在运行时引用正确的行为类型。  
我们用两个相似的方法performFly()与performQuack()取代Duck类中的fly()与quack()。  
实现performQuack()：
<pre class="mcode">
// 在这部分代码中，我们不在乎quackBehavior接口的对象到底是什么，
// 我们只关心该对象知道如何进行呱呱叫就够了。
public class Duck {
    // 每只鸭子都会引用实现QuackBehavior接口的对象
    QuackBehavior quackBehavior;
    
    public void performQuack(){
        // 鸭子对象不亲自处理呱呱叫行为，而是委托给quackBehavior引用的对象
        quackBehavior.quack();
    }
}
</pre>
设定quackBehavior的实例变量：
<pre class="mcode">
public class MallardDuck extends Duck {
    public MallardDuck(){
        // 绿头鸭使用Quack类处理呱呱叫，所以当performQuack()被调用时，
        // 叫的职责被委托给Quack对象，而我们得到了真正的呱呱叫
        quackBehavior = new Quack();
    }
}
</pre>
同样的处理方式也可以用在飞行行为上。  

### 组合
每一个鸭子都有一个FlyBehavior和QuackBehavior，好将飞行和呱呱叫委托给它们代为处理。  
当你将两个类结合起来使用，如同本例一般，这就是组合(composition)。这种做法和“继承”不同的地方在于，鸭子的行为不是继承来的，而是和适当地行为对象“组合”来的。  
<p class="text-danger">设计原则：多用组合，少用继承。</p>
使用组合建立系统具有很大的弹性，不仅可将算法封装成类，更可以“在运行时动态地改变行为”，只要组合的行为对象符合正确的接口标准即可。

### 完整的代码
<pre class="mcode">
// Duck超类
public abstract class Duck {
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;

    public Duck(){
    }

    public abstract void display();

    public void performFly(){
        flyBehavior.fly();
    }

    public void performQuack(){
        quackBehavior.quack();
    }

    public void swim(){
        System.out.println("All ducks float, even decoys!");
    }
}

// MallardDuck类
public class MallardDuck extends Duck {
    public MallardDuck(){
        quackBehavior = new Quack();
        flyBehavior = new FlyWithWings();
    }

    public void display() {
        System.out.println("I'm a real Mallard duck");
    }
}

// 飞行行为接口
public interface FlyBehavior {
    public void fly();
}

// 飞行行为类
public class FlyWithWings implements FlyBehavior {
    public void fly(){
        System.out.println("I'm flying!!");
    }
}

// 飞行行为类
public class FlyNoWay implements FlyBehavior {
    public void fly(){
        System.out.println("I can't fly");
    }
}

// 叫声行为接口
public interface QuackBehavior {
    public void quack();
}

// 叫声行为类
public class Quack implements QuackBehavior{
    public void quack(){
        System.out.println("quack");
    }
}

// 叫声行为类
public class MuteQuack implements QuackBehavior{
    public void quack(){
        System.out.println("<< silence >>");
    }
}

// 叫声行为类
public class Squeak implements QuackBehavior{
    public void quack(){
        System.out.println("squeak");
    }
}

// 测试类
public class MiniDuckSilmulator{
    public static void main(String[] args){
        Duck mallard = new MallardDuck();
        mallard.performQuack();
        mallard.performFly();
    }
}
</pre>