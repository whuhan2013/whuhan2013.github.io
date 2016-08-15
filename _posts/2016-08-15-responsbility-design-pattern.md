---
layout: post
title: 设计模式之责任链模式
date: 2016-8-15
categories: blog
tags: [设计模式]
description: 设计模式
---

**概述**

责任链可以是一条直线、一个环或者一个树形结构，最常见的责任链是直线型，即沿着一条单向的链来传递请求。链上的每一个对象都是请求处理者，职责链模式可以将请求的处理者组织成一条链，并让请求沿着链传递，由链上的处理者对请求进行相应的处理，客户端无须关心请求的处理细节以及请求的传递，只需将请求发送到链上即可，实现请求发送者和请求处理者解耦。



### 核心

概念： 避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。责任链模式是一种对象行为型模式。

责任链模式结构重要核心模块：

#### Handler（抽象处理者）

定义一个处理请求的接口，一般设计为抽象类，由于不同的具体处理者处理请求的方式不同，因此在其中定义了抽象请求处理方法。因为每一个处理者的下家还是一个处理者，因此在抽象处理者中定义了一个抽象处理者类型的对象作为其对下家的引用。通过该引用，处理者可以连成一条链。（是不是有点像C语言的链表结构呢？）

#### ConcreteHandler（具体处理者）

抽象处理者的子类，可以处理用户请求，在具体处理者类中实现了抽象处理者中定义的抽象请求处理方法，在处理请求之前需要进行判断，看是否有相应的处理权限，如果可以处理请求就处理它，否则将请求转发给后继者；在具体处理者中可以访问链中下一个对象，以便请求的转发。

### 责任链模式分类：

**纯的责任链模式**

一个纯的职责链模式要求一个具体处理者对象只能在两个行为中选择一个：要么承担全部责任，要么将责任推给下家，不允许出现某一个具体处理者对象在承担了一部分或全部责任后又将责任向下传递的情况。而且在纯的职责链模式中，要求一个请求必须被某一个处理者对象所接收，不能出现某个请求未被任何一个处理者对象处理的情况。

**不纯的责任链模式**

在一个不纯的职责链模式中允许某个请求被一个具体处理者部分处理后再向下传递，或者一个具体处理者处理完某请求后其后继处理者可以继续处理该请求，而且一个请求可以最终不被任何处理者对象所接收。Java AWT 1.0中的事件处理模型应用的是不纯的职责链模式（其实Android的事件分发机制也是类似），

其基本原理如下：由于窗口组件（如按钮、文本框等）一般都位于容器组件中，因此当事件发生在某一个组件上时，先通过组件对象的handleEvent()方法将事件传递给相应的事件处理方法，该事件处理方法将处理此事件，然后决定是否将该事件向上一级容器组件传播；上级容器组件在接到事件之后可以继续处理此事件并决定是否继续向上级容器组件传播，如此反复，直到事件到达顶层容器组件为止；如果一直传到最顶层容器仍没有处理方法，则该事件不予处理。每一级组件在接收到事件时，都可以处理此事件，而不论此事件是否在上一级已得到处理，还存在事件未被处理的情况。显然，这就是不纯的职责链模式，早期的Java AWT事件模型(JDK 1.0及更早)中的这种事件处理机制又叫事件浮升(Event Bubbling)机制。从Java.1.1以后，JDK使用观察者模式代替责任链模式来处理事件。目前，在JavaScript中仍然可以使用这种事件浮升机制来进行事件处理。

### 使用场景

有多个对象可以处理同一个请求，具体哪个对象处理该请求待运行时刻再确定，客户端只需将请求提交到链上，而无须关心请求的处理对象是谁以及它是如何处理的。

在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。

可动态指定一组对象处理请求，客户端可以动态创建职责链来处理请求，还可以改变链中处理者之间的先后次序。


### 实例

这里展示一个责任链模式的例子（模拟Android App开发过程，程序猿写好代码需要找UI设计师审核UI效果、找Leader review代码、找测试工程师测试App，然后找产品经理确认App后发布App），严格遵守责任链模式几大核心模板，不多解释：

```
package yanbober.github.io;

//常量
interface Type {
    int UI_EFFECT = 0;
    int CODE_FORMAT = 1;
    int TEST_PROJECT = 2;
    int PUBLISH_APP = 3;
}
//Handler（抽象处理者）
abstract class Handler {
    private Handler mHandler;

    public Handler getmHandler() {
        return mHandler;
    }

    public void setmHandler(Handler mHandler) {
        this.mHandler = mHandler;
    }

    abstract String handleRequest(int type, String user);
}
//ConcreteHandler（具体处理者）
class ConcreteHandlerGUI extends Handler {
    @Override
    String handleRequest(int type, String user) {
        String ret = "";

        if (Type.UI_EFFECT == type) {
            ret = user + "’s App UI is Ok!";
        }
        else {
            if (getmHandler() != null) {
                System.out.println("*********** ConcreteHandlerGUI **************");
                ret = getmHandler().handleRequest(type, user);
            }
        }

        return ret;
    }
}

class ConcreteHandlerTeamLeader extends Handler {
    @Override
    String handleRequest(int type, String user) {
        String ret = "";

        if (Type.CODE_FORMAT == type) {
            ret = user + "’s App code format is Ok!";
        }
        else {
            if (getmHandler() != null) {
                System.out.println("*********** ConcreteHandlerTeamLeader **************");
                ret = getmHandler().handleRequest(type, user);
            }
        }

        return ret;
    }
}

class ConcreteHandlerPM extends Handler {
    @Override
    String handleRequest(int type, String user) {
        String ret = "";

        if (Type.PUBLISH_APP == type) {
            ret = user + "’s App can be publish!";
        }
        else {
            if (getmHandler() != null) {
                System.out.println("*********** ConcreteHandlerPM **************");
                ret = getmHandler().handleRequest(type, user);
            }
        }

        return ret;
    }
}

class ConcreteHandlerTestEngineer extends Handler {
    @Override
    String handleRequest(int type, String user) {
        String ret = "";

        if (Type.TEST_PROJECT == type) {
            ret = user + "’s App test is Ok!";
        }
        else {
            if (getmHandler() != null) {
                System.out.println("*********** ConcreteHandlerTestEngineer **************");
                ret = getmHandler().handleRequest(type, user);
            }
        }

        return ret;
    }
}
//客户端
public class Main {
    public static void main(String[] args) {
        Handler code = new ConcreteHandlerTeamLeader();
        Handler ui = new ConcreteHandlerGUI();
        Handler test = new ConcreteHandlerTestEngineer();
        Handler publish = new ConcreteHandlerPM();

        code.setmHandler(ui);
        ui.setmHandler(test);
        test.setmHandler(publish);
        //yanbo找team leader review代码
        System.out.println(code.handleRequest(Type.CODE_FORMAT, "YanBo"));
        //yanbo找team leader测试代码
        System.out.println(code.handleRequest(Type.TEST_PROJECT, "YanBo"));
    }
}
```

### 总结一把

职责链模式优点：

职责链模式使得一个对象无须知道是其他哪一个对象处理其请求，对象仅需知道该请求会被处理即可，接收者和发送者都没有对方的明确信息，且链中的对象不需要知道链的结构，由客户端负责链的创建，降低了系统的耦合度。

请求处理对象仅需维持一个指向其后继者的引用，而不需要维持它对所有的候选处理者的引用，可简化对象的相互连接。

在给对象分派职责时，职责链可以给我们更多的灵活性，可以通过在运行时对该链进行动态的增加或修改来增加或改变处理一个请求的职责。

在系统中增加一个新的具体请求处理者时无须修改原有系统的代码，只需要在客户端重新建链即可，从这一点来看是符合“开闭原则”的。

职责链模式缺点：

因为一个请求没有明确的接收者，所以不能保证它一定会被处理，该请求可能一直到链的末端都得不到处理；一个请求也可能因职责链没有被正确配置而得不到处理。

对于比较长的职责链，请求的处理可能涉及到多个处理对象，系统性能将受到一定影响，而且在进行代码调试时不太方便。

如果建链不当，可能会造成循环调用，将导致系统陷入死循环。


**参考链接**

[设计模式(行为型)之职责链模式(Chain of Responsibility Pattern) - 工匠若水 - 博客频道 - CSDN.NET](http://blog.csdn.net/yanbober/article/details/45531395)