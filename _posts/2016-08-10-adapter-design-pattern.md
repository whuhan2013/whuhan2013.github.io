---
layout: post
title: 设计模式之适配器模式
date: 2016-8-10
categories: blog
tags: [设计模式]
description: 设计模式
---


**前言**            

今天介绍适配器模式，举个生活中的例子，我们笔记本用的到充电器其实就是个适配器，笔记本电脑的工作电压是20V，而我国的家庭用电是220V，如何让20V的笔记本电脑能够在220V的电压下工作？就是靠这个充电器搞定的。

在软件开发中，有时也存在类似这种不兼容的情况，我们也可以像引入一个电源适配器一样引入一个称之为适配器的角色来协调这些存在不兼容的结构，这种设计方案即为适配器模式。

**适配器模式概念**

适配器模式(Adapter Pattern)：将一个接口转换成客户希望的另一个接口，使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

**适配器模式结构图**

适配器模式有类适配器模式和对象的适配器模式两种不同的形式。如下图所示，左边是的类的适配器模式（继承），右边是对象的适配器模式（引用）。

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p3.png)

#### 类的适配器模式                
类的适配器模式把被适配的类的API转换成目标类的API，其静态结构图如下所示。
![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p4.png)

在上图中可以看出，Adaptee类并没有simpleOperation2()方法，而客户端则期待这个方法。为使客户端能够使用Adaptee类，提供一个中间环节，即类Adapter，把Adaptee的API与Target类的API衔接起来。Adapter与Adaptee是继承关系，这决定了这个适配器模式是类的。

模式所涉及的角色有：

- 目标（Target）角色：这就是所期待得到的接口。注意，由于这里讨论的是类适配器模式，因此目标不可以是类。
- 源（Adaptee）角色：现有需要配置的接口。
- 适配器（Adapter）角色：适配器类是本模式的核心。适配器把源接口转换成目标接口。显然，这一角色不可以是接口，而必须是具体类。


#### 对象的适配器模式                       
与类的适配器模式一样，对象的适配器模式把被适配的类的API转换成目标类的API，与类的适配器模式不同的是，对象的适配器模式不是使用继承关系连接到Adaptee类，而是使用委派关系连接到Adaptee类。对象的适配器模式的静态结构如下图所示。

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p5.png)

从上图中可以看出，Adaptee类并没有simpleOperation2()方法，而客户端则期待这个方法。为使客户端能够使用Adaptee类，需要提供一个包装（Wrapper）类Adapter。这个包装类包装了一个Adaptee的实例，从而此包装类能够把Adaptee的API与Target类的API衔接起来。Adapter与Adaptee是委派关系，这决定了这个适配器模式是对象的。
从上图可以看出，模式所涉及的角色有：

- 目标（Target）角色：这就是所期待的接口，目标可以是具体的或抽象的类。
- 源（Adaptee）角色：现有需要适配的接口。
- 适配器（Adapter）角色：适配器类是本模式的核心。适配器把源接口转换成目标接口，显然，这一角色必须是具体类。


#### 代码示例
类的适配器模式代码

```
public interface Target {
    /**
     * 这是源类也有的方法simpleOperation1
     */
    void simpleOperation1();

    /**
     * 这是源类没有的方法simpleOperation2
     */
    void simpleOperation2();

}
```

上面给出的是目标角色的源代码，这个角色是以一个Java接口的形式实现的。可以看出，这个接口声明了两个方法：simpleOperation1()和simpleOperation2()。而源角色Adatpee是一个具体类，它有一个simpleOperation1()方法，但是没有simpleOperation2()方法，如下面代码清单所示。

```
public class Adaptee {
    /**
     * 源类含有方法simpleOperation1
     */
    public void simpleOperation1(){};
}
```

适配器角色Adapter扩展了Adaptee,同时又实现了目标接口。由于Adaptee没有提供simpleOperation2()方法，而目标接口又要求这个方法，因此适配器角色Adatper实现了这个方法，如下面代码清单所示。

```
public class Adapter extends Adaptee implements Target {

    /**
     * 由于源类没有方法simpleOperation2
     * 因此适配器类补充上这个方法
     */
    @Override
    public void simpleOperation2() {

    }
}
```

**类的适配器模式的效果：**

- 使用一个具体类把源（Adaptee）适配到目标（Target）中。这样一来，如果源以及源的子类都使用此类适配，就行不通了。
- 由于适配器类是源的子类，因此可以适配器类中之换掉（Override）源的一些方法。
- 由于只引进了一个适配器类，因此只有一个路线到目标类，使问题得到简化。


#### 对象的适配器模式代码

```
public interface Target {
    /**
     * 这是源类也有的方法simpleOperation1
     */
    void simpleOperation1();

    /**
     * 这是源类没有的方法simpleOperation2
     */
    void simpleOperation2();
}
```


上面给出的是目标角色的源代码，这个角色是以一个Java接口的形式实现的。可以看出，这个接口声明了两个方法：simpleOperation1()和simpleOperation2()。而源角色Adapatee是一个具体类，它有一个simpleOperation1()方法，但是没有simpleOperation2()方法，如下面带入清单所示。

```
public class Adaptee {
    /**
     * 源类含有方法simpleOperation1
     */
    public void simpleOperation1(){};
}
```

适配器类的源代码如下面代码清单所示。

```
public class Adapter implements Target {


    private Adaptee adaptee;


    public Adapter(Adaptee adaptee) {
        super();
        this.adaptee = adaptee;
    }


    /**
     * 源类有方法simpleOperation1
     * 因此适配器类直接委派即可
     */
    @Override
    public void simpleOperation1() {
        adaptee.simpleOperation1();
    }


    /**
     * 源类没有方法simpleOperation2
     * 因此适配器类补充上这个方法
     */
    @Override
    public void simpleOperation2() {
        //write you code here
    }
}
```

**对象的适配器模式的效果：**

- 一个适配器可以把多种不同的源适配到同一个目标。换言之，同一个适配器可以把源类和它的子类都适配到目标接口。
- 与类的适配器模式相比，要想置换源类的方法就不容易。如果一定要置换掉源类的一个或多个方法，就只好先做一个源类的子类，将源类的方法置换掉，然后再把源类的子类当作真正的源进行适配。
- 虽然想要置换源类的方法不容易，但是要想增加一些新的方法则方便得很，而且新增加的方法可同时适用于所有的源。


#### 总结

适配器模式将现有接口转化为客户类所期望的接口，实现了对现有类的复用，它是一种使用频率非常高的设计模式，在软件开发中得以广泛应用，在Spring等开源框架、驱动程序设计（如JDBC中的数据库驱动程序）中也使用了适配器模式。

**1. 主要优点**

无论是对象适配器模式还是类适配器模式都具有如下优点：

- 将目标类和适配者类解耦，通过引入一个适配器类来重用现有的适配者类，无须修改原有结构。
- 增加了类的透明性和复用性，将具体的业务实现过程封装在适配者类中，对于客户端类而言是透明的，而且提高了适配者的复用性，同一个适配者类可以在多个不同的系统中复用。
- 灵活性和扩展性都非常好，通过使用配置文件，可以很方便地更换适配器，也可以在不修改原有代码的基础上增加新的适配器类，完全符合“开闭原则”。

具体来说，类适配器模式还有如下优点：

- 由于适配器类是适配者类的子类，因此可以在适配器类中置换一些适配者的方法，使得适配器的灵活性更强。

对象适配器模式还有如下优点：

- 一个对象适配器可以把多个不同的适配者适配到同一个目标；
- 可以适配一个适配者的子类，由于适配器和适配者之间是关联关系，根据“里氏代换原则”，适配者的子类也可通过该适配器进行适配。


**2. 主要缺点**

类适配器模式的缺点如下：

- 对于Java、C#等不支持多重类继承的语言，一次最多只能适配一个适配者类，不能同时适配多个适配者；
- 适配者类不能为最终类，如在Java中不能为final类，C#中不能为sealed类；
- 在Java、C#等语言中，类适配器模式中的目标抽象类只能为接口，不能为类，其使用有一定的局限性。

对象适配器模式的缺点如下：

- 与类适配器模式相比，要在适配器中置换适配者类的某些方法比较麻烦。如果一定要置换掉适配者类的一个或多个方法，可以先做一个适配者类的子类，将适配者类的方法置换掉，然后再把适配者类的子类当做真正的适配者进行适配，实现过程较为复杂。

**适用场景**

在以下情况下可以考虑使用适配器模式：

- 系统需要使用一些现有的类，而这些类的接口（如方法名）不符合系统的需要，甚至没有这些类的源代码。
- 想创建一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作。
- (对对象的适配器模式而言）在设计里，需要改变多个已有的子类的接口，如果使用类的适配器模式，就要针对每一个子类做一个适配器类，而这不太实际。


![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p6.png)