---
layout: post
title: "《Head first设计模式》学习笔记 - 迭代器模式"
date: 2016-04-11 22:06:22 +0800
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