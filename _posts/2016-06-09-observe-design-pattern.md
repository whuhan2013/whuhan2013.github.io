---
layout: post
title: 设计模式之观察者模式
date: 2016-6-9
categories: blog
tags: [设计模式]
description: 设计模式之观察者模式
---

**观察者模式的定义** 

定义了对象之间的一对多的依赖，这样一来，当一个对象改变时，它的所有的依赖者都会收到通知并自动更新。
好了，对于定义的理解总是需要实例来解析的，如今的微信服务号相当火啊，下面就以微信服务号为背景，给大家介绍观察者模式。
看一张图：

![](http://img.blog.csdn.net/20140420130751265?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


如上图所示，服务号就是我们的主题，使用者就是观察者。现在我们明确下功能：        
1、服务号就是主题，业务就是推送消息                  
2、观察者只需要订阅主题，只要有新的消息就会送来             
3、当不想要此主题消息时，取消订阅                     
4、只要服务号还在，就会一直有人订阅                
好了，现在我们来看看观察者模式的类图：       


![](http://img.blog.csdn.net/20140420131355500?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


### 实例  

首先开始写我们的主题接口，和观察者接口：

```
package com.zhy.pattern.observer;

/**
 * 主题接口，所有的主题必须实现此接口
 * 
 * @author zhy
 * 
 */
public interface Subject
{
    /**
     * 注册一个观察着
     * 
     * @param observer
     */
    public void registerObserver(Observer observer);

    /**
     * 移除一个观察者
     * 
     * @param observer
     */
    public void removeObserver(Observer observer);

    /**
     * 通知所有的观察着
     */
    public void notifyObservers();

}
```


```
package com.zhy.pattern.observer;

/**
 * @author zhy 所有的观察者需要实现此接口
 */
public interface Observer
{
    public void update(String msg);

}
```


接下来3D服务号的实现类：


```
package com.zhy.pattern.observer;

import java.util.ArrayList;
import java.util.List;

public class ObjectFor3D implements Subject
{
    private List<Observer> observers = new ArrayList<Observer>();
    /**
     * 3D彩票的号码
     */
    private String msg;

    @Override
    public void registerObserver(Observer observer)
    {
        observers.add(observer);
    }

    @Override
    public void removeObserver(Observer observer)
    {
        int index = observers.indexOf(observer);
        if (index >= 0)
        {
            observers.remove(index);
        }
    }

    @Override
    public void notifyObservers()
    {
        for (Observer observer : observers)
        {
            observer.update(msg);
        }
    }

    /**
     * 主题更新消息
     * 
     * @param msg
     */
    public void setMsg(String msg)
    {
        this.msg = msg;
        
        notifyObservers();
    }

}
```


模拟两个使用者：


```
package com.zhy.pattern.observer;

public class Observer1 implements Observer
{

    private Subject subject;

    public Observer1(Subject subject)
    {
        this.subject = subject;
        subject.registerObserver(this);
    }

    @Override
    public void update(String msg)
    {
        System.out.println("observer1 得到 3D 号码  -->" + msg + ", 我要记下来。");
    }

}
```


```
package com.zhy.pattern.observer;

public class Observer2 implements Observer
{
    private Subject subject ; 
    
    public Observer2(Subject subject)
    {
        this.subject = subject  ;
        subject.registerObserver(this);
    }
    
    @Override
    public void update(String msg)
    {
        System.out.println("observer2 得到 3D 号码 -->" + msg + "我要告诉舍友们。");
    }
    
    

}
```



可以看出：服务号中维护了所有向它订阅消息的使用者，当服务号有新消息时，通知所有的使用者。整个架构是一种松耦合，主题的实现不依赖与使用者，当增加新的使用者时，主题的代码不需要改变；使用者如何处理得到的数据与主题无关；




对于JDK或者Andorid中都有很多地方实现了观察者模式，比如XXXView.addXXXListenter ， 当然了 XXXView.setOnXXXListener不一定是观察者模式，因为观察者模式是一种一对多的关系，对于setXXXListener是1对1的关系，应该叫回调。

恭喜你学会了观察者模式，上面的观察者模式使我们从无到有的写出，当然了java中已经帮我们实现了观察者模式，借助于java.util.Observable和java.util.Observer。
下面我们使用Java内置的类实现观察者模式：

首先是一个3D彩票服务号主题：


```
package com.zhy.pattern.observer.java;

import java.util.Observable;

public class SubjectFor3d extends Observable
{
    private String msg ; 
    
    
    public String getMsg()
    {
        return msg;
    }


    /**
     * 主题更新消息
     * 
     * @param msg
     */
    public void setMsg(String msg)
    {
        this.msg = msg  ;
        setChanged();
        notifyObservers();
    }
}
```          


下面是一个双色球的服务号主题：

```
package com.zhy.pattern.observer.java;

import java.util.Observable;

public class SubjectForSSQ extends Observable
{
    private String msg ; 
    
    
    public String getMsg()
    {
        return msg;
    }


    /**
     * 主题更新消息
     * 
     * @param msg
     */
    public void setMsg(String msg)
    {
        this.msg = msg  ;
        setChanged();
        notifyObservers();
    }
}
```

最后是我们的使用者：


```
package com.zhy.pattern.observer.java;

import java.util.Observable;
import java.util.Observer;

public class Observer1 implements Observer
{

    public void registerSubject(Observable observable)
    {
        observable.addObserver(this);
    }

    @Override
    public void update(Observable o, Object arg)
    {
        if (o instanceof SubjectFor3d)
        {
            SubjectFor3d subjectFor3d = (SubjectFor3d) o;
            System.out.println("subjectFor3d's msg -- >" + subjectFor3d.getMsg());
        }

        if (o instanceof SubjectForSSQ)
        {
            SubjectForSSQ subjectForSSQ = (SubjectForSSQ) o;
            System.out.println("subjectForSSQ's msg -- >" + subjectForSSQ.getMsg());
        }
    }
}
```



看一个测试代码：


```
package com.zhy.pattern.observer.java;

public class Test
{
    public static void main(String[] args)
    {
        SubjectFor3d subjectFor3d = new SubjectFor3d() ;
        SubjectForSSQ subjectForSSQ = new SubjectForSSQ() ;
        
        Observer1 observer1 = new Observer1();
        observer1.registerSubject(subjectFor3d);
        observer1.registerSubject(subjectForSSQ);
        
        
        subjectFor3d.setMsg("hello 3d'nums : 110 ");
        subjectForSSQ.setMsg("ssq'nums : 12,13,31,5,4,3 15");
        
    }
}
```


测试结果


```
subjectFor3d's msg -- >hello 3d'nums : 110 
subjectForSSQ's msg -- >ssq'nums : 12,13,31,5,4,3 15
```

可以看出，使用Java内置的类实现观察者模式，代码非常简洁，对了addObserver,removeObserver,notifyObservers都已经为我们实现了，所有可以看出Observable（主题）是一个类，而不是一个接口，基本上书上都对于Java的如此设计抱有反面的态度，觉得Java内置的观察者模式，违法了面向接口编程这个原则，但是如果转念想一想，的确你拿一个主题在这写观察者模式（我们自己的实现），接口的思想很好，但是如果现在继续添加很多个主题，每个主题的ddObserver,removeObserver,notifyObservers代码基本都是相同的吧，接口是无法实现代码复用的，而且也没有办法使用组合的模式实现这三个方法的复用，所以我觉得这里把这三个方法在类中实现是合理的。


### 参考链接

[设计模式 观察者模式 以微信公众服务为例 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/24179699)