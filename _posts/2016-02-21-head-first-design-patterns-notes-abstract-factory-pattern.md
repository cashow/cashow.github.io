---
layout: post
title: "《Head first设计模式》学习笔记 - 抽象工厂模式"
date: 2016-02-21 17:46:48 +0800
tags: 设计模式 学习笔记 工厂模式 抽象工厂模式
excerpt: <p>抽象工厂模式提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。</p>
---

<div class="alert alert-success" role="alert">抽象工厂模式提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。</div>

### 确保原料的一致
披萨店成功的关键在于新鲜、高质量的原料。要如何确保每家加盟店使用高质量的原料？你打算建造一家生成原料的工厂，并将原料运送到各家加盟店。对于这个做法，现在还剩下一个问题：加盟店坐落在不同的区域，纽约的红酱料和芝加哥的红酱料是不一样的。加盟店之间有相同的产品家族（意式腊肠、酱料、芝士、蔬菜等等），但是制作方式根据区域的不同而有差异。所以对于纽约和芝加哥，你准备了两组不同的原料。假如不久之后加州就有加盟店了，到时候是不是又需要一组新的原料？

### 建造原料工厂
现在，我们要建造一个工厂来生产原料，这个工厂将负责创建原料家族中的每一种原料。也就是说，工厂将需要生产面团、酱料、芝士等。  
开始先为工厂定义一个接口，这个接口负责创建所有的原料：
<pre class="mcode">
public interface PizzaIngredientFactory {
    // 在接口中，每个原料都有一个对应的方法创建该原料
    public Dough createDough();
    public Sauce createSauce();
    public Cheese createCheese();
    public Veggies[] createVeggies();
    public Pepperoni createPepperoni();
    public Clams createClams();
}
</pre>
现在要做的事情是：  
1. 为每个区域建造一个工厂。你需要创建一个继承自PizzaIngredientFactory的子类来实现每一个创建方法；  
2. 实现一组原料类供工厂使用，例如ReggianoCheese、RedPeppers、ThickCrustDough。这些类可以在合适的区域间共享；  
3. 然后你需要将一切组织起来，将新的原料工厂整合进旧的PizzaStore代码中。  

### 创建纽约原料工厂
这是纽约原料工厂的实现。这工厂专精于大蒜番茄酱料、Reggiano干酪、新鲜蛤蜊。

<pre class="mcode">
public class NYPizzaIngredientFactory implements PizzaIngredientFactory {
    // 对于原料家族内的每一种原料，我们都提供了纽约的版本
    @Override
    public Dough createDough() {
        return new ThinCrushDough();
    }

    @Override
    public Sauce createSauce() {
        return new MarinaraSauce();
    }

    @Override
    public Cheese createCheese() {
        return new ReggianoCheese();
    }

    @Override
    public Veggies[] createVeggies() {
        Veggies veggies[] = {new Garlic(), new Onion(), new Mushroom(), new RedPepper()};
        return veggies;
    }

    @Override
    public Pepperoni createPepperoni() {
        // 这是切片的意式腊肠，纽约和芝加哥都会用到它
        return new SlicedPepperoni();
    }

    @Override
    public Clams createClams() {
        // 纽约靠海，所以有新鲜的蛤蜊。芝加哥就必须使用冷冻的蛤蜊
        return new FreshClams();
    }
}
</pre>

### 重做披萨
工厂已经一切就绪，准备生成高质量原料了。现在我们只需要重做披萨，好让它们只使用工厂生产出来的原料。我们先从抽象的Pizza类开始：
<pre class="mcode">
public abstract class Pizza {
    String name;
    // 每个披萨都持有一组在准备时会用到的原料
    Dough dough;
    Sauce sauce;
    Veggies veggies[];
    Cheese cheese;
    Pepperoni pepperoni;
    Clams clams;
    
    // 现在把prepare()方法声明成抽象。在这个方法中，我们需要收集披萨所需的原料，而这些原料当然是来自原料工厂了。
    abstract void prepare();
    
    // 其他的方法保持不动
    void bake(){
        // ...
    }
    
    void cut(){
        // ...
    }
    
    void box(){
        // ...
    }
}
</pre>
现在已经有了一个抽象披萨，可以开始创建纽约和芝加哥风味的披萨了。从今以后，加盟店必须直接从工厂取得原料。  
我们曾经写过工厂方法的代码，有NYCheesePizza和ChicagoCheesePizza类。比较一下这两个类，唯一的差别在于使用区域性的原料，至于披萨的做法都一样。它们都依循着相同的准备步骤，只是使用不同的原料。  
所以，其实我们不需要设计两个不同的类来处理不同风味的披萨，让原料工厂处理这种区域差异就可以了。下面是CheesePizza：
<pre class="mcode">
public class CheesePizza extends Pizza {
    PizzaIngredientFactory ingredientFactory;
    
    // 要制作披萨，需要工厂提供原料。
    // 所以每个披萨类都需要从构造器参数中得到一个工厂，并把这个工厂存储在一个实例变量中。
    public CheesePizza(PizzaIngredientFactory ingredientFactory){
        this.ingredientFactory = ingredientFactory;
    }
    
    @Override
    void prepare() {
        // prepare()方法一步一步地创建芝士披萨，每当需要原料时，就跟工厂要。
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
        cheese = ingredientFactory.createCheese();
    }
}
</pre>
Pizza的代码利用相关的工厂生成原料。所生产的原料依赖所使用的工厂，Pizza类根本不关心这些原料，它只知道如何制作披萨。现在，Pizza和区域原料之间被解耦，无论原料工厂是在落基山脉还是在西北沿岸地区，Pizza类都可以轻易地复用，完全没有问题。

### 再回到披萨店
我们几乎完工了，只需再到加盟店短暂巡视一下，确认他们使用了正确的披萨。也需要让他们能和本地的原料工厂搭上线：
<pre class="mcode">
public class NYPizzaStore extends PizzaStore {

    protected Pizza createPizza(String item) {
        Pizza pizza = null;
        // 纽约店会用到纽约披萨原料工厂，由该原料工厂负责生产使用纽约风味披萨所需的原料
        PizzaIngredientFactory ingredientFactory = new NYPizzaIngredientFactory();

        if (item.equals("cheese")) {
            // 把工厂传递给每一个披萨，以便披萨能从工厂中取得原料
            pizza = new CheesePizza(ingredientFactory);
        } else if (item.equals("veggie")) {
            pizza = new VeggiePizza(ingredientFactory);
        } else if (item.equals("clam")) {
            pizza = new ClamPizza(ingredientFactory);
        } else if (item.equals("pepperoni")) {
            pizza = new PepperoniPizza(ingredientFactory);
        }
        return pizza;
    }
}
</pre>

### 我们做了些什么？
一连串的代码改变，我们到底做了些什么？  
我们引入新类型的工厂，也就是所谓的抽象工厂，来创建披萨原料家族。  
通过抽象工厂所提供的接口，可以创建产品的家族，利用这个接口书写代码，我们的代码将从实际工厂解耦，以便在不同环境中实现各式各样的工厂，制造出各种不同的产品。例如：不同的区域、不同的操作系统、不同的外观及操作。  
因为代码从实际的产品中解耦了，所以我们可以替换不同的工厂来取得不同的行为，例如取得大蒜番茄酱料，而不是取得番茄酱料。

### 比较工厂方法和抽象工厂
PizzaStore实现为工厂方法，因为我们需要根据区域变化创建产品。通过工厂方法，每个区域都有自己的具体工厂，他们都知道如何制作适合该区域的披萨。

因为我们需要创建一个产品家族（也就是原料），我们把PizzaIngredientFactory实现为抽象工厂。每个子类都使用其区域的供货商来实现这些原料。

工厂方法使用继承，把对象的创建委托给子类，子类实现工厂方法来创建对象。工厂方法允许类将实例化延迟到子类进行。

抽象工厂使用对象组合，对象的创建被实现在工厂接口所暴露出来的方法中。抽象工厂创建相关的对象家族，而不需要依赖它们的具体类。

所有工厂都是用来封装对象的创建。所有工厂模式都通过减少应用程序和具体类之间的依赖促进松耦合。