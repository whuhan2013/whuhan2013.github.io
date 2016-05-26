---
layout: post
title: 设计模式之工厂模式
date: 2016-5-26
categories: blog
tags: [设计模式]
description: 设计模式之工厂模式
---

### 转载  

### 本文主要包含以下内容 

1. 简单工厂模式
2. 工厂方法模式
3. 抽象工厂模式   

### 简单工厂模式


介绍简单工厂模式之前先通过一个披萨项目的例子来引出问题，然后给出简单工厂模式这种解决方案，然后随着披萨项目的不断扩展，遇到新的问题，引出工厂方法模式，然后又遇到新的问题，引出最终解决方案，抽象工厂模式。


**披萨项目介绍**

比如一个披萨店 ，店长一名，目前卖两种口味披萨，GreekPizza和CheesePizza，每个披萨都有prePare(),bake(),cut(),box()这4种步骤，原料，烘培，切割，打包，最后给用户吃。
把上述这个过程抽象后，设计如下：

![](http://upload-images.jianshu.io/upload_images/1637925-fa158e6217de0cc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Pizza披萨抽象类:

```
public abstract class Pizza {
    public abstract void prepare();
    public abstract void bake();
    public abstract void cut();
    public abstract void box();
}
```

GreekPizza披萨类:

```
public class GreekPizza  extends Pizza{
    public void prepare(){
       System.out.println("准备GreekPizza~");
    }
    public void bake(){
        System.out.println("正在烤GreekPizza~");
    }
    public void cut(){
        System.out.println("正在切GreekPizza~");
    }
    public void box(){
        System.out.println("正在打包GreekPizza~");
    }
}
```


CheesePizza披萨类:

```
public class CheesePizza extends Pizza{
   public void prepare(){
       System.out.println("准备CheesePizza~");
    }
    public void bake(){
        System.out.println("正在烤CheesePizza~");
    }
    public void cut(){
        System.out.println("正在切CheesePizza~");
    }
    public void box(){
        System.out.println("正在打包CheesePizza~");
    }
}
```

客户端,店长根据客户点的餐生成不同的披萨：

```
try{
  Pizza pizza;
  if("cheese".equal(orderType)) pizza = new CheesePizza();
  if("greek".equal(orderType)) pizza = new GreekPizza();
  }catch(Exception e){

 }
 ```

业务很简答，根据用户想买的披萨，生成不同的披萨。               
传统的设置这样也没错，如果业务发展，会造成什么问题呢？                           
现在如果多了一种口味 qiaokeliPizza，正常办法是生成一个QiaokeliPizza类，继承于Pizza，然后在OrderPizza中，添加

```
if("qiaokeli".equal(orderType)) pizza = new QiaokeliPizza();
```

如果后来披萨口味越来越多，负责点餐的店长会很不开心的，既要点餐又要做披萨，一个人忙不够来，希望请一个厨师来专门做披萨，那样他才会轻松点。          
他所想的解决方案，简单工厂模式就可以做到。

简单工厂模式
简单工厂模式是类的创建模式，又叫做静态工厂方法（Static Factory Method）模式。简单工厂模式是由一个工厂对象决定创建出哪一种产品类的实例。
简单工厂模式的结构如下：

![](http://upload-images.jianshu.io/upload_images/1637925-91f40e3c2fb23241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从图中可以看出，简单工厂模式涉及到工厂角色，抽象产品角色以及具体产品角色等三个角色：

- 工厂类（Factory）角色：担任这个角色的是工厂方法模式的核心，含有与应用紧密相关的商业逻辑。        
- 抽象产品（Product）角色：担任这个角色的类是由工厂方法模式所创建的对象的父类，或它们共同拥有的接口，这里指的就是Pizza这个类。        
- 具体产品（Concrete Product）角色：
工厂方法模式所创建的任务对象都是这个角色的实例，这里指GreekPizza和CheesePizza。
把上面的披萨项目用简单工厂模式来现实的话，无非就是创建一个工厂类（厨师）来接管店长之前要做得烤披萨的活，而店长只要告诉这个工厂类（厨师）他需要哪种披萨就好。


代码示例讲解 ：
SimplePizzaFactory简单工厂类，根据传递的参数来准备不同的披萨：

```
public class SimplePizzaFactory {
    public static Pizza CreatePizza(String orderType){
        Pizza pizza = null;
        if (orderType.equals("cheese")) {
            pizza = new CheesePizza();
        } else if (orderType.equals("greek")) {
            pizza = new GreekPizza();
        }
        return pizza;
    }
}
```

在使用时,店长只需要调用工厂类SimplePizzaFactory的静态方法CreatePizza()即可：

```
 try{
 Pizza pizza;
 pizza=SimplePizzaFactory.CreatePizza("cheese");
 pizza=SimplePizzaFactory.CreatePizza("greek");
 }catch(Exception e){
 ...
 }
 ```

这样设计后，店长就轻松多了，只要负责告诉工厂类（厨师）需要什么类型的披萨就可以，终于不要担心搞错了而负责任。

### 总结 

上面用披萨项目的列子来讲解了简单工厂模式的使用，总结下优缺点：  

**简单工厂模式的优点：**

模式的核心是工厂类。这个类含有必要的判断逻辑，可以决定在什么时候创建哪一个产品类的实例。而客户端则可以免除直接创建对象的责任（比如那个店长）。简单工厂模式通过这种做法实现了对责任的分割。

**简单工厂模式的缺点：**

这个工厂类集中了所以的创建逻辑，当有复杂的多层次等级结构时，所有的业务逻辑都在这个工厂类中实现。什么时候它不能工作了，整个系统都会受到影响。并且简单工厂模式违背了开闭原则（对扩展的开放，对修改的关闭）。



### 工厂方法模式

### 工厂方法模式概念

工厂方法模式是类的创建模式，又叫做虚拟构造子(Cirtual Constructor)模式或者多态工厂（Polymorphic Factory）模式。

工厂方法模式的用意是定义一个创建产品对象的工厂接口，将实际创建工作推迟到子类中。

首先，在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建的工作交给子类去做.这个核心类则摇身一变，成为了一个抽象工厂角色，仅负责给出具体工厂子类必须实现的接口，而不接触哪一个产品类应当被实例化这种细节。

这种进一步抽象化的结果，使这种工厂方法模式可以用来予许系统在不修改具体工厂角色的情况下引进新的产品，也就遵循了开闭原则。

### 工厂方法模式的结构 

工厂方法模式的结构图如下:

![](http://upload-images.jianshu.io/upload_images/1637925-713755ab6098c566.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


从上图可以看出， 工厂方法模式涉及到抽象工厂角色，具体工厂角色，抽象产品角色以及具体产品角色等四个角色：

- 抽象工厂角色：担任这个角色的是工厂方法模式的核心，它是与应用程序无关的。任何在模式中创建对象的工厂类必须实现这个接口。
- 具体工厂角色：担任这个角色的是实现了抽象工厂接口的具体Java类，具体工厂角色含有与应用密切相关的逻辑，并且受到应用程序的调用以创建产品对象。
- 抽象产品角色：工厂方法模式所创建的对象的超类型，也就是产品对象的共同父类或共同拥有的接口。
- 具体产品角色：这个角色实现了抽象产品角色所申明的接口。工厂方法模式所创建的每一个对象都是某个具体产品角色的实例。

结合披萨系统，用白话文来说就是之前厨师（工厂类）负责所有的烤披萨任务，太累了。于是招了两个厨师分别负责烤 GreekPizza披萨和 CheesePizza披萨，之前的厨师升级为厨师长（抽象工厂类），负责教那两位厨师（具体工厂类）烤披萨，自己则不用亲自动手烤披萨了。

附上代码前先来看看完整的类图：


![](http://upload-images.jianshu.io/upload_images/1637925-ddcf8d1248b040ef.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 代码示例讲解

抽象产品的角色Pizza，具体产品角色CheesePizza，具体产品角色GreekPizza的源代码同上  


下面是抽象工厂角色PizzaFactory的代码,这个角色是使用一个java接口实现，它声明了一个工厂方法，要求所有的具体工厂角色实现这个工厂方法:

```
 public interface PizzaFactory {
    /**
     * 工厂方法
     * @return
     */
    public Pizza createPizza();
}
```

下面是具体工厂角色CheesePizzaFactory的代码，这个角色现实了抽象工厂角色PizzaFactory所声明的工厂方法:

```
 public class CheesePizzaFactory implements PizzaFactory{
    @Override
    public Pizza createPizza() {
        return new CheesePizza();
    }
}
```

下面是具体工厂角色GreekPizzaFactory的代码，这个角色现实了抽象工厂角色PizzaFactory所声明的工厂方法:

```
public class GreekPizzaFactory  implements PizzaFactory{
    @Override
    public Pizza createPizza() {
        return new GreekPizza();
    }
}
```

下面是客户端角色的源代码：

```
public class OrderPizza {
    public static void main(String[] args){
        PizzaFactory factory=new CheesePizzaFactory();
        Pizza pizza=factory.createPizza();
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        factory=new GreekPizzaFactory();
        pizza=factory.createPizza();
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
    }
}
```

结果演示：

```
准备CheesePizza~
正在烤CheesePizza~
正在切CheesePizza~
正在打包CheesePizza~
准备GreekPizza~
正在烤GreekPizza~
正在切GreekPizza~
正在打包GreekPizza~
```

### 这里使用工厂方法模式的注意点：      
**工厂方法创建对象：**      
工厂方法不一定每一次都返还一个新的对象，但是它所返还的对象一定是它自己创建的。

**工厂方法返还的类型：**   
注意：工厂方法返还的应当是抽象类型，而不是具体类型，只有这样才能保证针对产品的多态性。当工厂方法模式发生上面的退化时，就不再是工厂方法模式了。

**工厂等级结构：**                 
工厂对象应当有一个抽象的超类型。换言之，应当有数个具体工厂类作为一个抽象超类型的具体子类存在于工厂等级结构中。如果等级结构中只有一个具体工程类的话，那么抽象工厂角色也可以省略，这时候，工厂方法模式就发生了退化，这一退化表现为针对工厂角色的多态性的丧失。

### 总结  

**工厂方法模式和简单工厂模式比较：**              
工厂方法模式跟简单工厂模式在结构上的不同是很明显的，工厂方法模式的核心是一个抽象工厂类，而简单工厂模式的核心在一个具体类。显而易见工厂方法模式这种结构更好扩展，权力下发，分布式比集中式更具优势。

如果系统需要加入一个新的产品，那么所需要的就是向系统中加入一个这个产品类以及它所对应的工厂类。没有必要修改客户端，也没有必要修改抽象工厂角色或者其他已有的具体工厂角色。对于增加新的产品类而言，这个系统完全支持开闭原则。


### 抽象工厂模式

接着上一篇工厂方法模式说，现在披萨店生意很好，除了卖披萨，又卖汉堡，并且为了适用不同的客户群体，增加了单人套餐和家庭套餐。这种情况下多了一个产品汉堡，已经不适合用工厂方法模式了，这时候就要用到更加抽象化的抽象工厂模式来满足这个系统。


### 抽象工厂模式概念
抽象工厂模式是所有形态的工厂模式中最为抽象和最具一般性的一种形态。抽象工厂模式可以向客户端提供一个接口，使得客户端在不必指定产品的具体类型的情况下，创建多个产品族中的产品对象。这就是抽象工厂的用意。

### 抽象工厂模式的结构
抽象工厂模式的简略类图如下:


![](http://upload-images.jianshu.io/upload_images/1637925-d9274032bfb17081.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



从上图可以看出， 抽象工厂模式涉及到抽象工厂角色，具体工厂角色，抽象产品角色以及具体产品角色等四个角色：

- 抽象工厂角色：担任这个角色的是工厂方法模式的核心，它是与应用程序无关的。任何在模式中创建对象的工厂类必须实现这个接口。        
- 具体工厂角色：担任这个角色的是实现了抽象工厂接口的具体Java类，具体工厂角色含有与应用密切相关的逻辑，并且受到应用程序的调用以创建产品对象。               
- 抽象产品角色：工厂方法模式所创建的对象的超类型，也就是产品对象的共同父类或共同拥有的接口。        
- 具体产品角色：这个角色实现了抽象产品角色所申明的接口。工厂方法模式所创建的每一个对象都是某个具体产品角色的实例。

为了更好地理解抽象工厂模式，我们先引入两个概念：   

**(1) 产品等级结构：**产品等级结构即产品的继承结构，如一个抽象类是电视机，其子类有海尔电视机、海信电视机、TCL电视机，则抽象电视机与具体品牌的电视机之间构成了一个产品等级结构，抽象电视机是父类，而具体品牌的电视机是其子类。

**(2) 产品族：**在抽象工厂模式中，产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品，如海尔电器工厂生产的海尔电视机、海尔电冰箱，海尔电视机位于电视机产品等级结构中，海尔电冰箱位于电冰箱产品等级结构中，海尔电视机、海尔电冰箱构成了一个产品族。

角色上跟工厂方法模式差不多。只是比之前多了一个产品汉堡，并且具体工厂类划分是按照产品族来划分的，这里划分为单人套餐（SingleFactory）以及家庭套餐（FamilyFactory）,单人套餐工厂类可以生产单人套餐披萨和单人套餐汉堡。家庭套餐工厂类可以生产家庭套餐披萨和家庭套餐汉堡。

下图所示是这个系统的产品角色的相图：

![](http://upload-images.jianshu.io/upload_images/1637925-7f8263b2d57fd562.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

附上代码前先来看看完整的类图：

![](http://upload-images.jianshu.io/upload_images/1637925-cb137cca56265c3f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


代码示例讲解
下面是抽象产品的角色Pizza的源代码：

```
public interface Pizza {
    public void create();
}
``

下面是具体产品的角色SinglePizza的源代码：

```
public class SinglePizza implements Pizza{
    @Override
    public void create() {
        System.out.println("单人套餐披萨");
    }
}
```

下面是具体产品的角色FamilyPizza的源代码：

```
public class FamilyPizza implements Pizza{
    @Override
    public void create() {
        System.out.println("家庭套餐披萨");
    }
}
```

下面是抽象产品的角色Hamburger的源代码：

```
public interface Hamburger {
    public void create();
}
```

下面是具体产品的角色SingleHamburger的源代码：

```
public class SingleHamburger  implements Hamburger{
    @Override
    public void create() {
        System.out.println("单人套餐汉堡");
    }
}
```

下面是具体产品的角色FamilyHamburger的源代码：

```
public class FamilyHamburger implements Hamburger{
    @Override
    public void create() {
        System.out.println("家庭套餐汉堡");
    }
}
```


下面是抽象工厂角色Factory的代码,这个角色是使用一个java接口实现，它声明了两个工厂方法，一个用来生产披萨，一个用来生产汉堡，并要求所有的具体工厂角色实现这个工厂方法:

```
public interface Factory {
    public Pizza createPizza();
    public Hamburger createHamburger();
}
```

下面是具体工厂角色SingleFactory的代码，这个角色现实了抽象工厂角色Factory所声明的工厂方法:

```
public class SingleFactory implements Factory{
    @Override
    public Pizza createPizza() {
        return new SinglePizza();
    }

    @Override
    public Hamburger createHamburger() {
        return new SingleHamburger();
    }
}
```

下面是具体工厂角色FamilyFactory的代码，这个角色现实了抽象工厂角色Factory所声明的工厂方法:

```
public class FamilyFactory implements Factory{
    @Override
    public Pizza createPizza() {
        return new FamilyPizza();
    }
    @Override
    public Hamburger createHamburger() {
        return new FamilyHamburger();
    }
}
```

下面是客户端角色的源代码：

```
public class OrderPizza {
    public static void main(String[] args){
        Factory factory=new SingleFactory();
        Pizza pizza=factory.createPizza();
        pizza.create();
        Hamburger hamburger=factory.createHamburger();
        hamburger.create();
        factory= new FamilyFactory();
        pizza=factory.createPizza();
        pizza.create();
        hamburger=factory.createHamburger();
        hamburger.create();
    }
}
```

结果演示：

```
单人套餐披萨
单人套餐汉堡
家庭套餐披萨
家庭套餐汉堡
```

### 实际常见的应用


![](http://upload-images.jianshu.io/upload_images/1637925-b0998b9895040a48.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 总结

**工厂方法模式和下抽象工厂模式对比**

- 工厂方法模式是一种极端情况的抽象工厂模式，而抽象工厂模式可以看成是工厂方法模式的推广。
- 工厂方法模式用来创建一个产品的等级结构，而抽象工厂模式是用来创建多个产品的等级结构。
- 工厂方法模式只有一个抽象产品类，而抽象工厂模式有多个抽象产品类。
- 工厂方法模式中具体工厂类只有一个创建方法，而抽象工厂模式中具体工厂类有多个创建方法。

到此，工厂模式中3种模式都学完了，那到底工厂模式的实现帮了我们什么？

- 系统可以在不修改具体工厂角色的情况下引进新的产品。
- 客户端不必关心对象如何创建，明确了职责。
- 更好的理解面向对象的原则，面向接口编程，而不要面向实现编程。




### 参考链接

[设计模式干货系列：（一）简单工厂模式【学习难度：★★☆☆☆，使用频率：★★★☆☆】 - 简书](http://www.jianshu.com/p/3aecc1a33fe4)

[设计模式干货系列：（二）工厂方法模式【学习难度：★★☆☆☆，使用频率：★★★★★】 - 简书](http://www.jianshu.com/p/563895db787e)

[设计模式干货系列：（三）抽象工厂模式【学习难度：★★★★☆，使用频率：★★★★★】 - 简书](http://www.jianshu.com/p/f111f06da05b)