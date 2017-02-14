---
layout: post
title: "《Head first设计模式》学习笔记 - 装饰者模式"
date: 2015-12-13 20:16:38 +0800
tags: 设计模式 学习笔记 装饰者模式
excerpt: <p>装饰者模式动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。</p>
---

<div class="alert alert-success" role="alert">装饰者模式动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。</div>
星巴兹是以扩张速度最快而闻名的咖啡连锁店。由于扩张速度太快，他们准备更新订单系统，以合乎他们的饮料供应要求。  
他们原先的类设计是这样的：  
<pre class="mcode">
// Beverage(饮料)是抽象的，店内所提供的饮料都必须继承自此类
public abstract class Beverage {
    // 由每个子类设置，用来描述饮料，如“超优深焙咖啡豆”
    protected String description;
    public String getDescription(){
        return description;
    }

    // cost()方法是抽象的，子类必须实现cost()来返回饮料的价格
    public abstract double cost();
}
</pre>
购买咖啡时，也可以要求在其中加入各种调料，例如蒸奶、豆浆、摩卡或覆盖奶泡。星巴兹会根据所加入的调料收取不同的费用。所以订单系统必须考虑到这些调料部分。  

### 订单系统的第一次尝试  
<pre class="mcode">
public class HouseBlend extends Beverage{
    @Override
    public double cost() {
        return 10.0;
    }
}

public class HouseBlendWithSteamedMilk extends Beverage{
    @Override
    public double cost() {
        return 10.5;
    }
}

public class HouseBlendWithSteamedMilkandMocha extends Beverage{
    @Override
    public double cost() {
        return 11.5;
    }
}

...
</pre>
很明显，星巴兹为自己制造了一个维护恶梦。如果牛奶的价格上涨，怎么办？新增一种焦糖调料风味时，怎么办？  

### 利用实例变量和继承追踪这些调料
先从基类下手，加上实例变量代表是否加上调料（牛奶、豆浆、摩卡、奶泡...）
<pre class="mcode">
public abstract class Beverage {
    protected String description;
    // 各种调料的布尔值
    private boolean milk;
    private boolean soy;
    private boolean mocha;
    private boolean whip;

    public String getDescription() {
        return description;
    }

    // 现在，Beverage类中的cost()不再是一个抽象方法，
    // 我们提供了cost()的实现，让它计算要加入各种饮料的调料价格。
    // 子类仍将覆盖cost()，但是会调用超类的cost()，计算出基本饮料加上调料的价格。
    public double cost() {
        double cost = 0.0;
        if (hasMilk()) {
            cost += 1.0;
        }
        if (hasMocha()) {
            cost += 2.5;
        }
        if (hasSoy()) {
            cost += 1.5;
        }
        if (hasWhip()) {
            cost += 2.0;
        }
        return cost;
    }

    // 以下方法取得和设置调料的布尔值
    public boolean hasMilk() {
        return milk;
    }

    public void setMilk(boolean milk) {
        this.milk = milk;
    }

    public boolean hasSoy() {
        return soy;
    }

    public void setSoy(boolean soy) {
        this.soy = soy;
    }

    public boolean hasMocha() {
        return mocha;
    }

    public void setMocha(boolean mocha) {
        this.mocha = mocha;
    }

    public boolean hasWhip() {
        return whip;
    }

    public void setWhip(boolean whip) {
        this.whip = whip;
    }
}
</pre>
现在加入子类，每个类代表菜单上的一种饮料：
<pre class="mcode">
public class HouseBlend extends Beverage{
    public HouseBlend(){
        description = "HouseBlend";
    }

    // 子类的cost()方法需要计算该饮料的价钱，然后通过调用超类的cost()实现，加入调料的价钱
    @Override
    public double cost() {
        return 10.0 + super.cost();
    }
}
</pre>

### 以上设计的潜在问题
当哪些需求或因素改变时会影响这个设计？  
1. 调料价格的改变会使我们更改现有代码；  
2. 一旦出现新的调料，我们就需要加上新的方法，并改变超类中的cost()方法；  
3. 以后可能会开发出新饮料。对这些饮料而言（例如：冰茶），某些调料可能并不适合，但是在这个设计方式中，Tea（茶）子类仍将继承那些不适合的方法，例如：hasWhip()（加奶泡）；  
4. 万一顾客想要双倍摩卡咖啡，怎么办？  

### 继承与复用
尽管继承威力强大，但它并不总是能够实现最有弹性和最好维护的设计。  
利用组合（composition）和委托（delegation）可以在运行时具有继承行为的效果。  
利用继承设计子类的行为，是在编译时静态决定的，而且所有的子类都会继承到相同的行为。然而，如果能够利用组合的做法拓展对象的行为，就可以在运行时动态地进行扩展。可以利用此技巧把多个新职责，甚至是设计超类时还没有想到的职责加在对象上。而且，可以不用修改原来的代码。  
通过动态地组合对象，可以写新的代码添加新功能，而无须修改现有代码。既然没有改变现有的代码，那么引进bug或产生意外副作用的机会将大幅度减少。  
<p class="text-danger">设计原则：类应该对扩展开放，对修改关闭。</p>
我们的目标是允许类容易扩展，在不修改现有代码的情况下，就可搭配新的行为。如能实现这样的目标，有什么好处呢？这样的设计具有弹性可以应对改变，可以接受新的功能来应对改变的需求。  

### 认识装饰者模式
我们已经了解利用继承无法完全解决问题，在星巴兹遇到的问题有：类数量爆炸、设计死板，以及基类加入的新功能并不适用于所有的子类。  
所以，在这里要采用不一样的做法：我们要以饮料为主体，然后在运行时以调料来“装饰”（decorate）饮料。比方说，如果顾客想要摩卡和奶泡深焙咖啡，那么，要做的是：  
1. 拿一个深焙咖啡（DarkRoast）对象；  
2. 以摩卡（Mocha）对象装饰它；  
3. 以奶泡（Whip）对象装饰它；  
4. 调用cost()方法，并依赖委托（delegate）将调料的价格加上去。  

### 以装饰者模式构造饮料订单
1.以DarkRoast对象开始。  
DarkRoast继承自Beverage，且有一个用来计算饮料价钱的cost()方法。    
![decorator_pattern_darkroast](http://7xjvhq.com1.z0.glb.clouddn.com/decorator_pattern_darkroast.jpg)
2.顾客想要摩卡（Mocha），所以建立一个Mocha对象，并用它将DarkRoast对象包（wrap）起来。  
![decorator_pattern_mocha](http://7xjvhq.com1.z0.glb.clouddn.com/decorator_pattern_mocha.jpg)
Mocha对象是个装饰者，它的类型“反映”了它所装饰的对象（本例中，就是Beverage）。所谓的“反映”，指的就是两者类型一致。  
所以Mocha也有一个cost()方法。通过多态，也可以把Mocha所包裹的任何Beverage当成是Beverage（因为Mocha是Beverage的子类型）。  
3.顾客也想要奶泡（Whip），所以需要建立一个Whip装饰者，并用它将Mocha对象包起来。别忘了，DarkRoast继承自Beverage，且有一个cost()方法，用来计算饮料价钱。  
![decorator_pattern_whip](http://7xjvhq.com1.z0.glb.clouddn.com/decorator_pattern_whip.jpg)
Whip是一个装饰者，所以它也反映了DarkRoast类型，并包括一个cost()方法。  
所以，被Mocha和Whip包起来的DarkRoast对象仍然是一个Beverage，仍然可以具有DarkRoast的一切行为，包括调用它的cost()方法。  
4.现在，该是为顾客算钱的时候了。通过调用最外圈装饰者（Whip）的cost()就可以办得到。Whip的cost()会先委托它装饰的对象（也就是Mocha）计算出价钱，然后再加上奶泡的价钱。  
![decorator_pattern_cost](http://7xjvhq.com1.z0.glb.clouddn.com/decorator_pattern_cost.jpg)

### 定义装饰者模式
我们目前所知道的一切：  
1.装饰者和被装饰者有相同的超类型；  
2.你可以用一个或多个装饰者包装一个对象；  
3.既然装饰者和被装饰者对象有相同的超类型，所以在任何需要原始对象（被包装的）的场合，可以用装饰过的对象代替它；  
4.装饰者可以在所委托被装饰者的行为之前或之后，加上自己的行为，以达到特定的目的。  
5.对象可以在任何时候被装饰，所以可以在运行时动态地、不限量地用你喜欢的装饰者来装饰对象。  
以下是实际应用装饰者模式的样例：  
<pre class="mcode">
// 基础组件
public abstract class Component {
    public void methodA(){
    }
}

// 我们将要动态地加上新行为的对象，它扩展自Component
public class ConcreteComponent extends Component {
    public void methodA(){
    }
}

// 这是装饰者共同实现的接口
public abstract class Decorator extends Component {
    public void methodA(){
    }
}

// 每个装饰者都“有一个”（包装一个）组件，也就是说，
// 装饰者有一个实例变量以保存某个Component的引用。
public class ConcreteDecoratorA extends Decorator {
    // wrappedObj是装饰者包着的Component
    Component wrappedObj;
    public void methodA(){
    }
}
</pre>

### 装饰我们的饮料
<pre class="mcode">
// Beverage相当于抽象的Component类
public abstract class Beverage {
    String description = "Unknow Beverage";

    public String getDescription() {
        return description;
    }

    // cost()必须在子类实现
    public abstract double cost();
}

// 让Espresso扩展自Beverage类，因为Espresso是一种饮料（浓缩咖啡）
public class Espresso extends Beverage{
    public Espresso(){
        // 饮料的描述。description实例变量继承自Beverage
        description = "Espresso";
    }

    // 浓缩咖啡的价格
    @Override
    public double cost() {
        return 10.0;
    }
}

// 另一种饮料
public class HouseBlend extends Beverage{
    public HouseBlend(){
        description = "HouseBlend";
    }

    @Override
    public double cost() {
        return 10.0;
    }
}

// 必须让CondimentDecorator能取代Beverage，所以将CondimentDecorator扩展自Beverage类
public abstract class CondimentDecorator extends Beverage{
    // 所有的调料装饰者都必须重新实现getDescription()方法
    public abstract String getDescription();
}

// 摩卡是一个装饰者，所以让它扩展自CondimentDecorator
public class Mocha extends CondimentDecorator{
    // 要让Mocha能够引用一个Beverage，做法如下：
    // 1.用一个实例变量记录饮料，也就是被装饰者
    // 2.想办法让被装饰者（饮料）被记录到实例变量中。
    // 这里的做法是：把饮料当作构造器的参数，再由构造器将此饮料记录在实例变量中
    Beverage beverage;
    
    public Mocha(Beverage beverage){
        this.beverage = beverage;
    }

    // 我们希望叙述不只是描述饮料（例如“DarkRoast”），而是完整地连调料都描述出来（例如“DarkRoast, Mocha”）。
    // 所以首先利用委托的做法，得到一个叙述，然后在其后加上附加的叙述（例如“Mocha”）
    public String getDescription(){
        return beverage.getDescription() + ", Mocha";
    }

    @Override
    public double cost() {
        return 3.5 + beverage.cost();
    }
}

// 测试代码：
// 订一杯Espresso，不需要调料，打印出它的描述与价钱
Beverage beverage = new Espresso();
System.out.println(beverage.getDescription() + " $" + beverage.cost());

// 制造出一个DarkRoast对象
Beverage beverage2 = new DarkRoast();
// 用Mocha装饰它
beverage2 = new Mocha(beverage2);
// 用第二个Mocha装饰它
beverage2 = new Mocha(beverage2);
// 用Milk装饰它
beverage2 = new Milk(beverage2);
System.out.println(beverage2.getDescription() + " $" + beverage2.cost());
</pre>