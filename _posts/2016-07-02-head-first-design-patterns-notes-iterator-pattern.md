---
layout: post
title: "《Head first设计模式》学习笔记 - 迭代器模式"
date: 2016-07-02 22:50:59 +0800
tags: 设计模式 学习笔记 迭代器模式
excerpt: <p>迭代器模式提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。</p>
---

<div class="alert alert-success" role="alert">迭代器模式提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。</div>
爆炸性新闻：对象村餐厅和对象村煎饼屋合并了！  
真是个好消息！现在我们可以在同一个地方，享用煎饼屋美味的煎饼早餐，和好吃的餐厅午餐了。但是，好像有一点小麻烦：  
新的餐厅想用煎饼屋菜单当作早餐的菜单，使用餐厅的菜单当做午餐的菜单，大家都同意了这样实现菜单项。但是大家无法同意菜单的实现。煎饼屋使用ArrayList记录他的菜单项，而餐厅使用的是数组。他们两个都不愿意改变他们的实现，毕竟有太多代码依赖于它们了。

### 检查菜单项
让我们先检查每份菜单上的项目和实现。  
<pre class="mcode">
public class MenuItem {
    // 名称
    String name;
    // 描述
    String description;
    // 是否为素食
    boolean vegetarian;
    // 价格
    double price;

    public MenuItem(String name,
                    String description,
                    boolean vegetarian,
                    double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public double getPrice() {
        return price;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }
}
</pre>

### 两个餐厅的菜单实现 
我们先来看看两个餐厅的菜单实现
<pre class="mcode">
// 这是煎饼屋的菜单实现
public class PancakeHouseMenu {
    // 煎饼屋使用一个ArrayList存储他的菜单项
    ArrayList menuItems;

    public PancakeHouseMenu() {
        menuItems = new ArrayList();

        // 在菜单的构造器中，每一个菜单项都会被加入到ArrayList中
        // 每个菜单项都有一个名称、一个描述、是否为素食、还有价格
        addItem("K&B's Pancake Breakfast",
                "Pancakes with scrambled eggs, and toast",
                true,
                2.99);

        addItem("Regular Pancake Breakfast",
                "Pancakes with fried eggs, sausage",
                false,
                2.99);

        addItem("Blueberry Pancakes",
                "Pancakes made with fresh blueberries",
                true,
                3.49);

        addItem("Waffles",
                "Waffles, with your choice of blueberries or strawberries",
                true,
                3.59);
    }

    // 要加入一个菜单项，煎饼屋的做法是创建一个新的菜单项对象，
    // 传入每一个变量，然后将它加入ArrayList中
    public void addItem(String name, String description, 
                        boolean vegetarian, double price) {
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
        menuItems.add(menuItem);
    }

    // 这个方法返回菜单项列表
    public ArrayList getMenuItems() {
        return menuItems;
    }

    // 这里还有菜单的其他方法，这些方法都依赖于这个ArrayList，所以煎饼屋不希望重写全部的代码！
    // ...
}

// 餐厅的菜单实现
public class DinnerMenu {
    // 餐厅采用使用的是数组，所以可以控制菜单的长度，
    // 并且在取出菜单项时，不需要转型
    static final int MAX_ITEMS = 6;
    int numberOfItems = 0;
    MenuItem[] menuItems;

    public DinnerMenu() {
        menuItems = new MenuItem[MAX_ITEMS];

        // 和煎饼屋一样，餐厅使用addItem()辅助方法在构造器中创建菜单项的
        addItem("Vegetarian BLT",
                "(Fakin') Bacon with lettuce & tomato on whole wheat",
                true,
                2.99);
        addItem("BLT",
                "Bacon with lettuce & tomato on whole wheat",
                false,
                2.99);
        addItem("Soup of the day",
                "Soup of the day, with a side of potato salad",
                false,
                3.29);
        addItem("Hotdog",
                "A hot dog, with saurkraut, relish, onions, topped with cheese",
                false,
                3.05);
    }

    public void addItem(String name, String description,
                        boolean vegetarian, double price) {
        // 餐厅坚持让菜单保持在一定的长度之内
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
        if (numberOfItems >= MAX_ITEMS) {
            System.err.println("Sorry, menu is full! Can't add item to menu");
        } else {
            menuItems[numberOfItems] = menuItem;
            numberOfItems = numberOfItems + 1;
        }
    }

    // getMenuItems()返回一个菜单项的数组
    public MenuItem[] getMenuItems() {
        return menuItems;
    }

    // 正如煎饼屋那样，这里还有很多其他的菜单代码依赖于这个数组
    // ...
}
</pre>

### 两种不同的菜单表现方式，会带来什么问题？
想了解为什么有两种不同的菜单表现方式会让事情变得复杂化，让我们试着实现一个同时使用这两个菜单的客户代码。假设你已经被两个餐厅合租的新公司雇用，你的工作是创建一个Java版本的女招待，她能应对顾客的需要打印定制的菜单，甚至告诉你是否某个菜单项是素食的，而无需询问厨师。跟我们来看看这份关于女招待的规格，然后看看如何实现她。  

#### Java版本的女招待规格：

printMenu():  | 打印出菜单上的每一项  
printBreakfastMenu():  | 只打印早餐项  
printLunchMenu():  | 只打印午餐项  
printVegetarianMenu():  | 打印所有的素食菜单项  
isItemVegetarian(name):  | 指定项的名称，如果该项是素食，返回true，否则返回false  

我们先从实现printMenu()方法开始：  
1.打印每份菜单上的所有项，必须调用PancakeHouseMenu和DinnerMenu的getMenuItems()方法，来取得它们各自的菜单项。请注意，两者的返回类型是不一样的。  
<pre class="mcode">
// getMenuItems()方法看起来是一样的，但是调用所返回的结果却是不一样的类型。
// 早餐项是在一个ArrayList中，午餐项则是在一个数组中
PancakeHouseMenu pancakeHouseMenu = new PancakeHouseMenu();
ArrayList breakfastItems = pancakeHouseMenu.getMenuItems();

DinnerMenu dinnerMenu = new DinnerMenu();
MenuItem[] lunchItems = dinnerMenu.getMenuItems();
</pre>
2.现在，想要打印PancakeHouseMenu的项，我们用循环将早餐ArrayList内的项一一列出来。想要打印DinnerMenu的项目，我们用循环将数组内的项一一列出来。
<pre class="mcode">
// 现在，我们必须实现两个不同的循环，个别处理这两个不同的菜单
for (int i = 0; i < breakfastItems.size(); i++) {
    MenuItem menuItem = (MenuItem) breakfastItems.get(i);
    System.out.print(menuItem.getName() + " ");
    System.out.print(menuItem.getPrice() + " ");
    System.out.println(menuItem.getDescription() + " ");
}

for (int i = 0; i < lunchItems.length; i++) {
    MenuItem menuItem = lunchItems[i];
    System.out.print(menuItem.getName() + " ");
    System.out.print(menuItem.getPrice() + " ");
    System.out.println(menuItem.getDescription() + " ");
}
</pre>
3.实现女招待中的其他方法，做法也都和这里的方法相类似。我们总是需要处理两个菜单，并且用两个循环遍历这些项。如果还有第三家餐厅以不同的实现出现，我们就需要有三个循环。

### 下一步呢？
这两个餐厅让我们很为难，他们都不想改变自身的实现，因为意味着要重写许多代码。但是如果他们其中一人不肯退让，我们就很难办了，我们所写出来的女招待程序将难以维护、难以扩展。  
如果我们能够找到一个方法，让他们的菜单实现一个相同的接口，该有多好！这样一来，我们就可以最小化女招待代码中的具体引用，同时还有希望摆脱遍历这两个菜单所需的多个循环。  
听起来很棒！但要怎么做呢？  
如果你从本书中学到了一件事情，那就是封装变化的部分。很明显，在这里发生变化的是：由不同的集合（collection）类型所造成的遍历。但是，这能够被封装吗？让我们来看看这个想法：  
1.要遍历早餐项，我们需要使用ArrayList的size()和get()方法：
<pre class="mcode">
for (int i = 0; i < breakfastItems.size(); i++) {
    MenuItem menuItem = (MenuItem) breakfastItems.get(i);
}
</pre>
2.要遍历午餐项，我们需要使用数组的length字段和中括号：
<pre class="mcode">
for (int i = 0; i < lunchItems.length; i++) {
    MenuItem menuItem = lunchItems[i];
}
</pre>
3.现在我们创建一个对象，把它称为迭代器(Iterator)，利用它来封装“遍历集合内的每个对象的过程”。先让我们在ArrayList上试试：
<pre class="mcode">
// 我们从breakfastMenu中取得一个菜单项迭代器
Iterator iterator = breakfastMenu.createIterator();
// 当还有其他项时
while (iterator.hasNext()) {
    // 取得下一项
    MenuItem menuItem = (MenuItem) iterator.next();
}
</pre>
4.将它也在数组上试试：
<pre class="mcode">
// 这里的情况也是一样的：客户只需要调用hasNext()和next()即可，
// 而迭代器会暗中使用数组的下标
Iterator iterator = lunchMenu.createIterator();
while (iterator.hasNext()) {
    MenuItem menuItem = (MenuItem) iterator.next();
}
</pre>

### 会见迭代器模式
看起来我们对遍历的封装已经奏效了；你大概也已经猜到，这正是一个设计模式，称为迭代器模式。  
关于迭代器模式，你所需要知道的第一件事情，就是它依赖于一个名为迭代器的接口。  
<pre class="mcode">
public interface Iterator {
    // hasNext()方法返回一个布尔值，让我们知道是否还有更多的元素
    boolean hasNext();
    // next()方法返回下一个元素
    Object next();
}
</pre>
现在，一旦我们有了这个接口，就可以为各种对象集合实现迭代器：数组、列表、散列表……  
让我们继续实现这个迭代器，并将它挂钩到DinnerMenu中，看它是如何工作的。

### 用迭代器改写餐厅菜单
现在我们需要实现一个具体的迭代器，为餐厅菜单服务：   
<pre class="mcode">
public class DinnerMenuIterator implements Iterator {
    MenuItem[] items;
    // position记录当前数组遍历的位置
    int position = 0;

    // 构造器需要被传入一个菜单项的数组当做参数
    public DinnerMenuIterator(MenuItem[] items) {
        this.items = items;
    }

    // next()方法返回数组内的下一项，并递增其位置
    public Object next() {
        MenuItem menuItem = items[position];
        position = position + 1;
        return menuItem;
    }

    // hasNext()方法会检查我们是否已经取得数组内所有的元素。
    // 如果还有元素待遍历，则返回true
    public boolean hasNext() {
        if (position >= items.length || items[position] == null) {
            return false;
        } else {
            return true;
        }
    }
}
</pre>
好了，我们已经有了迭代器。现在就利用它来改写餐厅菜单：我们只需要加入一个方法创建一个DinnerMenuIterator，并将它返回给客户：  
<pre class="mcode">
public class DinnerMenu {
    static final int MAX_ITEMS = 6;
    int numberOfItems = 0;
    MenuItem[] menuItems;

    // ...
    
    // 我们不再需要getMenuItems()方法，事实上，我们根本不想要这个方法，
    // 因为它会暴露我们内部的实现。
    // 这是createIterator()方法，用来从菜单项数组创建一个DinnerMenuIterator，
    // 并将它返回给客户
    public Iterator createIterator() {
        return new DinnerMenuIterator(menuItems);
    }

    // ...
}
</pre>
现在将迭代器代码整合进女招待中。
<pre class="mcode">
public class Waitress {
    PancakeHouseMenu pancakeHouseMenu;
    DinnerMenu dinnerMenu;
    
    // 在构造器中，女招待照顾两个菜单
    public Waitress(PancakeHouseMenu pancakeHouseMenu, DinnerMenu dinnerMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinnerMenu = dinnerMenu;
    }
    
    public void printMenu() {
        // 这个printMenu()方法为每一个菜单各自创建一个迭代器
        Iterator pancakeIterator = pancakeHouseMenu.createIterator();
        Iterator dinnerIterator = dinnerMenu.createIterator();
        // 然后调用重载的printMenu()，将迭代器传入
        printMenu(pancakeIterator);
        printMenu(dinnerIterator);
    }
    
    // 这个重载的printMenu()方法，使用迭代器来遍历菜单项并打印出来
    private void printMenu(Iterator iterator) {
        while (iterator.hasNext()) {
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.getName() + " " + 
                    menuItem.getPrice() + " " + menuItem.getDescription());
        }
    }
}
</pre>

### 到目前为止，我们做了些什么？
首先，我们让对象村的厨师们非常快乐。他们可以保持他们自己的实现又可以摆平差别。只要我们给他们这两个迭代器（PancakeHouseMenuIterator和DinnerMenuIterator），他们只需要加入一个createIterator()方法，一切就大功告成了。  
这个过程中，我们也帮了我们自己。女招待将会更容易维护和扩展。让我们来彻底检查一下到底我们做了哪些事，以及后果如何：  

**难以维护的女招待实现** | **由迭代器支持的新女招待**
菜单封装得不好，餐厅使用的是ArrayList，而煎饼屋使用的是数组。 | 菜单的实现已经被封装起来了。女招待不知道菜单是如何存储菜单项集合的。
需要两个循环来遍历菜单项。 | 只要实现迭代器，我们只需要一个循环，就可以多态地处理任何项的集合。
女招待捆绑于具体类（MenuItem[]和ArrayList）。 | 女招待现在只使用一个接口（迭代器）。
女招待捆绑于两个不同的具体菜单类，尽管这两个类的接口大致上是一样的。 | 现在的菜单接口完全一样。但是，我们还是没有一个共同的接口，也就是说女招待仍然捆绑于两个具体的菜单类。这一点我们最好再修改一下。

### 做一些改良
好了，我们已经知道这两份菜单的接口完全一样，但没有为它们设计一个共同的接口。所以，接下来就要这么做，让女招待更干净一些。  
Java有一个内置的Iterator接口，让我们先来看看：
<pre class="mcode">
public interface Iterator&lt;E&gt; {
    /**
     * Returns true if there is at least one more element, false otherwise.
     * @see #next
     */
    public boolean hasNext();

    /**
     * Returns the next object and advances the iterator.
     *
     * @return the next object.
     * @throws NoSuchElementException
     *             if there are no more elements.
     * @see #hasNext
     */
    public E next();

    /**
     * Removes the last object returned by {@code next} from the collection.
     * This method can only be called once between each call to {@code next}.
     *
     * @throws UnsupportedOperationException
     *             if removing is not supported by the collection being
     *             iterated.
     * @throws IllegalStateException
     *             if {@code next} has not been called, or {@code remove} has
     *             already been called after the last call to {@code next}.
     */
    public void remove();
}
</pre>
这个接口看起来和我们之前定义的一样，只不过多了一个附加的方法，允许我们从聚合中删除由next()方法返回的最后一项。  
接下来让我们用java.util.Iterator来清理代码。  
让我们先从煎饼屋菜单开始，先把它改用java.util.Iterator，这很容易，只需要删除煎饼屋菜单迭代器，然后在煎饼屋菜单的代码前面加上 import java.util.Iterator。再改变下面这一行代码就可以了：
<pre class="mcode">
public Iterator createIterator() {
    return menuItems.iterator();
}
</pre>
这样PancakeHouseMenu就完成了。  
接着，我们处理DinnerMenu，以符合java.util.Iterator的需求。
<pre class="mcode">
public class DinnerMenuIterator implements Iterator {
    MenuItem[] items;
    int position = 0;

    public DinnerMenuIterator(MenuItem[] items) {
        this.items = items;
    }

    public Object next() {
        MenuItem menuItem = items[position];
        position = position + 1;
        return menuItem;
    }
    
    public boolean hasNext() {
        if (position >= items.length || items[position] == null) {
            return false;
        } else {
            return true;
        }
    }
    
    // 我们需要实现remove()方法。因为使用的是固定长度的数组，
    // 所以在remove()方法被调用时，我们将后面的所有元素往前移动一个位置。
    @Override
    public void remove() {
        if (position <= 0) {
            throw new IllegalStateException("You can't remove
             an item until you've done at least one next()");
        }
        if (items[position - 1] != null) {
            for (int i = position-1; i < (items.length - 1); i++) {
                items[i] = items[i + 1];
            }
            items[items.length - 1] = null;
        }
    }
}
</pre>
我们只需要给菜单一个共同的接口，然后再稍微改一下女招待。这个Menu接口相当简单：
<pre class="mcode">
public interface Menu {
    public Iterator createIterator();
}
</pre>
现在，我们需要让煎饼屋菜单类和餐厅菜单类都实现Menu接口，然后更新女招待的代码：
<pre class="mcode">
public class Waitress {
    Menu pancakeHouseMenu;
    Menu dinnerMenu;

    // 将具体菜单类改成Menu接口
    public Waitress(Menu pancakeHouseMenu, Menu dinnerMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinnerMenu = dinnerMenu;
    }

    // 以下的代码没有修改
    public void printMenu() {
        Iterator pancakeIterator = pancakeHouseMenu.createIterator();
        Iterator dinnerIterator = dinnerMenu.createIterator();
        printMenu(pancakeIterator);
        printMenu(dinnerIterator);
    }

    private void printMenu(Iterator iterator) {
        while (iterator.hasNext()) {
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.getName() + " " +
                    menuItem.getPrice() + " " + menuItem.getDescription());
        }
    }
}
</pre>
这为我们带来了什么好处？煎饼屋菜单和餐厅菜单的类，都实现了Menu接口，女招待可以利用接口（而不是具体类）引用每一个菜单对象。这样，通过“针对接口编程，而不针对实现编程”，我们就可以减少女招待和具体类之间的依赖。

### 定义迭代器模式
现在我们来看看这个模式的正式定义：
<div class="alert alert-success" role="alert">迭代器模式提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。</div>
迭代器模式让我们能游走于聚合内的每一个元素，而又不暴露内部的表示。把游走的任务放在迭代器上，而不是聚合上，这样简化了聚合的接口和实现，也让责任各得其所。  
这很有意义：这个模式给你提供了一种方法，可以顺序访问一个聚集对象中的元素，而又不用知道内部是如何表示的。你已经在前面的两个菜单实现中看到了这一点。在设计中使用迭代器的影响是明显的：如果你有一个统一的方法访问聚合中的每一个对象，你就可以编写多态的代码和这些聚合搭配使用，如同前面的printMenu()方法一样，只要有了迭代器这个方法，根本不管菜单项究竟是由数组还是由ArrayList（或者其他能创建迭代器的东西）来保存的。  
另一个对你的设计造成重要影响的，是迭代器模式把这些元素之间游走的责任交给迭代器，而不是聚合对象。这不仅让聚合的接口和实现变得更简洁，也可以让聚合更专注在它所应该专注的事情上面（也就是管理对象组合），而不必去理会遍历的事情。

### 单一责任
如果我们允许我们的聚合实现它们内部的集合，以及相关的操作和遍历的方法，又会如何？我们已经知道这会增加聚合中的方法个数，但又怎样呢？为什么这么做不好？  
想知道为什么，首先你需要认清楚，当我们允许一个类不但要完成自己的事情（管理某种聚合），还同时要担负更多的责任（例如遍历）时，我们就给了这个类两个变化的原因。两个？没错，就是两个！如果这个集合改变的话，这个类也必须改变，如果我们遍历的方式改变的话，这个类也必须跟着改变。所以，再一次地，我们的老朋友“改变”又成了我们设计原则的中心：
<p class="text-danger">设计原则：一个类应该只有一个引起变化的原因</p>
我们知道要避免类内的改变，因为修改代码很容易造成许多潜在的错误。如果有一个类具有两个改变的原因，那么这会使得将来该类的变化几率上升，而当它真的改变时，你的设计中同时有两个方面将会受到影响。  
没错，这听起来很容易，但其实做起来并不简单：区分设计中的责任，是最困难的事情之一。我们的大脑很习惯看着一大群的行为，然后将它们集中在一起，尽管他们可能属于两个或者多个不同的责任。想要成功的唯一方法，就是努力不懈地检查你的设计，随着系统的成长，随时观察有没有迹象显示某个类改变的原因超出一个。