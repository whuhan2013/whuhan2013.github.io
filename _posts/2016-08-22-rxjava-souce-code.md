---
layout: post
title: RXJava源码解析
date: 2016-8-22
categories: blog
tags: [源码解析]
description: RXJava
---


### 前言
RxJava是用java实现的ReactiveX（Reactive Extensions）框架开源库。ReactiveX则是大名鼎鼎的响应式编程。而响应式编程和观察者模式紧紧的相关联。在看RxJava的源码中，分析起来会有点麻烦，所以才有了这篇文章，和对这个有兴趣的同学一起窥探一二。


### 观察者模式  

2.1 基本原理                          
观察者模式是对象的行为模式，又叫发布-订阅(Publish/Subscribe)模式即让多个观察者对象同时监听某一个主题对象。这个主题对象在状态上发生变化时，会通知所有观察者对象，使它们能够自动更新自己。在JDK中已有对观察者模式的封装：Observable.java/Observer.java（请自行对照JDK源码看看，下面会提出2个问题）：


2.2 jdk实现版本                        
在JDK的实现版本中，Observable是被观察者，有一个Vector数组来存放观察者，当被观察者事件改变时，调用notifyObservers来通知所有的观察者，每个观察者调用自己的update来做相应的更新操作。由于这个模式在JDK中实现的比较早（JDK1.0），有2个地方值得思考：

1、观察者数组使用Vector，为什么不使用List？是否有替代品？                    
由于观察者数组必须考虑到线程安全（比如在被观察者发出通知的那一刻，同时添加新的观察者，或者删除某个观察者等操作会让被观察者不知道到底通知哪些观察者，即会引发线程安全），所以JDK版本中在对vector进行操作的时候，都会加上synchronized且在通知观察者的时候反向遍历数组，以此来保证线程安全。     

（1）为什么不用List？                         
这里可以看到通知观察者的时候需要反向遍历数组，来保证如果是发出通知时有新的观察者进来，新的观察者不会收到当前通知。如果使用List反向遍历会比使用数组更加复杂。

（2）是否有替代品？                                   
这里的Vector操作满足（a）读大于写，（b）线程安全，（c）写时复制，所以可以用CopyOnWriteArrayList来代替，代替后实现代码会更加的简洁明了。            

通知方法，通知每个观察者的时候没有使用try—catch，是否可以加上？

JDK observable通知方法notifyObservers代码实现：

```
for (int i = arrLocal.length-1; i>=0; i--)
    ((Observer)arrLocal[i]).update(this, arg);
```

初看到这个段代码时，会有疑惑这个循环中没有对update加try-catch，当某个观察者的更新操作抛异常会导致其他的观察者更新失败。
那么如果我们加上try-catch会发生什么呢，虽然能够保证每个观察者都能做更新操作，但是一旦某个观察者有异常，被被观察者给捕获了，而被观察者捕获后又不知道交给谁，怎么处理，会导致代码编写者以为所有的观察者都正常执行了，所以在实现Observer的update操作时需要观察者自己加上try-catch。


### 响应式编程   

3.1 响应式编程的特点               

上面有点跑了题。回归正传，有了观察者模式的实现做铺垫，对我们理解响应式编程原理会有很大的帮助。相同点，都是多个观察者观察一个被观察者的状态，观察者对状态变化做出自己的处理。不同点，从我目前看到源码的程度上来看，我觉得观察者模式和响应式编程的区别主要有以下几个：

- 1 观察者模式是Observable-Observer，如果我们要在观察者做操作前，对数据做一些其他的处理怎么办？观察者模式无法解耦，只能在observer的update操作中来做。而响应式编程是Obsevable-Operator1-OperatorN-Observer，可以很好的解耦控制数据流操作。
- 2 响应式编程通过链式调用，让用户在代码流程上能够更加清晰的掌控（即使是异步操作）。
- 3 观察者模式无法处理观察者的异常，需要用户自己加try-catch结构。而响应式编程提供了另一种方案，用户不需要使用try-catch只需实现错误处理方法就可以做到。
- 4 观察者模式若需要使用多线程，需要用户在observer中实现多线程操作，也就是将观察者和任务调度糅杂在一起。而响应式编程对观察者和任务调度解耦，可以通过Schedulers，来处理线程调度问题。
- 5 观察者模式，观察者无法处理自己的调用超时问题，响应式编程则可以设定观察者的调度超时机制。
- 6 响应式编程提供阻塞式(BlockingObservable)和非阻塞式Observable的调用


### RxJava源码分析            
上文讲了RxJava框架相比观察者模式的各种优点，现在我们来分析源码，由于RxJava的源码比较庞大，本篇博客只分析RxJava的基本原理，即区别中的第一点和第二点。对于其他的点会在另起blog来分析。

3.2.1 调用展现

好了，分析RxJava的基本原理前，我们先上一段测试代码（此处的Observable是RxJava中的，不是JDK中的）：

```
Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("hello");
    }
}).map(new Func1<String, String>() {
    @Override
    public String call(String s) {
        return s + " word";
    }
}).subscribe(new Subscriber<String>() {
    @Override
    public void onCompleted() {
        System.out.println("completed");
    }
    @Override
    public void onError(Throwable e) {
        System.out.println("error");
    }
    @Override
    public void onNext(String s) {
        System.out.println("rx " + s);
    }
});
```

输出：rx hello world                      
这种连.的写法对看代码来说有点累，简化下：          

```
(1)Observable<String> observable1 = Observable.create(new OnSubscribe());
(2)Observable<String> observable2 = observable.map(new Func1());
(3)Subscription sub = observable2.subscribe(new Subscriber());
```

代码一简化，我们就可以对上文说的3.1节中响应式编程的特点1：数据流操作和观察者处理解耦的原理有所了解了。
这里的解耦方式，其实就是在原被观察者进行数据流操作（map）后生成一个新的被观察者，观察者其实订阅的是数据流操作生成的被观察者。
在整个过程中，可以看到三个重要的类：

- Observable：被观察者
- OnSubscriber：Observable的成员变量，用来调用观察者处理方法
- Subscriber：观察者


#### 代码分析

RxJava具体怎么处理，分3步看：

3.2.2.1 生成被观察者Observable对象

第一行代码，(1)Observable observable1 = Observable.create(new OnSubscribe());
创建了一个被观察者，进入源码

```
public final static <T> Observable<T> create(OnSubscribe<T> f) {
    return new Observable<T>(hook.onCreate(f));
}

/**
 * Invoked when Observable.subscribe is called.
 */
public interface OnSubscribe<T> extends Action1<Subscriber<? super T>> {
    // cover for generics insanity
}
```

即create为Observable的对象observable1创建了一个默认的钩子hook和一个Onsubscribe对象。

3.2.2.2 数据流处理

第二行代码，(2)Observable observable2 = observable.map(new Func1());
创建了一个新的被观察者2，进入源码：

```
public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
    return lift(new OperatorMap<T, R>(func));
}
```

这一行代码很关键，也很绕，2个动作                
（1）生成了一个OperatorMap对象。这个对象call方法，参数为观察者Subscriber对象；返回值是一个新的观察者Subscriber对象。新的Subscriber对象的onNext方法先调用operator操作（map的call方法）的call方法，拿Operator操作返回的数据，再调用外部观察者的onNext方法。很熟悉是吗？上文说到的ReactiveX编程可以实现Observable-Operator1-----OperatorN-Observer，在这段代码中得到了体现：在调用Observer的OnNext方法前，会先调用OperatorN的call方法对数据处理，处理完成后的数据值，作为参数传入Observer的OnNext方法。

我们需要注意的是，OperatorMap的call方法返回的是一个新的观察者，新的观察者通过OnNext方法将老的观察者给链接起来的。那么新的观察者什么时候订阅被观察者的呢？这个就是接下来的第（2）个动作：lift


```
public final class OperatorMap<T, R> implements Operator<R, T> {
    private final Func1<? super T, ? extends R> transformer;
    public OperatorMap(Func1<? super T, ? extends R> transformer) {
        this.transformer = transformer;
    }
    @Override
    public Subscriber<? super T> call(final Subscriber<? super R> o) {
        return new Subscriber<T>(o) {
            @Override
            public void onCompleted() {
                o.onCompleted();
            }
            @Override
            public void onError(Throwable e) {
                o.onError(e);
            }
            @Override
            public void onNext(T t) {
                try {
                    o.onNext(transformer.call(t));
                } catch (Throwable e) {
                    Exceptions.throwOrReport(e, this, t);
                }
            }
        };
    }

}
```

（2）调用了observable1的lift方法，这个方法用来干什么呢？

```
public final <R> Observable<R> lift(final Operator<? extends R, ? super T> operator) {
    return new Observable<R>(new OnSubscribe<R>() {
        @Override
        public void call(Subscriber<? super R> o) {
            Subscriber<? super T> st = hook.onLift(operator).call(o);
            st.onStart();
            onSubscribe.call(st); 
        });
}
```

源码略大我简化了一下，其实就是原Observable（observable1）的lift方法创建了一个新的Observable对象（observable2），这个新的对象的OnSubscribe成员变量的call方法将map-OpreatorMap生成的观察者传入到了最开始的Observable的OnSubscirbe中进行处理。

过程再整理一下：

- 1、 Create操作：创建一个Observable对象observable1，对象中的OnSubscribe用来响应Subscriber
- 2、 OperatorMap操作生成新的观察者，新的观察者通过OnNext方法，先调用map的call方法处理数据，再调用1中的观察者Subscriber的OnNext方法。
- 3、lift 操作生成新的被观察者observable2，observable2中的call方法通过observable1的成员变量OnSubscribe来调用2中生成的新的观察者的call方法。

接下来就是最后一步了，什么时候调用observable2的call方法呢？这个就是下面的介绍，被观察者订阅观察者，在订阅的时候进行调用。

3.2.2.3 观察者订阅

第三行代码，Subscription sub = observable2.subscribe(new Subscriber());
订阅subscribe源码核心代码：

``
hook.onSubscribeStart(observable,observable.onSubscribe).call(subscriber);
```

其实这个就是调用observable2的OnSubscribe的call方法，参数是传入的订阅者。
说白了观察者订阅其实就是用被观察者的subscribe方法订阅观察者，且调用被观察者的OnSubscribe方法来调用观察者的处理方法onNext。

这里可能就会有疑问：为什么是被观察者来订阅观察者，而不是观察者来订阅被观察者？
这种做法可以使整个过程使用连点（.）来完成从数据处理到订阅者响应的所有流程，代码上会更加的清晰。

好了，是时候祭出大招了，映照下面这个图，你会更加的清晰。

![](http://img4.tbcdn.cn/L1/461/1/58f17559af316aa98adf11b3e7fe08bc18b9b354.png)

**参考：**

[RxJava源码初探-博客-云栖社区-阿里云](https://yq.aliyun.com/articles/4201#11)

[jtsky/RxJava--analyse: 关于Rxjava的源码分析](https://github.com/jtsky/RxJava--analyse)


### 原理解析  

![](http://img.blog.csdn.net/20160504093452047)

我先根据这张图给出RxJava能这样运行的3条准则：

- 从create函数能向下执行到subscribe函数，靠的是返回Observable类的对象；
- 从subscribe函数能再向上执行到onSubscriber1，靠的是下面的Observable对象能够调用上面Observable对象的onSubscribe属性；
- 从上面能再次往下依次对数据进行处理，靠的是Subscriber责任链。  

为什么我不先分析源码而先给准则呢？我发现如果在不知道准则（或者称作设计思路）情况下看源码就会“见树不见林”，而知道了准则就把握了框架脉络，剩下的只是到源码中找答案而已。既然本文标榜了“快速理解RxJava源码的设计理念”，那当然不能徒有虚名。下面我们到源码中找答案。

首先我们来证明准则1：

```
public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
    return lift(new OperatorMap<T, R>(func));
}
```


map返回的是Observable，证明完毕。快吧，就这么快。在此将map返回的Observable对象标记为observable2。

再证明准则2：

对照demo中的代码，我们调用map之后，就调用了subscribe方法，也就是调用了这里的observable2的subscribe方法。 
根据上面的介绍，调用subscribe之后，就会调用observable2.onSubscribe.call方法


```
public final <R> Observable<R> lift(final Operator<? extends R, ? super T> operator) {
    return new Observable<R>(new OnSubscribe<R>() {
        @Override
        public void call(Subscriber<? super R> o) {
            Subscriber<? super T> st = hook.onLift(operator).call(o);
            st.onStart();
            onSubscribe.call(st);
        }
    });
}
```

其中，call函数的第一句与准则3的证明有关，暂时不看。第二句也不看（不好意思，我没看懂这句干什么的，不过不影响继续分析），我们看第三句：

```
onSubscribe.call(st);
```

这句很关键，准则2的证明靠他了。我们要搞明白onSubscribe是谁的属性。我将demo换个方式写出来：

```
...
Observable.create(onSubscriber1);
observable1.map(transformer1);
observable2.subscribe(subscriber1);
```

map函数是observable1的，那么lift函数也是observable1的，按Java语法规定内部类可以调用外部类的属性，所以onSubscribe是observable1的。这样就实现了observable2跨入到observable1的目的。证毕。为更好地理解，我把observable2.onSubscribe.call方法【代码1】再用括号注释下：

```
public final <R> Observable<R> （Observable1的）lift(final Operator<? extends R, ? super T> operator) {
    return new （返回了Observable2）Observable<R>(new OnSubscribe<R>() {
        @Override
        public void call(Subscriber<? super R> o) {
            Subscriber<? super T> st = hook.onLift(operator).call(o);
            st.onStart();
            （Observable1的）onSubscribe.call(st);
        }
    });
}
```

通俗地说，因为observable2是在observable1中（lift函数中）生成的，所以可以利用Java的内部类特性调用外部对象（observable1）的onSubscribe。 

**最后证明准则3：**

说实话，写到这里我感觉还是有部分同学没明白怎么回事。不过没关系，我将会继续围绕讲述。 
我们再来看看observable2.onSubscribe.call方法：

```
public final <R> Observable<R> lift(final Operator<? extends R, ? super T> operator) {
    return new Observable<R>(new OnSubscribe<R>() {
        @Override
        public void call(Subscriber<? super R> o) {
            Subscriber<? super T> st = hook.onLift(operator).call(o);
            st.onStart();
            onSubscribe.call(st);
        }
    });
}
```

前面说了，call函数的第一句与准则3的证明有关。我们先不看它做了什么，返回值给了st，st是个Subscriber对象。我们在第一节总结说了，处理数据是订阅类Subscriber的职责，因此st有处理数据的职责。再看第三句，将st给了observable1。问题来了，准则3想要运行，就必须有Subscriber责任链。“链”在哪里？不用猜了，我直接告诉你答案：往下“链”的代码在：


```
Subscriber<? super T> st = hook.onLift(operator).call(o);
```

我们看看OperatorMap的call方法：

```
@Override
public Subscriber<? super T> call(final Subscriber<? super R> o) {
    return new Subscriber<T>(o) {
        @Override
        public void onNext(T t) {
            o.onNext(transformer.call(t));
        }
    };
}
```

入口参数o在Demo里就是subscriber1对象。我们看它的这句：

```
o.onNext(transformer.call(t));
```

按照语法规定，是先执行括号里的。我把它写得再直白一点：

临时数据=map(处理前的数据);//本步骤处理                
subscriber1.onNext(临时数据);//提交给下一步处理

往上“链”的代码在：

```
onSubscribe.call(st);
```

总结一下： 

在observable2这种操作符返回的被观察者中，OnSubscribe要做两件事：

- 在本级中生成订阅类Subscriber，在Subscriber中将本级的操作（map）与下一级操作连接起来（向下链接）
- 将本级中生成订阅类Subscriber交给上一级Observable，方便上一级向下链接

因此，在此的订阅类Subscriber的职责实际上有两个：

- 处理数据
- 连接下一个步骤的Subscriber 

再上一张图帮助消化：

![](http://img.blog.csdn.net/20160504163651762)


**参考链接** 

[快速理解RxJava源码的设计理念 - byamao1的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/byamao1/article/details/51312602)

[RxJava && Agera从源码简要分析基本调用流程](http://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=2649796857&idx=1&sn=ed8325aeddac7fd2bd81a0717c010e98&scene=1&srcid=0821XpNhBoQKsitwdYeM9PyE#wechat_redirect)

现在我们将整个流程梳理一下：            
1、一次map()变换                         
2、根据Operator实例生成新的Subscriber               
3、通过lift()生成新的Observable                   
4、原Subscriber订阅新的Observavble                              
5、新的Observable中onSubscribe通知新Subscriber订阅原Observable                  
6、新Subscriber将消息传给原Subscriber。                      


