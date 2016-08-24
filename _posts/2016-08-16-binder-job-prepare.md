---
layout: post
title: 跟面试官讲Binder
date: 2016-8-16
categories: blog
tags: [求职]
description: 求职
---


### 转载   

**面试的时候，面试官问你说，简单说一下Android的Binder机制，你会怎么回答？**

我想，我会这么说。

在Android启动的时候，Zygote进程孵化出第一个子进程叫SystemServer，而在这个进程中，很多系统提供的服务，比如ActivityManagerSerivce, PowerManagerService等，都在此进程中的某一条线程上运行。

而很多用户开发的应用程序，也就是我们常说的APP，基于安全的考虑，在安装到系统中的时候，都会被分配一个独特的UID，运行在一个单独的进程中。

基于Linux的用户安全机制，它们是没有办法直接访问到上述系统服务的。

但是在实际中，App很多时候是需要跟这些系统服务进行通信的，也就是要进行进程间通信。

而Binder机制则是Android系统实现进程间通信（IPC）的机制之一。


**面试官可能还会问，那么Binder机制是怎么实现的呢？**

我觉得，要想知道Binder机制是怎么实现的，首先要知道下面四个概念：

1）Server

2）Client

3）ServiceManager

4）Binder驱动

Server和Client无需多说，我们只需要知道一个是用来提供服务的，一个是获取服务的，并且分别运行在不同的进程当中。

那么ServiceManager是用来干什么的呢？

在前面一篇文章，关于Android的启动过程中，我们有提到，ServiceManager是和Zygote进程一样，是Android系统的守护进程之一。

它的主要作用，就是帮助系统去维护众多的Service列表，所以叫Service Manager。

它就像个电话本，你要打电话给某个人问点事情（想要获取某种Service），但是你没有这个人的电话（不知道Service的访问点，只知道名字），于是你先找到电话本（ServiceManager），在电话本上找到这个人的电话（通过名字找到电话），然后你才能打电话给他（开始获取Service）。

我们知道，ServiceManager，也是一个单独的进程。要找到ServiceManager，本身也要涉及到进程间通信啊，不是吗？

是的。

从Client和Server的角度来看，ServiceManager永远都是一个Server，任何访问ServiceManager的都是Client。

我们先想一想，ServiceManager本身就是一个进程，那为什么这个进程就能够成为ServiceManager呢？

回想一下，在Android启动ServiceManager进程的时候，都做了什么事？

1）首先打开“/dev/binder”设备文件，映射内存

2）利用BINDER_SET_CONTEXT_MGR命令，令自己成为上下文管理者，其实也就是成为ServiceManager。

3）进入一个无限循环，等待Client的请求到来。

所以，是通过Binder驱动，通过命令"BINDER_SET_CONTEXT_MGR"，让某个进程成为ServiceManager的。


**面试官这时候，又适当的提了一个问题，Binder驱动是用来干什么的？它跟网卡驱动，声卡驱动一样的吗？**

不一样。

Binder驱动其实跟硬件没有什么关系，它就是一段运行在内核空间的代码，通过一个叫"/dev/binder"的文件在内核空间和用户空间来回搬数据。

它是整个Binder机制的核心。正是通过它，Binder才能实现进程间的通信。

因为Server, Client和ServiceManager是运行在用户空间，不同的进程，彼此是不能互相访问的。

那么在不同的进程间，要进行数据的传递，只有通过内核空间来传递数据，

而只有Binder驱动是工作在内核空间中的。

它负责Binder节点的建立，负责Binder在进程间的传递，负责对Binder引用的计数管理，在不同进程间传输数据包等底层操作。

整个进程间通信的大部分工作都是通过 open(), mmap()和 ioctl()等文件操作，在内核空间与用户空间中进进出出，来实现的。

- 通常意义下，Binder指的是一种通信机制；我们说AIDL使用Binder进行通信，指的就是Binder这种IPC机制。
- 对于Server进程来说，Binder指的是Binder本地对象
- 对于Client来说，Binder指的是Binder代理对象，它只是Binder本地对象的一个远程代理；对这个Binder代理对象的操作，会通过驱动最终转发到Binder本地对象上去完成；对于一个拥有Binder对象的使用者而言，它无须关心这是一个Binder代理对象还是Binder本地对象；对于代理对象的操作和对本地对象的操作对它来说没有区别。
- 对于传输过程而言，Binder是可以进行跨进程传递的对象；Binder驱动会对具有跨进程传递能力的对象做特殊处理：自动完成代理对象和本地对象的转换。


**面试官接着问，那"BINDER_SET_CONTEXT_MGR"又是怎么回事？**

是这样的。

ServiceManager进程启动的时候，此时它还不是ServiceManager呢。

不过，当它打开了/dev/binder文件，将BINDER_SET_CONTEXT_MGR命令传给Binder驱动的时候，Binder驱动就会为其在内核空间中创建一个节点（binder_node），

这个节点就是binder_context_mgr_node，也就是ServiceManager的Binder实体。

而这个节点所在的进程，就是ServiceManager。


**面试官突然间插了一句话，“那么，你怎么知道ServiceManager在哪的？“**

在整个系统中，只会有一个binder_context_mgr_node，所以也只会有一个ServiceManager的进程，那么对ServiceManager的访问，驱动就可以在系统中定义好其句柄，也就是 0。


所以只要是想找ServiceManager的进程，只要告诉Binder驱动，想访问的是句柄为 0的进程，Binder驱动就知道，它们是在找ServiceManager。

它就会唤醒ServiceManager，让它接受访问。

所以，"BINDER_SET_CONTEXT_MGR" 就是某某进程告诉驱动，它想成为ServiceManager。而只要它是第一个发送这个命令的，它就是ServiceManager了。


**面试官好像稍微明白了一点，不过他想了一想，又问：”什么是句柄？“**

呃，这个我也不知道怎么说，句柄理解上应该就是一个区分不同服务的标识吧。不过在Android系统中，只有跨进程的通信，才会用到句柄这个术语，因为没有办法直接访问。

而如果服务运行在本地进程中，由于共享同一个进程空间，是可以直接访问的，这个时候就不用句柄来描述了，而称之为引用。

所以，句柄与引用的区别，就在于一个是远程访问（跨进程），一个是本地访问（相同进程）。


面试官若有所思地看着面前这个人，目光中好像有那么点怀疑的样子，不过也就没再继续问下去。

**但是他突然间又说了一句话，”你还没有回答我，Binder的机制是怎么实现的呢，先简单地说一下？“**

通过上面的简单描述，我们可以这样认为，每一个提供服务的Server都会通过Binder驱动，将自身给注册到ServiceManager中，方便众多想获取服务的Client可以去ServiceManager找到自己。

那么，这些Service都会经过内核空间的Binder驱动，其实这个"经过"的说法，本质上，就是Server们会将自身作为一个对象，封装在数据包中，将这些数据复制到内核空间中，由Binder驱动访问。

而Binder驱动读取数据包的时候，如果发现其中有Binder实体，类似ServiceManager那样的服务提供商，那么也会为对应的Binder实体创建对应的Binder节点（BinderNode）。

这些节点位于Server所属的进程内。

Binder驱动也会为这些服务分配句柄（大于0），同时会将这些句柄也记录在Binder驱动中，然后再将这些句柄和名字发送给ServiceManager，由ServiceManager来维护。

也就是说，Server们通过和Binder驱动通信，Binder驱动做了如下两件事：

1）在内核空间中创建了一个Binder节点，隶属于Server们的进程。

2）驱动为这些Binder节点分配一个大于0的包柄，将这些句柄和名字发送给ServiceManager。

那么Client们是怎么和Binder驱动通信的呢？

很显然，无论如何，Client得知道要跟谁通信，它们知道一个名字。

所以，

1）它们就会把想要获取服务的名字，加上一个句柄为 0 的值，封装为一个数据包，打开Binder设备文件，将这个数据发送给Binder驱动。

2）Binder驱动接收到句柄为0，就会将这数据包扔给ServiceManager。

3）ServiceManager接收到这个数据包，就会分析，发现是要找某个名字的服务，于是就找找找，然后将对应服务的句柄发送回来（某大于0的句柄）。

4）驱动再将这个句柄发回给Client。

5）Client获取句柄之后，就会再加上想要的服务，还有这个句柄，再发送给Binder驱动，Binder驱动就会找到对应的句柄，然后。。。。


**参考链接**      

[跟面试官讲Binder（零） - 持剑 - 博客频道 - CSDN.NET](http://blog.csdn.net/linmiansheng/article/details/37918333)

[跟面试官讲Binder（一） - 持剑 - 博客频道 - CSDN.NET](http://blog.csdn.net/linmiansheng/article/details/41518627)

[跟面试官讲Binder（二）之关于AIDL的认识 - 持剑 - 博客频道 - CSDN.NET](http://blog.csdn.net/linmiansheng/article/details/42438813)

[Binder学习指南 - 简书](http://www.jianshu.com/p/af2993526daf?from=groupmessage&isappinstalled=0)

**Binder学习路径**             

- 看Android文档，Parcel, IBinder, Binder等涉及到跨进程通信的类；
- 不依赖AIDL工具，手写远程Service完成跨进程通信
- 看[《Binder设计与实现》](http://blog.csdn.net/universus/article/details/6211589)
- 看老罗的[博客](http://blog.csdn.net/luoshengyang/article/details/6618363)或者书（书结构更清晰）
- 再看[《Binder设计与实现》](http://blog.csdn.net/universus/article/details/6211589)
- 学习Linux系统相关知识；自己看源码。

