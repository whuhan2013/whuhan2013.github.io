---
layout: post
title: 设计模式之建造者模式
date: 2016-7-12
categories: blog
tags: [设计模式]
description: 设计模式
---


**建造者模式概念**   

建造者模式(Builder Pattern)：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。建造者模式是一种对象创建型模式。

**产品的内部表象**

一个产品常有不同的组成成分作为产品的零件，这些零件有可能是对象，也有可能不是对象，它们通常又叫做产品的内部表象（internal representation）。不同的产品可以有不同的内部表象，也就是不同的零件。使用建造模式可以使客户端不需要知道所生成的产品有哪些零件，每个产品的对应零件彼此有何不同，是怎么建造出来的，以及怎么组成产品。

**对象性质的建造**

有些情况下，一个对象会有一些重要的性质，在它们没有恰当的值之前，对象不能作为一个完整的产品使用。比如，一个电子邮件有发件人地址、收件人地址、主题、内容、附录等部分，而在最起码的收件人地址得到赋值之前，这个电子邮件不能发送。

有些情况下，一个对象的一些性质必须按照某个顺序赋值才有意义。在某个性质没有赋值之前，另一个性质则无法赋值。这些情况使得性质本身的建造涉及到复杂的商业逻辑。这时候，此对象相当于一个有待建造的产品，而对象的这些性质相当于产品的零件，建造产品的过程是建造零件的过程。由于建造零件的过程很复杂，因此，这些零件的建造过程往往被“外部化”到另一个称做建造者的对象里，建造者对象返还给客户端的是一个全部零件都建造完毕的产品对象。

建造模式利用一个导演者对象和具体建造者对象一个个地建造出所有的零件，从而建造出完整的产品对象。建造者模式将产品的结构和产品的零件的建造过程对客户端隐藏起来，把对建造过程进行指挥的责任和具体建造者零件的责任分割开来，达到责任划分和封装的目的。

**建造者模式结构图**        

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p2.jpg)


在建造者模式结构图中包含如下几个角色：

- Builder（抽象建造者）：它为创建一个产品Product对象的各个部件指定抽象接口，在该接口中一般声明两类方法，一类方法是buildPartX()，它们用于创建复杂对象的各个部件；另一类方法是getResult()，它们用于返回复杂对象。Builder既可以是抽象类，也可以是接口。

- ConcreteBuilder（具体建造者）：它实现了Builder接口，实现各个部件的具体构造和装配方法，定义并明确它所创建的复杂对象，也可以提供一个方法返回创建好的复杂产品对象。

- Product（产品角色）：它是被构建的复杂对象，包含多个组成部件，具体建造者创建该产品的内部表示并定义它的装配过程。

- Director（指挥者）：指挥者又称为导演类，它负责安排复杂对象的建造次序，指挥者与抽象建造者之间存在关联关系，可以在其construct()建造方法中调用建造者对象的部件构造与装配方法，完成复杂对象的建造。客户端一般只需要与指挥者进行交互，在客户端确定具体建造者的类型，并实例化具体建造者对象（也可以通过配置文件和反射机制），然后通过指挥者类的构造函数或者Setter方法将该对象传入指挥者类中。

导演者角色是与客户端打交道的角色。导演者将客户端创建产品的请求划分为对各个零件的建造请求，再将这些请求委派给具体建造者角色。具体建造者角色是做具体建造工作的，但是却不为客户端所知。

一般来说，每有一个产品类，就有一个相应的具体建造者类。这些产品应当有一样数目的零件，而每有一个零件就相应地在所有的建造者角色里有一个建造方法。这里就举买电脑的例子来介绍，我们假设电脑需要2个组件，主机和显示器。


#### 代码示例            
下面是Computer电脑产品的源代码：

```
public class Computer {
    private String host;//主机
    private String display;// 显示屏

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public String getDisplay() {
        return display;
    }

    public void setDisplay(String display) {
        this.display = display;
    }
}
```

下面是抽象建造者类Builder的源代码：

```
public interface  Builder {
    public void buildHost();
    public void buildDisplay();
    public Computer retrieveResult();
}
```

下面是具体建造者类LenovoBuilder的源代码：

```
public class LenovoBuilder implements Builder {
    Computer computer = new Computer();

    /**
     * 产品零件建造方法1
     */
    @Override
    public void buildDisplay() {
        //构建产品的第一个零件
        computer.setDisplay("联想显示器");
    }

    /**
     * 产品零件建造方法1
     */
    @Override
    public void buildHost() {
        //构建产品的第二个零件
        computer.setHost("联想主机");
    }

    /**
     * 产品返还方法
     */
    @Override
    public Computer retrieveResult() {
        return computer;
    }

}
```

下面是具体建造者类HuaweiBuilder的源代码：

```
public class HuaweiBuilder implements  Builder{
    Computer computer = new Computer();

    /**
     * 产品零件建造方法1
     */
    @Override
    public void buildDisplay() {
        //构建产品的第一个零件
        computer.setDisplay("华为显示器");
    }

    /**
     * 产品零件建造方法1
     */
    @Override
    public void buildHost() {
        //构建产品的第二个零件
        computer.setHost("华为主机");
    }
    /**
     * 产品返还方法
     */
    @Override
    public Computer retrieveResult() {
        return computer;
    }
}
```

下面是指挥者类Director类的源代码：

```
public class Director {
    /**
     * 持有当前需要使用的建造器对象
     */
    private Builder builder;
    /**
     * 构造方法，传入建造器对象
     * @param builder 建造器对象
     */
    public Director(Builder builder){
        this.builder = builder;
    }
    /**
     * 产品构造方法，负责调用各个零件建造方法
     */
    public void construct(){
        builder.buildHost();
        builder.buildDisplay();
    }
}
```

下面是客户端类Client源代码：

```
public class Client {
    public static void main(String[]args){
        Builder builder = new LenovoBuilder();
        Director director = new Director(builder);
        director.construct();
        Computer computer = builder.retrieveResult();
        System.out.println(computer.getHost());
        System.out.println(computer.getDisplay());
    }
}
```

输出：

> 联想主机
> 联想显示器

我们来看看时序图：
![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p1.png)

客户端负责创建导演者和具体建造者对象。然后，客户端把具体建造者对象交给导演者，导演者操作具体建造者，开始创建产品。当产品完成后，建造者把产品返还给客户端。

把创建具体建造者对象的任务交给客户端而不是导演者对象，是为了将导演者对象与具体建造者对象的耦合变成动态的，从而使导演者对象可以操纵数个具体建造者对象中的任何一个。


### 总结
建造者模式的核心在于如何一步步构建一个包含多个组成部件的完整对象，使用相同的构建过程构建不同的产品，在软件开发中，如果我们需要创建复杂对象并希望系统具备很好的灵活性和可扩展性可以考虑使用建造者模式。

#### 1.主要优点

- 在建造者模式中，客户端不必知道产品内部组成的细节，将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象。

- 每一个具体建造者都相对独立，而与其他的具体建造者无关，因此可以很方便地替换具体建造者或增加新的具体建造者，用户使用不同的具体建造者即可得到不同的产品对象。由于指挥者类针对抽象建造者编程，增加新的具体建造者无须修改原有类库的代码，系统扩展方便，符合“开闭原则”

- 可以更加精细地控制产品的创建过程。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。

#### 2.主要缺点

- 建造者模式所创建的产品一般具有较多的共同点，其组成部分相似，如果产品之间的差异性很大，例如很多组成部分都不相同，不适合使用建造者模式，因此其使用范围受到一定的限制。

- 如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大，增加系统的理解难度和运行成本。

#### 3.适用场景

- 需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员属性。

- 需要生成的产品对象的属性相互依赖，需要指定其生成顺序。

- 对象的创建过程独立于创建该对象的类。在建造者模式中通过引入了指挥者类，将创建过程封装在指挥者类中，而不在建造者类和客户类中。

- 隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品。

