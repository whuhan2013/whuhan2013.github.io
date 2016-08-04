---
layout: post
title: 校招准备之Android       
date: 2016-7-31
categories: blog
tags: [求职]
description: 
---

**1、Android内存泄漏可能原因？**                     

- 集合类泄漏               
集合类如果仅仅有添加元素的方法，而没有相应的删除机制，导致内存被占用。如果这个集合类是全局性的变量 (比如类中的静态属性，全局性的 map 等即有静态引用或 final 一直指向它)，那么没有相应的删除机制，很可能导致集合所占用的内存只增不减。             
- 单例造成的内存泄漏           
由于单例的静态特性使得其生命周期跟应用的生命周期一样长，所以如果使用不恰当的话，很容易造成内存泄漏。比如下面一个典型的例子，                  
- 匿名内部类/非静态内部类和异步线程            
非静态内部类默认会持有外部类的引用。               
- Handler 造成的内存泄漏           
由于 Handler 属于 TLS(Thread Local Storage) 变量, 生命周期和 Activity 是不一致的。因此这种实现方式一般很难保证跟 View 或者 Activity 的生命周期保持一致，故很容易导致无法正确释放。                         
解决方法：即推荐使用静态内部类 + WeakReference 这种方式。每次使用前注意判空。                     

**2、线程通信基础流程分析**          
当我们调用handler.sendMessage(msg)方法发送一个Message时，实际上这个Message是发送到与当前线程绑定的一个MessageQueue中，然后与当前线程绑定的Looper将会不断的从MessageQueue中取出新的Message，调用msg.target.dispathMessage(msg)方法将消息分发到与Message绑定的handler.handleMessage()方法中。

一个Thread对应多个Handler 一个Thread对应一个Looper和MessageQueue，Handler与Thread共享Looper和MessageQueue。 Message只是消息的载体，将会被发送到与线程绑定的唯一的MessageQueue中，并且被与线程绑定的唯一的Looper分发，被与其自身绑定的Handler消费。

**3、插件化技术学习**                         
Android插件化基础                  
- 开发者将插件代码封装成Jar或者APK
- 宿主下载或者从本地加载Jar或者APK到宿主中
- 将宿主调用插件中的算法或者Android特定的Class（如Activity）               

为什么引入动态加载技术？

一个应用程序dex文件的方法数最大不能超过65536个
可以让应用程序实现插件化、插拔式结构，对后期维护有益


什么是动态加载技术

动态加载技术就是使用类加载器加载相应的apk、dex、jar(必须含有dex文件)，再通过反射获得该apk、dex、jar内部的资源（class、图片、color等等）进而供宿主app使用。             


**4、自定义控件**          

- 自定义View的属性
- 在View的构造方法中获得我们自定义View的步骤
- 重写onMeasure(不必须)
- 重写onLayout(不必须)
- 重写onDraw

**5、Android 事件分发机制**             
　　　　
呈Ｕ字型，dispatchTouchEVent从上到下，onTouchEvent从下到上。        


**6、ANR什么时候出现**          
Error          
OOM，内存溢出          
StackOverFlowError            
Runtime,比如说空指针异常         

解决的办法 

注意内存的使用和管理        
使用Thread.UncaughtExceptionHandler接口     

**7、ART和Dalvik区别**             
Art上应用启动快，运行快，但是耗费更多存储空间，安装时间长，总的来说ART的功效就是"空间换时间"。

ART: Ahead of Time Dalvik: Just in Time

什么是Dalvik：Dalvik是Google公司自己设计用于Android平台的Java虚拟机。Dalvik虚拟机是Google等厂商合作开发的Android移动设备平台的核心组成部分之一，它可以支持已转换为.dex(即Dalvik Executable)格式的Java应用程序的运行，.dex格式是专为Dalvik应用设计的一种压缩格式，适合内存和处理器速度有限的系统。Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik应用作为独立的Linux进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

什么是ART:Android操作系统已经成熟，Google的Android团队开始将注意力转向一些底层组件，其中之一是负责应用程序运行的Dalvik运行时。Google开发者已经花了两年时间开发更快执行效率更高更省电的替代ART运行时。ART代表Android Runtime,其处理应用程序执行的方式完全不同于Dalvik，Dalvik是依靠一个Just-In-Time(JIT)编译器去解释字节码。开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运行。ART则完全改变了这套做法，在应用安装的时候就预编译字节码到机器语言，这一机制叫Ahead-Of-Time(AOT)编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。

ART优点：

系统性能的显著提升
应用启动更快、运行更快、体验更流畅、触感反馈更及时
更长的电池续航能力
支持更低的硬件
ART缺点：

更大的存储空间占用，可能会增加10%-20%
更长的应用安装时间

**8、Android关于OOM的解决方案**　　　　　
　　　　
OOM　　　　　　　　　　　　　　　　　

内存溢出（Out Of Memory）　　　　　　　　　　　　　　　
也就是说内存占有量超过了VM所分配的最大　　　　　　　　　　
出现OOM的原因　　　　　　　　　　　　　　

加载对象过大　　　　　　　　　　
相应资源过多，来不及释放　　　　　　

如何解决　　　　　　　　　　

在内存引用上做些处理，常用的有软引用、强化引用、弱引用　　　　　　　
在内存中加载图片时直接在内存中作处理，如边界压缩　　　　　　　　　　　　
动态回收内存　　　　　　　　　　　　　
优化Dalvik虚拟机的堆内存分配　　　　　　　　　　
自定义堆内存大小　　　　　　　　


**9、APP启动过程**　　　　　　　 
![](https://camo.githubusercontent.com/cd3ff70e4f8141ffa6cd84dbb446753562be0a52/687474703a2f2f37786e74646d2e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f61637469766974795f73746172745f666c6f772e706e67)　　 
　
上图就可以很好的说明App启动的过程         

- ActivityManagerService组织回退栈时以ActivityRecord为基本单位，所有的ActivityRecord放在同一个ArrayList里，可以将mHistory看作一个栈对象，索引0所指的对象位于栈底，索引mHistory.size()-1所指的对象位于栈顶
- Zygote进程孵化出新的应用进程后，会执行ActivityThread类的main方法.在该方法里会先准备好Looper和消息队列，然后调用attach方法将应用进程绑定到ActivityManagerService，然后进入loop循环，不断地读取消息队列里的消息，并分发消息。
- ActivityThread的main方法执行后,应用进程接下来通知ActivityManagerService应用进程已启动，ActivityManagerService保存应用进程的一个代理对象，这样ActivityManagerService可以通过这个代理对象控制应用进程，然后ActivityManagerService通知应用进程创建入口Activity的实例，并执行它的生命周期方法


**10、Android图片中的三级缓存**         
什么是三级缓存

网络加载，不优先加载，速度慢，浪费流量     
本地缓存，次优先加载，速度快                 
内存缓存，优先加载，速度最快          

三级缓存原理

首次加载 Android App 时，肯定要通过网络交互来获取图片，之后我们可以将图片保存至本地SD卡和内存中         
之后运行 App 时，优先访问内存中的图片缓存，若内存中没有，则加载本地SD卡中的图片          
总之，只在初次访问新内容时，才通过网络获取图片资源            

**11、热修复技术**             
一个ClassLoader可以包含多个dex文件，每个dex文件是一个Element，多个dex文件排列成一个有序的数组dexElements，当找类的时候，会按顺序遍历dex文件，然后从当前遍历的dex文件中找类，如果找类则返回，如果找不到从下一个dex文件继续查找。          
同时，在虚拟机启动的时候，当verify选项被打开的时候，如果static方法、private方法、构造函数等，其中的直接引用（第一层关系）到的类都在同一个dex文件中，那么该类就会被打上CLASS_ISPREVERIFIED标志。

那么，我们要做的就是，阻止该类打上CLASS_ISPREVERIFIED的标志。


总结下：

其实就是两件事：1、动态改变BaseDexClassLoader对象间接引用的dexElements；2、在app打包的时候，阻止相关类去打上CLASS_ISPREVERIFIED标志。


**11、AIDL与Binder**          
什么是aidl                 
aidl是 Android Interface definition language的缩写，一看就明白，它是一种android内部进程通信接口的描述语言，通过它我们可以定义进程间的通信接口
icp:interprocess communication :内部进程通信         

AIDL:xxx.aidl -> xxx.java ,注册service

用aidl定义需要被调用方法接口      
实现这些方法        
调用这些方法             

AIDL基于Binder

首先Binder是Android系统进程间通信(IPC)方式之一。                  
Binder使用Client－Server通信方式。Binder框架定义了四个角色：Server,Client,ServiceManager以及Binder驱动。

client组件和server组件进行通信，要通过ServiceManager查询server组件的引用。server向ServiceManager注册一个binder实体，client通过ServiceManager获得binder实体的一个引用，这样client和server就可以通信了。

**12、MVC,MVP,MVVM的区别**            

MVC

软件可以分为三部分

视图（View）:用户界面         
控制器（Controller）:业务逻辑          
模型（Model）:数据保存         
各部分之间的通信方式如下：             
 
View传送指令到Controller         
Controller完成业务逻辑后，要求Model改变状态      
Model将新的数据发送到View，用户得到反馈        
Tips：所有的通信都是单向的。              

互动模式

接受用户指令时，MVC可以分为两种方式。一种是通过View接受指令，传递给Controller。

另一种是直接通过Controller接受指令

MVP

MVP模式将Controller改名为Presenter，同时改变了通信方向。

各部分之间的通信，都是双向的         
View和Model不发生联系，都通过Presenter传递    
View非常薄，不部署任何业务逻辑，称为"被动视图"(Passive View)，即没有任何主动性，而Presenter非常厚，所有逻辑都部署在那里。 

MVVM

MVVM模式将Presenter改名为ViewModel，基本上与MVP模式完全一致。

唯一的区别是，它采用双向绑定(data-binding)：View的变动，自动反映在ViewModel，反之亦然。


**13、Android开机过程**           

- BootLoder引导,然后加载Linux内核.
- 0号进程init启动.加载init.rc配置文件,配置文件有个命令启动了zygote进程
- zygote开始fork出SystemServer进程
- SystemServer加载各种JNI库,然后init1,init2方法,init2方法中开启了新线程ServerThread.
- 在SystemServer中会创建一个socket客户端，后续AMS（ActivityManagerService）会通过此客户端和zygote通信
- ServerThread的run方法中开启了AMS,还孵化新进程ServiceManager,加载注册了一溜的服务,最后一句话进入loop 死循环
- run方法的SystemReady调用resumeTopActivityLocked打开锁屏界面


**14、EventBus**            
EventBus是一款针对Android优化的发布/订阅（publish/subscribe）事件总线。主要功能是替代Intent,Handler,BroadCast在Fragment，Activity，Service，线程之间传递消息。简化了应用程序内各组件间、组件与后台线程间的通信。优点是开销小，代码更优雅。以及将发送者和接收者解耦。比如请求网络，等网络返回时通过Handler或Broadcast通知UI，两个Fragment之间需要通过Listener通信，这些需求都可以通过EventBus实现。            

EventBus作为一个消息总线，有三个主要的元素：

Event：事件。可以是任意类型的对象            
Subscriber：事件订阅者，接收特定的事件。在EventBus中，使用约定来指定事件订阅者以简化使用。即所有事件订阅都都是以onEvent开头的函数，具体来说，函数的名字是onEvent,onEventMainThread，onEventBackgroundThread，onEventAsync这四个，这个和 ThreadMode（下面讲）有关。               
Publisher：事件发布者，用于通知 Subscriber 有事件发生。可以在任意线程任意位置发送事件，直接调用eventBus.post(Object) 方法，可以自己实例化 EventBus 对象，但一般使用默认的单例就好了：EventBus.getDefault()， 根据post函数参数的类型，会自动调用订阅相应类型事件的函数。            

前面说了，Subscriber的函数只能是那4个，因为每个事件订阅函数都是和一个ThreadMode相关联的，ThreadMode指定了会调用的函数。有以下四个ThreadMode：

- PostThread：           
事件的处理在和事件的发送在相同的进程，所以事件处理时间不应太长，不然影响事件的发送线程，而这个线程可能是UI线程。对应的函数名是onEvent。
- MainThread:              事件的处理会在UI线程中执行。事件处理时间不能太长，这个不用说的，长了会ANR的，对应的函数名是onEventMainThread。
- BackgroundThread：              
事件的处理会在一个后台线程中执行，对应的函数名是onEventBackgroundThread，虽然名字是BackgroundThread，事件处理是在后台线程，但事件处理时间还是不应该太长，因为如果发送事件的线程是后台线程，会直接执行事件，如果当前线程是UI线程，事件会被加到一个队列中，由一个线程依次处理这些事件，如果某个事件处理时间太长，会阻塞后面的事件的派发或处理。
- Async：               
事件处理会在单独的线程中执行，主要用于在后台线程中执行耗时操作，每个事件会开启一个线程（有线程池），但最好限制线程的数目。



**15、Android启动过程深入解析**               
- 第一步：启动电源以及系统启动            
- 第二步：引导程序        
- 第三步：内核            
- 第四步：init进程      
init是第一个进程，我们可以说它是root进程或者说有进程的父进程。init进程有两个责任，一是挂载目录，比如/sys、/dev、/proc，二是运行init.rc脚本。              
- 第五步：Zygote加载进程   
在Java中，我们知道不同的虚拟机实例会为不同的应用分配不同的内存。假如Android应用应该尽可能快地启动，但如果Android系统为每一个应用启动不同的Dalvik虚拟机实例，就会消耗大量的内存以及时间。因此，为了克服这个问题，Android系统创造了”Zygote”。Zygote让Dalvik虚拟机共享代码、低内存占用以及最小的启动时间成为可能。Zygote是一个虚拟器进程，正如我们在前一个步骤所说的在系统引导的时候启动。Zygote预加载以及初始化核心库类。通常，这些核心类一般是只读的，也是Android SDK或者核心框架的一部分。在Java虚拟机中，每一个实例都有它自己的核心库类文件和堆对象的拷贝。                          
- 第六步：系统服务或服务            
完成了上面几步之后，运行环境请求Zygote运行系统服务。系统服务同时使用native以及java编写，系统服务可以认为是一个进程。同一个系统服务在Android SDK可以以System Services形式获得。系统服务包含了所有的System Services。     
- 第七步：引导完成              
一旦系统服务在内存中跑起来了，Android就完成了引导过程。在这个时候“ACTION_BOOT_COMPLETED”开机启动广播就会发出去。

参考链接：[Android启动过程深入解析 - 文章 - 伯乐在线](http://blog.jobbole.com/67931/)


