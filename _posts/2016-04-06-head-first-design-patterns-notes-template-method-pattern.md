---
layout: post
title: "《Head first设计模式》学习笔记 - 模板方法模式"
date: 2016-04-06 23:45:34 +0800
tags: 设计模式 学习笔记 模板方法模式
excerpt: <p>模板方法模式在一个方法中定义了一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。</p>
---

<div class="alert alert-success" role="alert">模板方法模式在一个方法中定义了一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。</div>
有些人没有咖啡就活不下去；有些人则离不开茶。两者共同的成分是什么？当然是咖啡因了！  
但还不只这样。茶和咖啡的冲泡方式非常相似：  

#### 星巴兹咖啡冲泡法
1. 把水煮沸  
2. 用沸水冲泡咖啡  
3. 把咖啡倒进杯子  
4. 加糖和牛奶  

#### 星巴兹茶冲泡法
1. 把水煮沸  
2. 用沸水冲泡茶叶  
3. 把茶倒进杯子  
4. 加柠檬  

下面我们用代码来创建咖啡和茶：  
<pre class="mcode">
// 这是我们的咖啡类，用来煮咖啡
public class Coffee {
    // 这是我们的咖啡冲泡法
    void prepareRecipe() {
        boilWater();
        brewCoffeeGrinds();
        pourInCup();
        addSugerAndMilk();
    }

    // 煮沸水
    private void boilWater() {
        System.out.println("Boiling water");
    }

    // 冲泡咖啡
    private void brewCoffeeGrinds() {
        System.out.println("Dripping coffee through filter");
    }

    // 把咖啡倒进杯子
    private void pourInCup() {
        System.out.println("Pouring into cup");
    }

    // 加糖和奶
    private void addSugerAndMilk() {
        System.out.println("Adding Sugar and Milk");
    }
}

// 这是我们的茶类，用来煮茶
public class Tea {
    void prepareRecipe() {
        boilWater();
        steepTeaBag();
        pourInCup();
        addLemon();
    }

    // 煮沸水。这个方法和咖啡类完全一样
    private void boilWater() {
        System.out.println("Boiling water");
    }

    // 冲泡茶叶
    private void steepTeaBag() {
        System.out.println("Steeping the tea");
    }

    // 把茶倒进杯子。这个方法和咖啡类完全一样
    private void pourInCup() {
        System.out.println("Pouring into cup");
    }

    // 加柠檬
    private void addLemon() {
        System.out.println("Adding Lemon");
    }
}
</pre>
我们发现了重复的代码，这表示我们需要清理一下设计了。在这里，茶和咖啡是如此得相似，似乎我们应该将共同的部分抽取出来，放进一个基类中。

### 第一版设计
看起来这个咖啡和茶的设计相当简单，你的第一版设计，可能看起来像这样：  
<pre class="mcode">
public abstract class CaffeineBeverage {
    // prepareRecipe()方法在每个类中都不一样，所以定义成抽象方法。
    abstract void prepareRecipe();

    // 以下两个方法被两个子类所共享，所以被定义在这个超类中
    public void boilWater() {
        System.out.println("Boiling water");
    }

    public void pourInCup() {
        System.out.println("Pouring into cup");
    }
}

public class Coffee extends CaffeineBeverage {
    void prepareRecipe() {
        boilWater();
        brewCoffeeGrinds();
        pourInCup();
        addSugerAndMilk();
    }

    private void brewCoffeeGrinds() {
        System.out.println("Dripping coffee through filter");
    }

    private void addSugerAndMilk() {
        System.out.println("Adding Sugar and Milk");
    }
}

public class Tea extends CaffeineBeverage {
    void prepareRecipe() {
        boilWater();
        steepTeaBag();
        pourInCup();
        addLemon();
    }

    private void steepTeaBag() {
        System.out.println("Steeping the tea");
    }

    private void addLemon() {
        System.out.println("Adding Lemon");
    }
}
</pre>

### 更进一步的设计
以上的设计是不是忽略了某些其他的共同点？咖啡和茶之间还有什么是相似的？  
注意到两份冲泡法都采用了相同的算法：  
1. 把水煮沸  
2. 用热水泡咖啡或茶  
3. 把饮料倒进杯子  
4. 在饮料中加入适当的调料  
其中，第2步和第4步并没有被抽取出来，但他们是一样的，只是应用在了不同的饮料上。我们有办法把prepareRecipe()方法也抽象化吗？让我们先从每一个子类中逐步抽象prepareRecipe()。  

### 抽象prepareRecipe()
我们遇到的第一个问题就是，咖啡使用brewCoffeeGrinds()和addSugerAndMilk()方法，而茶使用steepTeaBag()和addLemon()方法。让我们思考这一点：浸泡(steep)和冲泡(brew)差异其实不大。所以我们给它一个新的方法名称，比方说brew()，然后不管是泡茶或者冲泡咖啡我们都用这个名称。类似地，加糖和牛奶都是在饮料中加入调料。让我们也给它一个新的方法名称：addCondiments()。这样一来，新的prepareRecipe()方法看起来像是这样：  
<pre class="mcode">
void prepareRecipe() {
    boilWater();
    brew();
    pourInCup();
    addCondiments();
}
</pre>
现在我们有了新的prepareRecipe()方法，但是需要让它能够符合代码。要想这么做，我们先从CaffeineBeverage超类开始： 
<pre class="mcode">
public abstract class CaffeineBeverage {
    // 现在，用同一个prepareRecipe()方法来处理茶和咖啡。
    // prepareRecipe()方法被声明为final，因为我们不希望子类覆盖这个方法
    // 我们将第2步和第4步泛化成为brew()和addCondiments()
    final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }

    // 因为咖啡和茶处理这些方法的做法不同，所以这两个方法必须被声明为抽象，
    // 剩余的东西留给子类去操心
    abstract void addCondiments();
    abstract void brew();

    public void boilWater() {
        System.out.println("Boiling water");
    }

    public void pourInCup() {
        System.out.println("Pouring into cup");
    }
}
</pre>
最后，我们需要处理咖啡和茶类，这两个类现在都是依赖超类来处理冲泡法，所以只需要自行处理冲泡和添加调料部分：  
<pre class="mcode">
public class Coffee extends CaffeineBeverage {
    @Override
    void brew() {
        System.out.println("Dripping coffee through filter");
    }

    @Override
    void addCondiments() {
        System.out.println("Adding Sugar and Milk");
    }
}

public class Tea extends CaffeineBeverage {
    @Override
    void brew() {
        System.out.println("Steeping the tea");
    }

    @Override
    void addCondiments() {
        System.out.println("Adding Lemon");
    }
}
</pre>

### 认识模板方法
基本上，我们刚刚实现的就是模板方法模式。咖啡因饮料类的结构包含了实际的“模板方法”：prepareRecipe()方法。为什么？
因为：  
1.毕竟它是一个方法。  
2.它用作一个算法的模板，在这个例子中，算法是用来制作咖啡因饮料的。  
在这个模板中，算法内的每一个步骤都被一个方法代表了。某些方法是由这个类（也就是超类）处理的，某些方法则是由子类处理的。需要由子类提供的方法，必须在超类中声明为抽象。  
<div class="alert alert-success" role="alert">模板方法定义了一个算法的步骤，并允许子类为一个或多个步骤提供实现。</div>

### 模板方法带给我们什么？

**不好的茶和咖啡实现** | **模板方法提供的酷炫咖啡因饮料**
Coffee和Tea主导一切，它们控制了算法。 | 由CaffeineBeverage类主导一切，它拥有算法，并且保护这个算法。
Coffee和Tea之间存在着重复的代码。 | 对子类来说，CaffeineBeverage类的存在，可以将代码的复用最大化。
对于算法所做的代码改变，需要打开子类修改许多地方。 | 算法只存在一个地方，所以很容易修改。
由于类的组织方式不具有弹性，所以加入新种类的咖啡因饮料需要做许多工作。 | 这个模板方法提供了一个框架，可以让其他的咖啡因饮料插进来。新的咖啡因饮料只需要实现自己的方法就可以了。
算法的知识和它的实现会分散在许多类中。 | CaffeineBeverage类专注在算法本身，而由子类提供完整的实现。

### 定义模板方法模式
你已经看到了茶和咖啡的例子中如何使用模板方法模式。现在，就让我们来看看这个模式的正式定义和所有的细节：  
<div class="alert alert-success" role="alert">模板方法模式在一个方法中定义了一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。</div>
这个模式是用来创建一个算法的模板。什么是模板？如你所见的，模板就是一个方法。更具体地说，这个方法将算法定义成一组步骤，其中的任何步骤都可以是抽象的，由子类负责实现。这可以确保算法的结构保持不变，同时由子类提供部分实现。  
让我们细看抽象类是如何被定义的，包括了它内含的模板方法和原语操作。
<pre class="mcode">
// 这就是我们的抽象类。它被声明为抽象，用来作为基类，其子类必须实现其操作
public abstract class AbstractClass {
    // 这就是模板方法。它被声明为final，以免子类改变这个算法的顺序。
    final void templateMethod() {
        // 模板方法定义了一连串的步骤，每个步骤由一个方法代表
        primitiveOperation1();
        primitiveOperation2();
        concreteOperation();
    }

    // 在这个范例中有两个原语操作，具体子类必须实现它们
    abstract void primitiveOperation1();
    abstract void primitiveOperation2();
    
    // 这个抽象类有一个具体的操作。
    void concreteOperation() {
    	// ...
    }
}
</pre>
现在我们要“更靠近一点”，详细看看此抽象类内可以有哪些类型的方法：  
<pre class="mcode">
public abstract class AbstractClass {
    final void templateMethod() {
        primitiveOperation1();
        primitiveOperation2();
        concreteOperation();
        // 我们加进一个新方法调用
        hook();
    }

    // 这两个方法还是和以前一样，定义成抽象，由具体的子类实现。
    abstract void primitiveOperation1();
    abstract void primitiveOperation2();

    // 这个具体的方法被定义在抽象类中。
    // 将它声明为final，这样一来子类就无法覆盖它。
    // 它可以被模板方法直接使用，或者被子类使用。
    final void concreteOperation() {
        // ...
    }

    // 我们也可以有“默认不做事的方法”，我们称这种方法为“hook”（钩子）。
    // 子类可以视情况决定要不要覆盖它们。在下面，我们就会知道钩子的实际用途
    void hook() {}
}
</pre>

### 对模板方法进行挂钩
钩子是一种被声明在抽象类中的方法，但只有空的或者默认的实现。钩子的存在，可以让子类有能力对算法的不同点进行挂钩。要不要挂钩，由子类决定。  
钩子有好几种用途，让我们先看其中一个，稍后再看其他几个：  
<pre class="mcode">
public abstract class CaffeineBeverageWithHook {
    final void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        // 我们加上了一个小小的条件语句，而该条件是否成立，
        // 是由一个具体方法customerWantsCondiments()决定的。
        // 如果顾客“想要”调料，只有这时我们才调用addCondiments()。
        if (customerWantsCondiments()) {
            addCondiments();
        }
    }

    abstract void addCondiments();
    abstract void brew();

    public void boilWater() {
        System.out.println("Boiling water");
    }

    public void pourInCup() {
        System.out.println("Pouring into cup");
    }

    // 我们在这里定义了一个方法，（通常）是空的缺省实现。这个方法只会返回true，不做别的事。
    // 这就是一个钩子，子类可以覆盖这个方法，但不见得一定要这么做。
    boolean customerWantsCondiments() {
        return true;
    }
}
</pre>
钩子有几种用法。钩子可以让子类实现算法中可选的部分，或者在钩子对于子类的实现并不重要的时候，子类可以对此钩子置之不理。钩子的另一个用法，是让子类能够有机会对模板方法中某些即将发生的（或刚刚发生的）步骤做出反应。比方说，名为justReOrderList()的钩子方法允许子类在内部列表重新组织后执行某些动作（例如在屏幕上重新显示数据）。正如你刚刚看到的，钩子也可以让子类有能力为其抽象类做一些决定。

### 好莱坞原则
我们有一个新的设计原则，称为好莱坞原则：  
<p class="text-danger">好莱坞原则：别调用（打电话给）我们，我们会调用（打电话给）你。</p>
很容易记吧，但这和OO设计又有什么关系呢？  
好莱坞原则可以给我们一种防止“依赖腐败”的方法。当高层组件依赖低层组件，而低层组件又依赖高层组件，而高层组件又依赖边侧组件，而边侧组件又依赖低层组件时，依赖腐败就发生了。在这种情况下，没有人可以轻易地搞懂系统是如何设计的。  
在好莱坞原则之下，我们允许低层组件将自己挂钩到系统上，但是高层组件会决定什么时候和怎样使用这些低层组件。换句话说，高层组件对待低层组件的方式是“别调用我们，我们会调用你”。  
好莱坞原则和模板方法之间的连接其实还算明显：当我们设计模板方法模式时，我们告诉子类，“不要调用我们，我们会调用你”。怎样才能办到呢？让我们再看一次咖啡因饮料的设计：  
1. CaffeineBeverage是我们的高层组件，它能够控制冲泡法的算法，只有在需要子类实现某个方法时，才调用子类。  
2. 饮料的客户代码只依赖CaffeineBeverage抽象，而不依赖具体的Tea或Coffee，这可以减少整个系统的依赖。  
3. Tea和Coffee子类只简单提供brew()和addCondiments()方法的实现细节。如果Tea和Coffee没有先被调用，绝对不会直接调用抽象类。