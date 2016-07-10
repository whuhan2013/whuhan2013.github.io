---
layout: post
title: 设计模式之单例模式
date: 2016-7-10
categories: blog
tags: [设计模式]
description: 设计模式
---


**单例模式概念**               

单例模式(Singleton Pattern)：确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种对象创建型模式。                          

**单例模式结构图**              
单例模式是结构最简单的设计模式，在它的核心结构中只包含一个被称为单例类的特殊类。    

单例模式有三个特性：

- 单例类只能有一个实例
- 单例类必须自行创建自己的唯一的实例
- 单例类必须给所有其他对象提供这一实例                     

单例模式结构如图所示：

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p1.gif)

Singleton（单例）：在单例类的内部实现只生成一个实例，同时它提供一个静态的getInstance()工厂方法，让客户可以访问它的唯一实例；为了防止在外部对其实例化，将其构造函数设计为私有；在单例类内部定义了一个Singleton类型的静态对象，作为外部共享的唯一实例。


### 单例模式的几种实现方式        

**懒汉式，线程不安全**               
是否 Lazy 初始化：是          
是否多线程安全：否                     
实现难度：易                 
描述：这种方式是最基本的实现方式，这种实现最大的问题就是不支持多线程。因为没有加锁 synchronized，所以严格意义上它并不算单例模式。                             
这种方式 lazy loading 很明显，不要求线程安全，在多线程不能正常工作。                 

代码实例：

```
public class Singleton {
    private static Singleton instance;
    private Singleton (){}
    public static Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
    }
}
```

接下来介绍的几种实现方式都支持多线程，但是在性能上有所差异。


**懒汉式，线程安全**                 
是否 Lazy 初始化：是   
是否多线程安全：是            
实现难度：易                      
描述：这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。       
优点：第一次调用才初始化，避免内存浪费。        
缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。             
getInstance() 的性能对应用程序不是很关键（该方法使用不太频繁）。              


代码实例：

```
public class Singleton {
    private static Singleton instance;
    private Singleton (){}
    public static synchronized Singleton getInstance() {
    if (instance == null) {
        instance = new Singleton();
    }
    return instance;
    }
}
```             

**饿汉式**            
是否 Lazy 初始化：否            
是否多线程安全：是              
实现难度：易                                
描述：这种方式比较常用，但容易产生垃圾对象。                 
优点：没有加锁，执行效率会提高。                        
缺点：类加载时就初始化，浪费内存。                  
它基于classloder机制避免了多线程的同步问题，不过，instance在类装载时就实例化，虽然导致类装载的原因有很多种，在单例模式中大多数都是调用getInstance方法，                   但是也不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化instance显然没有达到lazy loading的效果。   

代码实例

```
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
}
```

**双检锁/双检查锁（DCL，即 double-checked locking)**          

JDK 版本：JDK1.5 起          
是否 Lazy 初始化：是           
是否多线程安全：是           
实现难度：较复杂              
描述：这种方式称为双重检查锁(Double-Check Locking)，需要注意的是，如果使用双重检查锁定来实现懒汉式单例类，需要在静态成员变量instance之前增加修饰符volatile，被volatile修饰的成员变量可以确保多个线程都能够正确处理，且该代码只能在JDK 1.5及以上版本中才能正确执行。由于volatile关键字会屏蔽Java虚拟机所做的一些代码优化，可能会导致系统运行效率降低，因此即使使用双重检查锁定来实现单例模式也不是一种完美的实现方式。            

```
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```

**静态内部类**         
是否 Lazy 初始化：是           
是否多线程安全：是           
实现难度：一般              
描述：饿汉式单例类不能实现延迟加载，不管将来用不用始终占据内存；懒汉式单例类线程安全控制烦琐，而且性能受影响。可见，无论是饿汉式单例还是懒汉式单例都存在这样那样的问题，有没有一种方法，能够将两种单例的缺点都克服，而将两者的优点合二为一呢？答案是：Yes！下面我们来学习这种更好的被称之为Initialization Demand Holder (IoDH)的技术。

在IoDH中，我们在单例类中增加一个静态(static)内部类，在该内部类中创建单例对象，再将该单例对象通过getInstance()方法返回给外部使用。由于静态单例对象没有作为Singleton的成员变量直接实例化，因此类加载时不会实例化Singleton，第一次调用getInstance()时将加载内部类SingletonHolder，在该内部类中定义了一个static类型的变量instance，此时会首先初始化这个成员变量，由Java虚拟机来保证其线程安全性，确保该成员变量只能初始化一次。由于getInstance()方法没有任何线程锁定，因此其性能不会造成任何影响。通过使用IoDH，我们既可以实现延迟加载，又可以保证线程安全，不影响系统性能，不失为一种最好的Java语言单例模式实现方式**（其缺点是与编程语言本身的特性相关，很多面向对象语言不支持IoDH）

```
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}
```

**枚举**     

JDK 版本：JDK1.5 起           
是否 Lazy 初始化：否            
是否多线程安全：是           
实现难度：易        

描述：这种实现方式还没有被广泛采用，但这是实现单例模式的最佳方法。它更简洁，自动支持序列化机制，绝对防止多次实例化。
这种方式是Effective Java作者Josh Bloch提倡的方式，它不仅能避免多线程同步问题，而且还自动支持序列化机制，防止反序列化重新创建新的对象，绝对防止多次实例化。不过，由于 JDK1.5 之后才加入 enum 特性，用这种方式写不免让人感觉生疏，在实际工作中，也很少用。
不能通过reflection attack来调用私有构造方法

```
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}
```

经验之谈：一般情况下，不建议使用第 1 种和第 2 种懒汉方式，建议使用第 3 种饿汉方式。只有在要明确实现lazy loading效果时，才会使用第 5 种登记方式。如果涉及到反序列化创建对象时，可以尝试使用第 6 种枚举方式。如果有其他特殊的需求，可以考虑使用第 4 种双检锁方式。


**Java中的语言中的单例模式**         
Java语言中就有很多单例模式的应用实例，这里举例一个。

Java的Runtime对象       

在Java语言内部，java.lang.Runtime对象就是一个使用单例模式的例子。在每一个Java应用程序里面，都有唯一的一个Runtime对象，应用程序可以与其运行环境发生相互作用。
Runtime类提供一个静态工厂方法getRuntime():

public static Runtime getRuntime();         
通过调用此方法，可以获得Runtime类唯一的一个实例：            

Runtime rt=Runtime.getRuntime();


### 总结              
单例模式作为一种目标明确、结构简单、理解容易的设计模式，在软件开发中使用频率相当高，在很多应用软件和框架中都得以广泛应用。

**1.主要优点**                      
单例模式的主要优点如下：

单例模式提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它。
由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象单例模式无疑可以提高系统的性能.      
允许可变数目的实例。基于单例模式我们可以进行扩展，使用与单例控制相似的方法来获得指定个数的对象实例，既节省系统资源，又解决了单例单例对象共享过多有损性能的问题。        

**2.主要缺点**               

单例模式的主要缺点如下：

由于单例模式中没有抽象层，因此单例类的扩展有很大的困难。
单例类的职责过重，在一定程度上违背了“单一职责原则”。因为单例类既充当了工厂角色，提供了工厂方法，同时又充当了产品角色，包含一些业务方法，将产品的创建和产品的本身的功能融合到一起。         

现在很多面向对象语言(如Java、C#)的运行环境都提供了自动垃圾回收的技术，因此，如果实例化的共享对象长时间不被利用，系统会认为它是垃圾，会自动销毁并回收资源，下次利用时又将重新实例化，这将导致共享的单例对象状态的丢失。

**3.适用场景**               
在以下情况下可以考虑使用单例模式：

系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或资源管理器，或者需要考虑资源消耗太大而只允许创建一个对象。
客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。

**references**   

[设计模式干货系列：（四）单例模式](http://www.jianshu.com/p/d8bf5d08a147)








