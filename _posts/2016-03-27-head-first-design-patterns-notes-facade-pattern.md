---
layout: post
title: "《Head first设计模式》学习笔记 - 外观模式"
date: 2016-03-27 17:16:12 +0800
tags: 设计模式 学习笔记 外观模式
excerpt: <p>外观模式提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。</p>
---

<div class="alert alert-success" role="alert">外观模式提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。</div>
我们已经知道适配器模式是如何将一个类的接口转换成另一个符合客户期望的接口的。现在我们要看一个改变接口的新模式，但是它改变接口的原因是为了简化接口。这个模式被巧妙地命名为外观模式，之所以这么称呼，是因为它将一个或数个类的复杂的一切都隐藏在背后，只显露出一个干净美好的外观。

### 甜蜜的家庭影院
在我们进入外观模式的细节之前，让我们看一个风行全美的热潮：建立自己的家庭影院。  
通过一番研究比较，你组装了一套杀手级的系统，内含DVD播放器、投影机、自动屏幕、环绕立体声，甚至还有爆米花机。  
你花了好几个星期布线、挂上投影机、连接所有的装置并进行微调。现在，你准备开始享受一部电影……  
挑选一部DVD影片，放松，准备开始感受电影的魔幻魅力。哎呀！忘了一件事：想看电影，必须先执行一些任务：  
1. 打开爆米花机；  
2. 开始爆米花；  
3. 将灯光调暗；  
4. 放下屏幕；  
5. 打开投影机；  
6. 将投影机的输入切换到DVD；  
7. 将投影机设置在宽屏模式；  
8. 打开功放；  
9. 将功放的输入设置为DVD；  
10. 将功放设置为环绕立体声；  
11. 将功放音量调到中；  
12. 打开DVD播放器；  
13. 开始播放DVD。  

让我们将这些任务写成类和方法的调用：  
<pre class="mcode">
// 打开爆米花机，开始爆米花
popper.on();
popper.pop();

// 灯光调到10%的亮度
lights.dim(10);

// 把屏幕放下来
screen.down();

// 打开投影机，并将它设置在宽屏模式
projector.on();
projector.setInput(dvd);
projector.wideScreenMode();

// 打开功放，设置为DVD，调整成环绕立体声模式，音量调到5
amp.on();
amp.setDvd(dvd);
amp.setSurroundSound();
amp.setVolume(5);

// 打开DVD机，“终于”可以看电影了！
dvd.on();
dvd.play(movies);
</pre>
但还不只这样：  
看完电影后，你还要把一切都关掉，怎么办？难道要反向地把这一切动作再进行一次？  
如果要听CD或者广播，难道也会这么麻烦？  
如果你决定要升级你的系统，可能还必须重新学习一套稍微不同的操作过程。  
怎么办？使用你的家庭影院竟变得如此复杂！让我们看看外观模式如何解决这团混乱，好让你能轻易地享受电影。

### 构造家庭影院外观
你需要的正是一个外观：有了外观模式，通过实现一个提供更合理的接口的外观类，你可以将一个复杂的子系统变得容易使用。如果你需要复杂子系统的强大威力，别担心，还是可以使用原来的复杂接口的；但如果你需要的是一个方便使用的接口，那就使用外观。  
现在是为家庭影院系统创建一个外观的时候了，于是我们了创建一个名为HomeTheaterFacade的新类，它对外暴露出几个简单的方法，例如watchMovie()。  
这个外观类将家庭影院的诸多组件视为一个系统，通过调用这个子系统，来实现watchMovie()方法。  
现在，你的客户代码可以调用此家庭影院外观所提供的方法，而不必再调用这个子系统的方法。所以，想要看电影，我们只要调用一个方法（也就是watchMovie()）就可以了。灯光、DVD播放器、投影机、功放、屏幕、爆米花，一口气全部搞定。  
外观只是提供你更直接的操作，并未将原来的子系统阻隔起来。如果你需要子系统类的更高层功能，还是可以使用原来的子系统。  
让我们逐步构造家庭影院外观：第一步是使用组合让外观能够访问子系统中所有的组件。  
<pre class="mcode">
public class HomeTheaterFacade {
    // 这就是组合：我们会用到的子系统组件全部在这里
    Amplifier amp;
    Tuner tuner;
    DvdPlayer dvd;
    CdPlayer cd;
    Projector projector;
    TheaterLights lights;
    Screen screen;
    PopcornPopper popper;

    // 外观将子系统中每一个组件的引用都传入它的构造器中。
    // 然后外观把它们赋值给相应的实例变量
    public HomeTheaterFacade(Amplifier amp,
                             Tuner tuner,
                             DvdPlayer dvd,
                             CdPlayer cd,
                             Projector projector,
                             TheaterLights lights,
                             Screen screen,
                             PopcornPopper popper) {
        this.amp = amp;
        this.tuner = tuner;
        this.dvd = dvd;
        this.cd = cd;
        this.projector = projector;
        this.lights = lights;
        this.screen = screen;
        this.popper = popper;
    }
}
</pre>
现在该是时候将子系统的组件整合成一个统一的接口了。让我们实现watchMovie()和endMovie()方法。
<pre class="mcode">
public void watchMovie(String movie){
    // watchMovie()将我们之前手动进行的每项任务依次处理。
    // 请注意，每项任务都是委托子系统中相应的组件处理的。
    System.out.println("Get ready to watch a movie");
    popper.on();
    popper.pop();
    lights.dim(10);
    screen.down();
    projector.on();
    projector.wideScreenMode();
    amp.on();
    amp.setDvd(dvd);
    amp.setSurroundSound();
    amp.setVolume(5);
    dvd.on();
    dvd.play(movie);
}

public void endMovie(){
    // endMovie()负责关闭一切。每项任务也是委托子系统中合适的组件处理的。
    System.out.println("Shutting movie theater down");
    popper.off();
    lights.on();
    screen.up();
    projector.off();
    amp.off();
    dvd.stop();
    dvd.eject();
    dvd.off();
}
</pre>
现在，让我们用轻松的方式去观赏电影：  
<pre class="mcode">
HomeTheaterFacade homeTheaterFacade = new HomeTheaterFacade(amp, tuner, dvd, cd, projector, lights, screen, popper);
homeTheaterFacade.watchMovie("Pulp Fiction");
homeTheaterFacade.endMovie();
</pre>

### 定义外观模式
想要使用外观模式，我们创建了一个接口简化而统一的类，用来包装子系统中一个或多个复杂的类。外观模式相当直接，很容易理解，这方面和许多其他的模式不太一样。但这并不会降低它的威力：外观模式允许我们让客户和子系统之间避免紧耦合。  
外观模式提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让子系统更容易使用。  
现在我们来看一个新的OO原则：  
<p class="text-danger">最少知识原则：只和你的密友谈话。</p>
这到底是什么意思？这是说，当你正在设计一个系统，不管是任何对象，你都要注意它所交互的类有哪些，并注意它和这些类是如何交互的。  
这个原则希望我们在设计中，不要让太多的类耦合在一起，免得修改系统中一部分，会影响到其他部分。如果许多类之间相互依赖，那么这个系统会变成一个易碎的系统，它需要花许多成本维护，也会因为太复杂而不容易被其他人了解。  
究竟要怎样才能避免这样呢？这个原则提供了一些方针：就任何对象而言，在该对象的方法内，我们只应该调用属于以下范围的方法：  
1. 该对象本身；  
2. 被当做方法的参数而传递进来的对象；  
3. 此方法所创建或实例化的任何对象；  
4. 对象的任何组件。  
这听起来有点严厉，不是吗？如果调用从另一个调用中返回的对象的方法，会有什么害处呢？如果我们这样做，相当于向另一个对象的子部分发请求（而增加我们直接认识的对象数目）。在这种情况下，原则要我们改为要求该对象为我们做出请求，这么一来，我们就不需要认识该对象的组件了（让我们的朋友圈子维持在最小的状态）。比方说：  
<pre class="mcode">
// 这里，我们从气象站取得了温度计对象，然后再从温度计对象取得温度
public float getTemp(){
    Thermometer thermometer = station.getThermometer();
    return thermometer.getTemperature();
}

// 在应用最少知识原则时，我们在气象站中加进一个方法，用来向温度计请求温度。
// 这可以减少我们所依赖的类的数目
public float getTemp(){
    return station.getTemperature();
}
</pre>
