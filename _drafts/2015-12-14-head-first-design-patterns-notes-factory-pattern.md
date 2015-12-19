---
layout: post
title: "《Head first设计模式》学习笔记 - 工厂模式"
date: 2015-12-14 22:49:43 +0800
tags: 设计模式 学习笔记 工厂模式
excerpt: <p>工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化推迟到了子类。</p>
---

<div class="alert alert-success" role="alert">工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化推迟到了子类。</div>
###预定披萨
假设你有一个披萨店，预定披萨的代码可能是这么写的：  
<pre class="mcode">
Pizza orderPizza(){
    Pizza pizza = new Pizza();

    // 准备面皮，加调料等
    pizza.prepare();
    // 烘烤
    pizza.bake();
    // 切片
    pizza.cut();
    // 装盒
    pizza.box();

    return pizza;
}
</pre>
###更多的披萨类型
但是，你现在需要更多的披萨类型。你必须增加一些代码，来“决定”适合的披萨类型，然后再“制造”这个披萨：  
<pre class="mcode">
// 现在把披萨类型传入orderPizza
Pizza orderPizza(String type) {
    Pizza pizza;

    // 根据披萨的类型，我们实例化正确的具体类，然后将其赋值给pizza实例变量。
    // 请注意，这里的任何披萨都必须实现pizza接口。
    if (type.equals("cheese")) {
        pizza = new CheesePizza();
    } else if (type.equals("greek")) {
        pizza = new GreekPizza();
    } else if (type.equals("pepperoni")) {
        pizza = new PepperoniPizza();
    }

    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();

    return pizza;
}
</pre>
###修改披萨类型
你发现你所有的竞争者都已经在他们的菜单中加入了一些流行风味的披萨：Clam Pizza（蛤蜊披萨）、Veggie Pizza（素食披萨）。很明显，你必须要赶上他们，所以也要把这些风味加进你的菜单中。而最近Greek Pizza（希腊披萨）卖得不好，所以你决定将它从菜单中去掉：  
<pre class="mcode">
Pizza orderPizza(String type) {
    Pizza pizza;

    // 此代码没有对修改封闭。如果披萨店改变它所供应的披萨风味，就得进到这里进行修改。  
    if (type.equals("cheese")) {
        pizza = new CheesePizza();
//  } else if (type.equals("greek")) {
//      pizza = new GreekPizza();
    } else if (type.equals("pepperoni")) {
        pizza = new PepperoniPizza();
    } else if (type.equals("clam")) {
        pizza = new ClamPizza();
    } else if (type.equals("veggie")) {
        pizza = new VeggiePizza();
    }

    // 这里是我们不想改变的地方。因为披萨的准备、烘烤、包装，多年来持续不变，
    // 所以这部分的代码不会改变，只有发生这些动作的披萨会改变。
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();

    return pizza;
}
</pre>
很明显地，如果实例化“某些”具体类，将使orderPizza()出问题，而且也无法让orderPizza()对修改关闭；但是，现在我们已经知道哪些会改变，哪些不会改变，该是使用封装的时候了。  
###建立一个简单披萨工厂
现在最好将创建对象移到orderPizza()之外，但怎么做呢？我们可以把创建披萨的代码移到另一个对象中，由这个新对象专职创建披萨。  
我们称这个新对象为“工厂”。  
工厂（factory）处理创建对象的细节。一旦有了SimplePizzaFactory，orderPizza()就变成此对象的客户。当需要披萨时，就叫披萨工厂做一个。那些orderPizza()方法需要知道希腊披萨或者蛤蜊披萨的日子一去不复返了。现在orderPizza()方法只关心从工厂得到了一个披萨，而这个披萨实现了Pizza接口，所以它可以调用prepare()、bake()、cut()、box()来分别进行准备、烘烤、切片、装盒。  
<pre class="mcode">
// SimplePizzaFactory是我们的新类，它只做一件事情：帮它的客户创建披萨
public class SimplePizzaFactory {
    // 首先，在这个工厂内定义一个createPizza()方法，所有客户用这个方法来实例化新对象。
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        
        // 这是从orderPizza()方法中移过来的代码
        if (type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new PepperoniPizza();
        } else if (type.equals("clam")) {
            pizza = new ClamPizza();
        } else if (type.equals("veggie")) {
            pizza = new VeggiePizza();
        }
        return pizza;
    }
}

// 现在我们为PizzaStore加上一个对SimplePizzaFactory的引用
public class PizzaStore {
    SimplePizzaFactory factory;
    
    // PizzaStore的构造器，需要一个工厂作为参数
    public PizzaStore(SimplePizzaFactory factory){
        this.factory = factory;
    }
    
    public Pizza orderPizza(String type) {
        Pizza pizza;
        
        // orderPizza()方法通过简单传入订单类型来使用工厂创建披萨。
        // 请注意，我们把new操作符替换成工厂对象的创建方法。这里不再使用具体实例化
        pizza = factory.createPizza(type);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
}
</pre>
###加盟披萨店
你的披萨店经营有成，击败了竞争者，现在大家都希望能在自家附近有加盟店。身为加盟公司经营者，你希望确保加盟店运营的质量，所以希望这些店都使用你那些经过时间考验的代码。  
但是区域的差异呢？每家加盟店都可能想要提供不同风味的披萨（比方说纽约、芝加哥、加州），这受到了开店地点及该地区披萨美食家口味的影响。  
在推广SimplePizzaFactory时，你发现加盟店的确是采用你的工厂创建披萨，但是其他部分，却开始采用他们自创的流程：烘烤的做法有差异、不要切片、使用其他厂商的盒子。  
能不能建立一个框架，把加盟店和创建披萨捆绑在一起的同时又保持一定的弹性？  
###给披萨店使用的框架
有个做法可让披萨制作活动局限于PizzaStore类，而同时又能让这些加盟店依然可以自由地制作该区域的风味。  
所要做的事情，就是把createPizza()方法放回到PizzaStore中，不过要把它设置成“抽象方法”，然后为每个区域风味创建一个PizzaStore的子类。  
首先，看PizzaStore所做的改变：  
<pre class="mcode">
public abstract class PizzaStore {

    public Pizza orderPizza(String type) {
        Pizza pizza;

        // 现在createPizza()方法从工厂对象中移回PizzaStore
        pizza = createPizza(type);

        // 这些都没变
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }
    
    // 现在把工厂对象移到这个方法中
    // 在PizzaStore里，“工厂方法”现在是抽象的
    abstract Pizza createPizza(String type);
}
</pre>
现在已经有一个PizzaStore作为超类；让每个域类型（NYPizzaStore、ChicagoPizzaStore、CaliforniaPizzaStore）都继承这个PizzaStore，每个子类各自决定如何制作披萨。  
###允许子类做决定
PizzaStore已经有一个不错的订单系统，由orderPizza()方法负责处理订单，而你希望所有加盟店对于订单的处理都能一致。  
各个区域披萨店之间的差异在于他们制作披萨的风味（纽约披萨的薄脆、芝加哥披萨的饼厚等），我们现在要让现在createPizza()能够应对这些变化来负责创建正确种类的披萨。做法是让PizzaStore的各个子类负责定义自己的createPizza()方法。所以我们会得到一些PizzaStore具体的子类，每个子类都有自己的披萨变体，而仍然适合PizzaStore框架，并使用调试好的orderPizza()方法。  
<pre class="mcode">
public abstract class PizzaStore {

    public Pizza orderPizza(String type) {
        Pizza pizza;

        pizza = createPizza(type);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }

    // 每个子类都会覆盖createPizza()方法
    abstract Pizza createPizza(String type);
}

// 如果加盟店为顾客提供纽约风味的披萨，就使用NyStylePizzaStore，
// 因为此类的createPizza()方法会建立纽约风味的披萨
public class NyStylePizzaStore extends PizzaStore{

    @Override
    Pizza createPizza(String type) {
        Pizza pizza = null;

        if (type.equals("cheese")) {
            pizza = new NyStyleCheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new NyStylePepperoniPizza();
        } else if (type.equals("clam")) {
            pizza = new NyStyleClamPizza();
        } else if (type.equals("veggie")) {
            pizza = new NyStyleVeggiePizza();
        }
        return pizza;
    }
}

// 类似的，利用芝加哥子类，我们得到了带芝加哥原料的createPizza()实现
public class ChicagoStylePizzaStore extends PizzaStore{

    @Override
    Pizza createPizza(String type) {
        Pizza pizza = null;

        if (type.equals("cheese")) {
            pizza = new ChicagoCheesePizza();
        } else if (type.equals("pepperoni")) {
            pizza = new ChicagoPepperoniPizza();
        } else if (type.equals("clam")) {
            pizza = new ChicagoClamPizza();
        } else if (type.equals("veggie")) {
            pizza = new ChicagoVeggiePizza();
        }
        return pizza;
    }
}
</pre>
现在问题来了，PizzaStore的子类终究只是子类，如何能够做决定？在NyStylePizzaStore类中，并没有看到任何做决定逻辑的代码。  
关于这个方面，要从PizzaStore的orderPizza()方法观点来看，此方法在抽象的PizzaStore内定义，但是只在子类中实现具体类型。  
orderPizza()方法对对象做了许多事情（例如：准备、烘烤、切片、装盒），但由于Pizza对象是抽象的，orderPizza()并不知道哪些实际的具体类参与进来了。换句话说，这就是解耦（decouple）！  
当orderPizza()调用createPizza()时，某个披萨店子类将负责创建披萨。做哪一种披萨呢？当然是由具体的披萨店决定。  
那么，子类是实时做出这样的决定吗？不是，但从orderPizza()的角度看，如果选择在NyStylePizzaStore订购披萨，就是由这个子类（NyStylePizzaStore）决定。严格来说，并非由这个子类实际做“决定”，而是由“顾客”决定哪一家风味的披萨店才决定了披萨的风味。  
###工厂模式
工厂方法模式（Factory Method Pattern）通过让子类决定该创建的对象是什么，来达到将对象创建的过程封装的目的。  
PizzaStore就是创建者（Creator）类。它定义了一个抽象的工厂方法，让子类实现此方法制造产品。  
创建者通常会包含依赖于抽象产品的代码，而这些抽象产品由子类制造。创建者不需要真的知道在制造哪种具体产品。  
能够产生产品的类称为具体创建者。NYPizzaStore和ChicagoPizzaStore就是具体创建者。  
Pizza是产品类。工厂生产产品，对PizzaStore来说，产品就是Pizza。  
抽象的Creator提供了一个创建对象的方法的接口，也称为“工厂方法”。在抽象的Creator中，任何其他实现的方法，都可能使用到这个工厂方法所制造出来的产品，但只有子类真正实现这个工厂方法并创建产品。  
<pre class="mcode">
// Creator是一个类，它实现了所有操纵产品的方法，但不实现工厂方法
public abstract class Creator{
    void anOperation(){
        // ...
    }
    // Creator的所有子类都必须实现这个抽象的factoryMethod()方法
    abstract void factoryMethod();
}

// 具体的创建者
public class ConcreteCreator extends Creator{
    // ConcreteCreator实现了factoryMethod()，以实际制造出产品。
    @Override
    void factoryMethod() {
        // ...
    }
}

// 所有产品必须实现这个接口，这样一来，
// 使用这些产品的类就可以引用这个接口，而不是具体的类
public abstract class Product{
    void operation(){
        // ...
    }
}

// 具体的产品
public class ConcreteProduct extends Product{
}
</pre>