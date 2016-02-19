---
layout: post
title: "《Head first设计模式》学习笔记 - 观察者模式"
date: 2015-12-01 00:10:19 +0800
tags: 设计模式 学习笔记 观察者模式
excerpt: <p>观察者模式定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。</p>
---

<div class="alert alert-success" role="alert">观察者模式定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。</div>
客户有一个WeatherData对象，负责追踪温度、湿度和气压等数据。现在客户给我们提了个需求，让我们利用WeatherData对象取得数据，并更新三个布告板：目前状况、气象统计和天气预报。  
WeatherData对象提供了4个接口：  
getTemperature()：获取温度  
getHumidity()：获取湿度  
getPressure()：获取气压  
measurementsChanged()：一旦气象测量更新，此方法会被调用  
我们的工作是实现measurementsChanged()，好让它更新目前状况、气象统计、天气预报的显示布告板。  

###认识观察者模式
先看看报纸和杂志的订阅是怎么回事：  
1. 报社的业务就是出版报纸；  
2. 向某家报社订阅报纸，只要他们有新报纸出版，就会给你送来。只要你是他们的订户，你就会一直收到新报纸；  
3. 当你不想再看报纸的时候，取消订阅，他们就不会再送新报来；  
4. 只要报社还在运营，就会有人向他们订阅报纸或取消订阅报纸。  
观察者模式与订阅报纸类似，出版者改称为“主题”（Subject），订阅者改称为“观察者”（Observer）。  
主题对象管理某些数据。当主题内的数据改变，就会通知观察者。已经订阅了主题的观察者会在主题数据发送改变时收到通知。如果观察者不想接收新的数据，可以取消订阅，之后主题数据改变时就不会收到通知。  

###实现观察者模式
实现观察者模式的方法不只一种，但是以包含Subject与Observer接口的类设计的做法最常见。  
<pre class="mcode">
// 这是主题接口，对象使用此接口注册为观察者，或者把自己从观察者中删除
public interface Subject{
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    void notifyObservers();
}

// 一个具体的主题总是实现主题接口，除了注册和撤销方法之外，具体主题还实现了notifyObservers，此方法用于在状态改变时更新所有当前观察者
public class ConcreteSubject implements Subject {
    @Override
    public void registerObserver(Observer o) {
    }

    @Override
    public void removeObserver(Observer o) {
    }

    @Override
    public void notifyObservers() {
    }
}

// 所有潜在的观察者必须实现观察者接口，这个接口只有update()一个方法，当主题状态改变时它被调用
public interface Observer {
    void update();
}

// 具体的观察者可以是实现此接口的任意类。观察者必须注册具体主题，以便接收更新
public class ConcreteObserver implements Observer {
    @Override
    public void update() {
    }
}
</pre>

###松耦合的威力
观察者模式提供了一种对象设计，让主题和观察者之间松耦合。  
当两个对象之间松耦合，它们依然可以交互，但是不太清楚彼此的细节。  
关于观察者的一切，主题只知道观察者实现了某个接口（也就是Observer接口）。主题不需要知道观察者具体类是谁、做了些什么或其他任何细节。  
主题唯一依赖的东西是一个实现Observer接口的对象列表，所以我们可以随时增加观察者和删除观察者。  
有新类型的观察者出现时，主题的代码不需要修改。当有新的具体类需要当观察者，我们不需要为了兼容新类型而修改主题的代码，所有要做的就是在新的类里实现此观察者接口，然后注册为观察者即可。  
我们可以独立地复用主题或观察者。如果我们在其他地方需要使用主题或观察者，可以轻易地复用，因为两者并非紧耦合。  
<p class="text-danger">设计原则：为了交互对象之间的松耦合设计而努力。</p>
松耦合的设计之所以能让我们建立有弹性的OO系统，能够应对变化，是因为对象之间的互相依赖降低到了最低。

###利用观察者模式设计气象站系统
我们可以把WeatherData对象当作主题，把布告板当作观察者。布告板为了取得信息，就必须先向WeatherData对象注册。一旦WeatherData知道有某个布告板的存在，就会适时地调用布告板的某个方法来告诉布告板观测值是多少。每个布告板都应该有一个大概名为update()的方法，以供WeatherData对象调用。  
先从建立接口开始：  
<pre class="mcode">
public interface Subject{
    // registerObserver和removeObserver都需要一个观察者作为变量，该观察者是用来注册或被删除的
    void registerObserver(Observer o);
    void removeObserver(Observer o);
    // 当主题状态改变时。这个方法会被调用，以通知所有的观察者
    void notifyObservers();
}

public interface Observer {
    // 当气象观测值改变时，主题会把温度、湿度、气压的状态值当作方法的参数，传送给观察者
    public void update(float temp, float humidity, float pressure);
}

public interface DisplayElement {
    // 当布告板需要显示时，调用此方法
    public void display();
}
</pre>
实现WeatherData类：
<pre class="mcode">
// WeatherData现在实现了Subject接口
public class WeatherData implements Subject {
    // 我们加上一个ArrayList来记录观察者，此ArrayList是在构造器中建立的。
    public ArrayList observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList();
    }

    // 当注册观察者时，我们只要把它加到ArrayList的后面即可
    @Override
    public void registerObserver(Observer o) {
        observers.add(o);
    }

    // 当观察者想取消注册，我们把它从ArrayList中删除即可
    @Override
    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    // 在这里，我们把状态告诉每一个观察者
    // 因为观察者都实现了update()，所以我们知道如何通知它们
    @Override
    public void notifyObservers() {
        for (int i = 0; i < observers.size(); i++) {
            Observer observer = (Observer) observers.get(i);
            observer.update(temperature, humidity, pressure);
        }
    }
    
    // 当从气象站得到更新观测值时，我们通知观察者
    public void measurementsChanged(){
        notifyObservers();
    }
    
    public void setMeasurements(float temperature, float humidity, float pressure){
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
}
</pre>
实现目前状况布告板：
<pre class="mcode">
// 此布告板实现了Observer接口，所以可以从WeatherData对象中获得改变
// 它也实现了DisplayElement接口，因为我们的API规定所有的布告板都必须实现此接口
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    private Subject weatherData;
    
    // 构造器需要WeatherData对象（也就是主题）作为注册之用
    public CurrentConditionsDisplay(Subject weatherData){
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }
    
    // display()方法就是把最近的温度和湿度显示出来
    @Override
    public void display() {
        System.out.println("Cuttent conditions:" + temperature + "F degrees and " + humidity + "% humidity");
    }

    // 当update()被调用时，我们把温度和湿度保存起来，然后调用display()
    @Override
    public void update(float temp, float humidity, float pressure) {
        this.temperature = temp;
        this.humidity = humidity;
        display();
    }
}
</pre>
测试：
<pre class="mcode">
public class WeatherStation {
    public static void main(String[] args){
        // 建立WeatherData对象
        WeatherData weatherData = new WeatherData();

        // 建立目前状况布告板
        CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);

        // 模拟新的气象测量
        weatherData.setMeasurements(80, 65, 30.4f);
        weatherData.setMeasurements(82, 70, 29.2f);
        weatherData.setMeasurements(78, 90, 29.2f);
    }
}
</pre>