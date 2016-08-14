---
layout: post
title: Android之IPC机制
date: 2016-8-13
categories: blog
tags: [android]
description: IPC
---

 Android IPC简介

任何一个操作系统都需要有相应的IPC机制，Linux上可以通过命名通道、共享内存、信号量等来进行进程间通信。Android系统不仅可以使用了Binder机制来实现IPC，还可以使用Socket实现任意两个终端之间的通信。

**IPC基础概念介绍**

(1)Serializable接口是Java中为对象提供标准的序列化和反序列化操作的接口，而Parcelable接口是Android提供的序列化方式的接口。

(2)serialVersionUId是一串long型数字，主要是用来辅助序列化和反序列化的，原则上序列化后的数据中的serialVersionUId只有和当前类的serialVersionUId相同才能够正常地被反序列化。

serialVersionUId的详细工作机制：序列化的时候系统会把当前类的serialVersionUId写入序列化的文件中，当反序列化的时候系统会去检测文件中的serialVersionUId，看它是否和当前类的serialVersionUId一致，如果一致就说明序列化的类的版本和当前类的版本是相同的，这个时候可以成功反序列化；否则说明版本不一致无法正常反序列化。一般来说，我们应该手动指定serialVersionUId的值。

1.静态成员变量属于类不属于对象，所以不参与序列化过程；                 
2.声明为transient的成员变量不参与序列化过程。

(3)Parcelable接口内部包装了可序列化的数据，可以在Binder中自由传输，Parcelable主要用在内存序列化上，可以直接序列化的有Intent、Bundle、Bitmap以及List和Map等等.


(4)Binder是Android中的一个类，它实现了IBinder接口。从IPC角度看，Binder是Android中一种跨进程通信的方式；Binder还可以理解为虚拟的物理设备，它的设备驱动是/dev/binder；从Framework层角度看，Binder是ServiceManager连接各种Manager和相应的ManagerService的桥梁；从Android应用层来说，Binder是客户端和服务端进行通信的媒介，当bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于AIDL的服务。

在Android开发中，Binder主要用在Service中，包括AIDL和Messenger，其中普通Service中的Binder不涉及进程间通信，较为简单；而Messenger的底层其实是AIDL，正是Binder的核心工作机制。

(5)aidl工具根据aidl文件自动生成的java接口的解析：首先，它声明了几个接口方法，同时还声明了几个整型的id用于标识这些方法，id用于标识在transact过程中客户端所请求的到底是哪个方法；接着，它声明了一个内部类Stub，这个Stub就是一个Binder类，当客户端和服务端都位于同一个进程时，方法调用不会走跨进程的transact过程，而当两者位于不同进程时，方法调用需要走transact过程，这个逻辑由Stub内部的代理类Proxy来完成。

所以，这个接口的核心就是它的内部类Stub和Stub内部的代理类Proxy。 下面分析其中的方法：

1.asInterface(android.os.IBinder obj)：用于将服务端的Binder对象转换成客户端所需的AIDL接口类型的对象，这种转换过程是区分进程的，如果客户端和服务端是在同一个进程中，那么这个方法返回的是服务端的Stub对象本身，否则返回的是系统封装的Stub.Proxy对象。

2.asBinder：返回当前Binder对象。

3.onTransact：这个方法运行在服务端中的Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。

这个方法的原型是public Boolean onTransact(int code, Parcelable data, Parcelable reply, int flags)
服务端通过code可以知道客户端请求的目标方法，接着从data中取出所需的参数，然后执行目标方法，执行完毕之后，将结果写入到reply中。如果此方法返回false，说明客户端的请求失败，利用这个特性可以做权限验证(即验证是否有权限调用该服务)。

4.Proxy#[Method]：代理类中的接口方法，这些方法运行在客户端，当客户端远程调用此方法时，它的内部实现是：首先创建该方法所需要的参数，然后把方法的参数信息写入到_data中，接着调用transact方法来发起RPC请求，同时当前线程挂起；然后服务端的onTransact方法会被调用，直到RPC过程返回后，当前线程继续执行，并从_reply中取出RPC过程的返回结果，最后返回_reply中的数据。

如果搞清楚了自动生成的接口文件的结构和作用之后，其实是可以不用通过AIDL而直接实现Binder的.


(6)Binder的两个重要方法linkToDeath和unlinkToDeath                 
Binder运行在服务端，如果由于某种原因服务端异常终止了的话会导致客户端的远程调用失败，所以Binder提供了两个配对的方法linkToDeath和unlinkToDeath，通过linkToDeath方法可以给Binder设置一个死亡代理，当Binder死亡的时候客户端就会收到通知，然后就可以重新发起连接请求从而恢复连接了。

如何给Binder设置死亡代理呢？

1.声明一个DeathRecipient对象，DeathRecipient是一个接口，其内部只有一个方法bindeDied，实现这个方法就可以在Binder死亡的时候收到通知了。

```
private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
    @Override
    public void binderDied() {
        if (mRemoteBookManager == null) return;
        mRemoteBookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
        mRemoteBookManager = null;
        // TODO:这里重新绑定远程Service
    }
};
```

2.在客户端绑定远程服务成功之后，给binder设置死亡代理

```
mRemoteBookManager.asBinder().linkToDeath(mDeathRecipient, 0);
```


### Android中的IPC方式        

(1)使用Bundle：Bundle实现了Parcelable接口，Activity、Service和Receiver都支持在Intent中传递Bundle数据。

(2)使用文件共享：这种方式简单，适合在对数据同步要求不高的进程之间进行通信，并且要妥善处理并发读写的问题。
SharedPreferences是一个特例，虽然它也是文件的一种，但是由于系统对它的读写有一定的缓存策略，即在内存中会有一份SharedPreferences文件的缓存，因此在多进程模式下，系统对它的读写就变得不可靠，当面对高并发读写访问的时候，有很大几率会丢失数据，因此，不建议在进程间通信中使用SharedPreferences。

(3)使用Messenger：Messenger是一种轻量级的IPC方案，它的底层实现就是AIDL。Messenger是以串行的方式处理请求的，即服务端只能一个个处理，不存在并发执行的情形，详细的示例见原书。

(4)使用AIDL

大致流程：首先建一个Service和一个AIDL接口，接着创建一个类继承自AIDL接口中的Stub类并实现Stub类中的抽象方法，在Service的onBind方法中返回这个类的对象，然后客户端就可以绑定服务端Service，建立连接后就可以访问远程服务端的方法了。

1.AIDL支持的数据类型：基本数据类型、String和CharSequence、ArrayList、HashMap、Parcelable以及AIDL；    
2.某些类即使和AIDL文件在同一个包中也要显式import进来；                  
3.AIDL中除了基本数据类，其他类型的参数都要标上方向：in、out或者inout；            
4.AIDL接口中支持方法，不支持声明静态变量；             
5.为了方便AIDL的开发，建议把所有和AIDL相关的类和文件全部放入同一个包中，这样做的好处是，当客户端是另一个应用的时候，可以直接把整个包复制到客户端工程中。                 
6.RemoteCallbackList是系统专门提供的用于删除跨进程Listener的接口。RemoteCallbackList是一个泛型，支持管理任意的AIDL接口，因为所有的AIDL接口都继承自IInterface接口。               


实现的具体步骤参考：[Android：学习AIDL，这一篇文章就够了(上) - lypeer的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/luoyanglizi/article/details/51980630)

(5)使用ContentProvider           
1.ContentProvider主要以表格的形式来组织数据，并且可以包含多个表；           
2.ContentProvider还支持文件数据，比如图片、视频等，系统提供的MediaStore就是文件类型的ContentProvider；    
3.ContentProvider对底层的数据存储方式没有任何要求，可以是SQLite、文件，甚至是内存中的一个对象都行；     
4.要观察ContentProvider中的数据变化情况，可以通过ContentResolver的registerContentObserver方法来注册观察者；

(6)使用Socket           
Socket是网络通信中“套接字”的概念，分为流式套接字和用户数据包套接字两种，分别对应网络的传输控制层的TCP和UDP协议     


**Binder连接池**

(1)当项目规模很大的时候，创建很多个Service是不对的做法，因为service是系统资源，太多的service会使得应用看起来很重，所以最好是将所有的AIDL放在同一个Service中去管理。整个工作机制是：每个业务模块创建自己的AIDL接口并实现此接口，这个时候不同业务模块之间是不能有耦合的，所有实现细节我们要单独开来，然后向服务端提供自己的唯一标识和其对应的Binder对象；对于服务端来说，只需要一个Service，服务端提供一个queryBinder接口，这个接口能够根据业务模块的特征来返回相应的Binder对象给它们，不同的业务模块拿到所需的Binder对象后就可以进行远程方法调用了。
Binder连接池的主要作用就是将每个业务模块的Binder请求统一转发到远程Service去执行，从而避免了重复创建Service的过程。      

(2)作者实现的Binder连接池BinderPool的实现源码，建议在AIDL开发工作中引入BinderPool机制。


**选用合适的IPC方式**     

![](http://hujiaweibujidao.github.io/images/androidart_ipc.png)



### Android之Binder详解   

Binder框架定义了四个角色：Server，Client，ServiceManager（以后简称SMgr）以及Binder驱动。其中Server，Client，SMgr运行于用户空间，驱动运行于内核空间。这四个角色的关系和互联网类似：Server是服务器，Client是客户终端，SMgr是域名服务器（DNS），驱动是路由器。


**Binder 驱动**            

和路由器一样，Binder驱动虽然默默无闻，却是通信的核心。尽管名叫‘驱动’，实际上和硬件设备没有任何关系，只是实现方式和设备驱动程序是一样的：它工作于内核态，提供open()，mmap()，poll()，ioctl()等标准文件操作，以字符驱动设备中的misc设备注册在设备目录/dev下，用户通过/dev/binder访问该它。驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。驱动和应用程序之间定义了一套接口协议，主要功能由ioctl()接口实现，不提供read()，write()接口，因为ioctl()灵活方便，且能够一次调用实现先写后读以满足同步交互，而不必分别调用write()和read()。Binder驱动的代码位于linux目录的drivers/misc/binder.c中。


**ServiceManager 与实名Binder**

和DNS类似，SMgr的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。注册了名字的Binder叫实名Binder，就象每个网站除了有IP地址外还有自己的网址。Server创建了Binder实体，为其取一个字符形式，可读易记的名字，将这个Binder连同名字以数据包的形式通过Binder驱动发送给SMgr，通知SMgr注册一个名叫张三的Binder，它位于某个Server中。驱动为这个穿过进程边界的Binder创建位于内核中的实体节点以及SMgr对实体的引用，将名字及新建的引用打包传递给SMgr。SMgr收数据包后，从中取出名字和引用填入一张查找表中。

细心的读者可能会发现其中的蹊跷：SMgr是一个进程，Server是另一个进程，Server向SMgr注册Binder必然会涉及进程间通信。当前实现的是进程间通信却又要用到进程间通信，这就好象蛋可以孵出鸡前提却是要找只鸡来孵蛋。Binder的实现比较巧妙：预先创造一只鸡来孵蛋：SMgr和其它进程同样采用Binder通信，SMgr是Server端，有自己的Binder对象（实体），其它进程都是Client，需要通过这个Binder的引用来实现Binder的注册，查询和获取。SMgr提供的Binder比较特殊，它没有名字也不需要注册，当一个进程使用BINDER_SET_CONTEXT_MGR命令将自己注册成SMgr时Binder驱动会自动为它创建Binder实体（这就是那只预先造好的鸡）。其次这个Binder的引用在所有Client中都固定为0而无须通过其它手段获得。也就是说，一个Server若要向SMgr注册自己Binder就必需通过0这个引用号和SMgr的Binder通信。类比网络通信，0号引用就好比域名服务器的地址，你必须预先手工或动态配置好。要注意这里说的Client是相对SMgr而言的，一个应用程序可能是个提供服务的Server，但对SMgr来说它仍然是个Client。


**Client 获得实名Binder的引用**

Server向SMgr注册了Binder实体及其名字后，Client就可以通过名字获得该Binder的引用了。Client也利用保留的0号引用向SMgr请求访问某个Binder：我申请获得名字叫张三的Binder的引用。SMgr收到这个连接请求，从请求数据包里获得Binder的名字，在查找表里找到该名字对应的条目，从条目中取出Binder的引用，将该引用作为回复发送给发起请求的Client。从面向对象的角度，这个Binder对象现在有了两个引用：一个位于SMgr中，一个位于发起请求的Client中。如果接下来有更多的Client请求该Binder，系统中就会有更多的引用指向该Binder，就象java里一个对象存在多个引用一样。而且类似的这些指向Binder的引用是强类型，从而确保只要有引用Binder实体就不会被释放掉。通过以上过程可以看出，SMgr象个火车票代售点，收集了所有火车的车票，可以通过它购买到乘坐各趟火车的票-得到某个Binder的引用。

**匿名 Binder**

并不是所有Binder都需要注册给SMgr广而告之的。Server端可以通过已经建立的Binder连接将创建的Binder实体传给Client，当然这条已经建立的Binder连接必须是通过实名Binder实现。由于这个Binder没有向SMgr注册名字，所以是个匿名Binder。Client将会收到这个匿名Binder的引用，通过这个引用向位于Server中的实体发送请求。匿名Binder为通信双方建立一条私密通道，只要Server没有把匿名Binder发给别的进程，别的进程就无法通过穷举或猜测等任何方式获得该Binder的引用，向该Binder发送请求。

![](http://hi.csdn.net/attachment/201107/19/0_13110996490rZN.gif)

参考链接：[Android Bander设计与实现 - 设计篇 - universus的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/universus/article/details/6211589)


