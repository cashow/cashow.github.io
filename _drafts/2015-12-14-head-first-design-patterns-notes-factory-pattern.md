---
layout: post
title: "《Head first设计模式》学习笔记 - 工厂模式"
date: 2015-12-14 22:49:43 +0800
tags: 设计模式 学习笔记 工厂模式
excerpt: <p></p>
---

<div class="alert alert-success" role="alert"></div>
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