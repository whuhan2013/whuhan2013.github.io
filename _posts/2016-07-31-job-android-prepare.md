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


**如何避免内存泄漏** 

- 避免在 activity 或 fragment 之外传递 Context 对象。
永远永远不要创建静态的 Context 或 View 对象，或者将二者存储于静态变量中。这是内存泄露的首要标志。

private static TextView textView; //DO NOT DO THIS
private static Context context; //DO NOT DO THIS

- 总是记得在 onPause() 或 onDestroy() 方法中的取消注册监听器(listeners)。这包括 Android
监听器，以及位置服务、显示管理器服务，还有自定义的一些监听器。

- 不要在 AsyncTasks(异步任务)或后台线程中存储指向 activities 的强引用。Activity 可能会关闭，但是
AsyncTask 会继续执行，一直保存着对该 activity 的引用。

如果可以，使用 Context-application (getApplicationContext())，而不是某个 activity
的 Context 对象。

- 尽力避免使用非静态的内部类。将引用存储至某个 Activity 或 View 内部会导致内存泄露。如果不得不存储引用，请使用
WeakReference。

文／OneAPM_Official（简书作者）
原文链接：http://www.jianshu.com/p/a8c6b4827333
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。        


**内存泄露探测工具** 

MAT的使用

- 安装MAT
- 配置运行参数：VM参数-XX:+HeapDumpOnOutOfMemoryError
- 导入hprof文件

参考链接

[性能分析工具之-- Eclipse Memory Analyzer tool(MAT)（二） - 骆骆的博客 - 博客频道 - CSDN.NET](http://blog.csdn.net/rachel_luo/article/details/8992461)

[查找并修复Android中的内存泄露—OutOfMemoryError - 简书](http://www.jianshu.com/p/a8c6b4827333)


**2、线程通信基础流程分析**          
当我们调用handler.sendMessage(msg)方法发送一个Message时，实际上这个Message是发送到与当前线程绑定的一个MessageQueue中，然后与当前线程绑定的Looper将会不断的从MessageQueue中取出新的Message，调用msg.target.dispathMessage(msg)方法将消息分发到与Message绑定的handler.handleMessage()方法中。

一个Thread对应多个Handler 一个Thread对应一个Looper和MessageQueue，Handler与Thread共享Looper和MessageQueue。 Message只是消息的载体，将会被发送到与线程绑定的唯一的MessageQueue中，并且被与线程绑定的唯一的Looper分发，被与其自身绑定的Handler消费。

参考链接：

不能在子线程中更新主线程UI，所以如果是子线程中的handler不能更新，但可以用              
handler = new MyHandler(Looper.getMainLooper());                 
就可以更新了。

[Handler一定要在主线程实例化吗?new Handler()和new Handler(Looper.getMainLooper())的区别](http://blog.csdn.net/thanklife/article/details/17006865)

[Android之Handler有感(三) - lee0oo0 - 博客园](http://www.cnblogs.com/lee0oo0/archive/2012/04/06/2434985.html)

[android中looper的实现原理，为什么调用looper.prepare就在当前线程关联了一个lo_百度知道](http://zhidao.baidu.com/link?url=nO2k2ZJG6RFaoLvggfWgu-cNAv7v4OhLLkiVWYROYYJayzD2T1iba0IlT6BYP2ZiBAd4BbHDKUZa_hdEpEuYPcKNaJ9s5W11btKghG6jyJC)


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


**主apk如何加载插件apk中的activity**      

启动一个activity的过程 

(1) 我们app中是使用了Activity的startActivity方法，具体调用的是ContextImpl的同名函数。

(2) ContextImpl中会调用Instrumentation的execStartActivity方法。

(3) Instrumentation通过aidl进行跨进程通信，最终调用AMS的startActivity方法。

(4) 在系统进程中AMS中先判断权限，然后通过调用ActivityStackSupervisor和ActivityStack进行一系列的交互用来确定Activity栈的使用方式。

(5) 通过ApplicationThread进行跨进程通信，转回到app进程。

(6) 通过ActivityThread中的H（一个handler）传递消息，最终调用Instrumentation来创建一个Activity。      


插件化的Activity如何加载进去  

首先我们要明确，要解决的核心问题是［如何能让没有在manifst中注册的Activity能启动起来］。由于权限验证机制是系统做的，我们肯定是没办法修改的，既然我们没办法修改，那是不是考虑去欺骗呢？也就是说可以在manifest中预先定义好几个Activity，俗称占坑，比如名字就叫ActivityA，ActivityB，在校验权限之前把我们插件apk中的Activity替换成定义好的Activity，这样就能顺利通过校验，而在之后真正生成Activity的地方再换回来，瞒天过海。   


那怎么去欺骗呢？回归前面的代码，其实答案已经呼之欲出了——我们可以有两种选择，hook Instrumentation或者hook ActivityManagerNative。这也正好对应了Small和DroidPlugin的实现方案。 

参考链接：[Android插件机制](http://zjutkz.net/2016/05/06/%E8%AE%A9%E6%88%91%E4%BB%AC%E6%9D%A5%E8%81%8A%E4%B8%80%E8%81%8A%E6%8F%92%E4%BB%B6%E5%8C%96%E5%90%A7/)

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
　　　　
OOM内存溢出（Out Of Memory） 　　　　　　　　　　　　　　
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



**16、Retrofit基本构造**                            
Retrofit使用的最核心的两个技术：动态代理和反射。                     
其实Retrofit无非就是让用户创建接口，使用自己指定的规则进行网络访问，把接口传入Retrofit，接口上附着的规则由Retrofit进行层层解析后，再进行实际的网络调用。Retrofit所做的事情就是帮助用户简化了大量的网络访问代码，用户只需写少量代码就能得到想要的结果。如果你懂OO设计，了解一定的设计模式且熟悉JAVA的API，那么你完全可以自己写一个类似Retrofit一样的网络框架！          


**17、Android实现推送方式解决方案**              

几种常见的解决方案实现原理：
 
1）轮询(Pull)方式：应用程序应当阶段性的与服务器进行连接并查询是否有新的消息到达，你必须自己实现与服务器之间的通信，例如消息排队等。而且你还要考虑轮询的频率，如果太慢可能导致某些消息的延迟，如果太快，则会大量消耗网络带宽和电池。
 
 
2）SMS(Push)方式：在Android平台上，你可以通过拦截SMS消息并且解析消息内容来了解服务器的意图，并获取其显示内容进行处理。这是一个不错的想法，我就见过采用这个方案的应用程序。这个方案的好处是，可以实现完全的实时操作。但是问题是这个方案的成本相对比较高，我们需要向移动公司缴纳相应的费用。我们目前很难找到免费的短消息发送网关来实现这种方案。
 
 
 
3）持久连接(Push)方式：这个方案可以解决由轮询带来的性能问题，但是还是会消耗手机的电池。IOS平台的推送服务之所以工作的很好，是因为每一台手机仅仅保持一个与服务器之间的连接，事实上C2DM也是这么工作的。不过刚才也讲了，这个方案存在着很多的不足之处，就是我们很难在手机上实现一个可靠的服务，目前也无法与IOS平台的推送功能相比。      
长连接，发送心跳包，维持service，守护进程来保持service不被杀死
 
 
Android操作系统允许在低内存情况下杀死系统服务，所以我们的推送通知服务很有可能就被操作系统Kill掉了。 轮询(Pull)方式和SMS(Push)方式这两个方案也存在明显的不足。至于持久连接(Push)方案也有不足，不过我们可以通过良好的设计来弥补，以便于让该方案可以有效的工作。毕竟，我们要知道GMail，GTalk以及GoogleVoice都可以实现实时更新的。
 

参考：[Android实现推送方式解决方案 - Healtheon - 博客园](http://www.cnblogs.com/hanyonglu/archive/2012/03/04/2378971.html)
　　

**18、心跳包与长连接**                 

**为什么Android维护长连接需要心跳机制?**              

首先我们知道，维护任何一个长连接都需要心跳机制，客户端发送一个心跳给
服务器，服务器给客户端一个心跳应答，这样就形成客户端服务器的一次完整的握手，这个握手是让双方都知道他们之间的连接是没有断开，客户端是在线
的。如果超过一个时间的阈值，客户端没有收到服务器的应答，或者服务器没有收到客户端的心跳，那么对客户端来说则断开与服务器的连接重新建立一个
连接，对服务器来说只要断开这个连接即可。

那么在智能手机上的长连接心跳和在Internet上的长连接心跳有什么不同的目的呢？

原因就在于智能手机使用的
是移动无线网络，那么我们在讲长连接之前我们首先要了解无线移动网络的特点。

**无线移动网络的特点：**      

当一台智能手机连上移动网络时，其实并没有真正连接上Internet，运营商分配给手机的IP其实是运营商的内网IP，手机终端要连接上Internet还必须通过运营
商的网关进行IP地址的转换，这个网关简称为NAT(NetWork Address Translation)，简单来说就是手机终端连接Internet 其实就是移动内网IP，端口，外网IP之间
相互映射。相当于在手机终端在移动无线网络这堵墙上打个洞与外面的Internet相连。原理图如下：(来源网络)
![](http://img.blog.csdn.net/20130924221600546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemt3MTIzNTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)             

GGSN(GateWay GPRS Support Note 网关GPRS支持节点)模块就实现了NAT功能，由于大部分的移动无线网络运营商为了减少网关NAT映射表的负荷，如
果一个链路有一段时间没有通信时就会删除其对应表，造成链路中断，正是这种刻意缩短空闲连接的释放超时，原本是想节省信道资源的作用，没想到让互联网
的应用不得以远高于正常频率发送心跳来维护推送的长连接。


**19、RXJava与EventBus**                  
EventBus是一个发布 / 订阅的事件总线。简单点说，就是两人约定好怎么通信，一人发布消息，另外一个约定好的人立马接收到你发的消息。

Rx：函数响应式编程 ，响应式代码的基本组成部分是Observables和Subscribers（事实上Observer才是最小的构建块，但实践中使用最多的是Subscriber，因为Subscriber才是和Observables的对应的。）。Observable发送消息，而Subscriber则用于消费消息。

主要区别是，rx里面当建立起订阅关系时，你可以用操作符做任何处理（比如转换数据，更改数据等等），而且他能处理异步的操作。 eventbus 就相当于广播，发送了，总能接收到，他在发送后是不能做任何的数据改变，如果要改变，又要重新post一次。


**20、Android性能优化**               

- 布局优化         
布局优化的思想很简单，减少布局文件的层级                 
- 绘制优化             
绘制优化是指View的onDraw方法要避免执行大量的操作：                       
在onDraw中不要创建新的局部对象，这是因为onDraw方法可能会被频繁调用，这样就会在一瞬间产生大量的临时对象，这不仅占用了过多的内存而且还会导致系统更加频繁的gc，降低了程序的执行效率。                     
onDraw方法中不要指定耗时任务，也不能执行成千上万次的循环操作，View的绘制帧率保证60fps是最佳的，这就要求每帧的绘制时间不超过16ms，虽然程序很难保证16ms这个时间，但是尽量降低onDraw方法的复杂度总是切实有效的。                
- 内存泄漏优化                   
可能导致内存泄漏的场景很多，例如静态变量、单例模式、属性动画、AsyncTask、Handler等等        
- ListView和Bitmap优化           
ListView优化：采用ViewHolder并避免在getView方法中执行耗时操作；根据列表的滑动状态来绘制任务的执行效率；可以尝试开启硬件加速期来使ListView的滑动更加流畅。                   
Bitmap优化：根据需要对图片进行采样，主要是通过BitmapFactory.Options来根据需要对图片进行采样，采样主要用到了BitmapFactory.Options的inSampleSize参数。      
- 线程优化               
采用线程池，避免程序中存在大量的Thread。线程池可以重用内部的线程，从而避免了线程的创建和销毁所带来的性能开销，同时线程池还能有效的控制线程池的最大并发数，避免大量的线程因互相抢占系统资源从而导致阻塞现象的发生。


**更加详细版本参见**         

[PPT分享：老司机在实际项目中是如何做性能优化的](http://mp.weixin.qq.com/s?__biz=MzIwNjQ1NzQxNA==&mid=2247483791&idx=1&sn=e89655613e19262036b659cba04f54f9&scene=1&srcid=0818vZyVqodLcQspghgIfT5T&from=groupmessage&isappinstalled=0#wechat_redirect)             

**21、onSaveInstanceState()和onRestoreInstanceState()调用的过程和时机**

当某个activity变得“容易”被系统销毁时，该activity的onSaveInstanceState就会被执行，除非该activity是被用户主动销毁的，例如当用户按BACK键的时候。 

注意上面的双引号，何为“容易”？言下之意就是该activity还没有被销毁，而仅仅是一种可能性。这种可能性有哪些？通过重写一个activity的所有生命周期的onXXX方法，包括onSaveInstanceState和onRestoreInstanceState方法，我们可以清楚地知道当某个activity（假定为activityA）显示在当前task的最上层时，其onSaveInstanceState方法会在什么时候被执行，有这么几种情况：             

1、当用户按下HOME键时。              
这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。以下几种情况的分析都遵循该原则        
2、长按HOME键，选择运行其他的程序时。                    
3、按下电源按键（关闭屏幕显示）时。             
4、从activity A中启动一个新的activity时。                
5、屏幕方向切换时，例如从竖屏切换到横屏时。                   

总而言之，onSaveInstanceState的调用遵循一个重要原则，即当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据（当然你不保存那就随便你了）。 

至于onRestoreInstanceState方法，需要注意的是，onSaveInstanceState方法和onRestoreInstanceState方法“不一定”是成对的被调用的，onRestoreInstanceState被调用的前提是，activity A“确实”被系统销毁了，而如果仅仅是停留在有这种可能性的情况下，则该方法不会被调用，例如，当正在显示activity A的时候，用户按下HOME键回到主界面，然后用户紧接着又返回到activity A，这种情况下activity A一般不会因为内存的原因被系统销毁，故activity A的onRestoreInstanceState方法不会被执行。 
另外，onRestoreInstanceState的bundle参数也会传递到onCreate方法中，你也可以选择在onCreate方法中做数据还原。 


**22、onNewIntent调用时机**          

```
Activity启动模式设置：

        <activity android:name=".MainActivity" android:launchMode="standard" />

Activity的四种启动模式：

    1. standard

        默认启动模式，每次激活Activity时都会创建Activity，并放入任务栈中。

    2. singleTop

        如果在任务的栈顶正好存在该Activity的实例， 就重用该实例，否者就会创建新的实例并放入栈顶(即使栈中已经存在该Activity实例，只要不在栈顶，都会创建实例)。

    3. singleTask

        如果在栈中已经有该Activity的实例，就重用该实例(会调用实例的onNewIntent())。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移除栈。如果栈中不存在该实例，将会创建新的实例放入栈中。 

    4. singleInstance

        在一个新栈中创建该Activity实例，并让多个应用共享改栈中的该Activity实例。一旦改模式的Activity的实例存在于某个栈中，任何应用再激活改Activity时都会重用该栈中的实例，其效果相当于多个应用程序共享一个应用，不管谁激活该Activity都会进入同一个应用中。
```

大家遇到一个应用的Activity供多种方式调用启动的情况，多个调用希望只有一个Activity的实例存在，这就需要Activity的onNewIntent(Intent intent)方法了。只要在Activity中加入自己的onNewIntent(intent)的实现加上Manifest中对Activity设置lanuchMode=“singleTask”就可以。

onNewIntent（）非常好用，Activity第一启动的时候执行onCreate()---->onStart()---->onResume()等后续生命周期函数，也就时说第一次启动Activity并不会执行到onNewIntent(). 而后面如果再有想启动Activity的时候，那就是执行onNewIntent()---->onResart()------>onStart()----->onResume().  如果android系统由于内存不足把已存在Activity释放掉了，那么再次调用的时候会重新启动Activity即执行onCreate()---->onStart()---->onResume()等。

当调用到onNewIntent(intent)的时候，需要在onNewIntent() 中使用setIntent(intent)赋值给Activity的Intent.否则，后续的getIntent()都是得到老的Intent。


**23、 知道哪几种类型的广播，说一下它们的区别**　　　　　　　　　　

根据广播的发送方式，可以将其分为以下几种类型：

- Normal Broadcast：普通广播
此处将普通广播界定为：开发者自己定义的intent，以context.sendBroadcast_"AsUser"(intent, ...)形式
- System Broadcast: 系统广播
Android系统中内置了多个系统广播，只要涉及到手机的基本操作，基本上都会发出相应的系统广播。如：开启启动，网络状态改变，拍照，屏幕关闭与开启，点亮不足等等。每个系统广播都具有特定的intent-filter，其中主要包括具体的action，系统广播发出后，将被相应的BroadcastReceiver接收。系统广播在系统内部当特定事件发生时，有系统自动发出。
- Ordered broadcast：有序广播
有序广播的有序广播中的“有序”是针对广播接收者而言的，指的是发送出去的广播被BroadcastReceiver按照先后循序接收。有序广播的定义过程与普通广播无异，只是其的主要发送方式变为：sendOrderedBroadcast(intent, receiverPermission, ...)。
- Sticky Broadcast：粘性广播(在 android 5.0/api 21中deprecated,不再推荐使用，相应的还有粘性有序广播，同样已经deprecated)
既然已经deprecated，此处不再多做总结。
- Local Broadcast：App应用内广播
App应用内广播可以理解成一种局部广播的形式，广播的发送者和接收者都同属于一个App




**24、Android中ClassLoader和java中有什么关系和区别**       

java中使用java虚拟机，android中使用Dalvik虚拟机，它可以支持已转换为 .dex（即Dalvik Executable）格式的Java应用程序的运行，.dex格式是专为Dalvik设计的一种压缩格式，适合内存和处理器速度有限的系统。Dalvik 经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik 应用作为一个独立的Linux 进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

java class装载的过程    

首先何时需要加载类呢？ 

有一下集中情况：             
1. 第一次去访问类的静态变量             
2. 第一次去实例化类                                                        
3. 第一次调用Class.forName()                         
4. 遇到以上三种情况时，该类的所有祖先类都会被加载                     


加载class的过程主要分为以下几步：                     
1. 将二进制的class文件装入虚拟机，当然某个class的装载是由它的classloader来完成的。（之后会详细介绍）   
2. 验证该类的语法            
3. 准备阶段，该阶段是负责给该类分配变量空间，以及一些默认值的处理。          
4. 分析（可选），负责把常量池中的应用转化为直接应用              
5. 初始化，该步是执行类中的static变量和static的语句块               


**classloader**

![](http://my.csdn.net/uploads/201205/22/1337696409_6114.jpg)

从上图可以看出，两者有很多相同点，都是树状结构，都有一个根classloader叫做Bootstrap classloader，而且他们都采用了双亲代理模式，什么叫双亲代理模式呢？
所谓双亲代理模式就是装载一个类时，先由自己定义的类装载器请求其parent装载，parent再请求它自己的parent装载，直到顶级的Bootstrap ClassLoader。 若某一级的parent能装载则装载之，否则由它的“下级”自己尝试装载。


**Android程序比起一般Java程序在使用动态加载时麻烦在哪里**

通过上面的分析，我们知道使用ClassLoader动态加载一个外部的类是非常容易的事情，所以很容易就能实现动态加载新的可执行代码的功能，但是比起一般的Java程序，在Android程序中使用动态加载主要有两个麻烦的问题：

- Android中许多组件类（如Activity、Service等）是需要在Manifest文件里面注册后才能工作的（系统会检查该组件有没有注册），所以即使动态加载了一个新的组件类进来，没有注册的话还是无法工作；
- Res资源是Android开发中经常用到的，而Android是把这些资源用对应的R.id注册好，运行时通过这些ID从Resource实例中获取对应的资源。如果是运行时动态加载进来的新类，那类里面用到R.id的地方将会抛出找不到资源或者用错资源的异常，因为新类的资源ID根本和现有的Resource实例中保存的资源ID对不上；
说到底，抛开虚拟机的差别不说，一个Android程序和标准的Java程序最大的区别就在于他们的上下文环境（Context）不同。Android中，这个环境可以给程序提供组件需要用到的功能，也可以提供一些主题、Res等资源，其实上面说到的两个问题都可以统一说是这个环境的问题，而现在的各种Android动态加载框架中，核心要解决的东西也正是“如何给外部的新类提供上下文环境”的问题。

**参考链接**   

[java和android classloader分析 - donway的日志 - eoe 移动开发者论坛 - Powered by Discuz!](http://www.eoeandroid.com/blog-626979-3065.html)        
[Android动态加载基础 ClassLoader工作机制 - 中二病也要开发ANDROID - SegmentFault](https://segmentfault.com/a/1190000004062880)


**25、Android动画**                                                            
在Android中开发动效有两套框架可以使用，分别为 Animation 和 Property Animation；    

- 区别一：需要的Anroid API level不一样 Property Animation需要Android API level 11的支持,当然可以使用nineoldandroids.jar进行兼容，而View Animation则是最原始的版本。
- 区别二：适用范围不一样 Property Animation适用于所有的Object对象，而View Animation则只能应用于View对象。
- 区别三，实际改变不一样： Animation：改变的是view的绘制位置，而非实际属性； 
Property Animation：改变的是view的实际属性； 
如：用两套框架分别对一个button做位移动画，用animation处理之后的点击响应会在原处，而用Property Animation处理后的点击响应会在最终位置处；


Animation分为帧动画，补间动画。

**Property Animation**

属性动画，它更改的是对象的实际属性，在View Animation（Tween Animation）中，其改变的是View的绘制效果，真正的View的属性保持不变，比如无论你在对话中如何缩放Button的大小，Button的有效点击区域还是没有应用动画时的区域，其位置与大小都不变。而在Property Animation中，改变的是对象的实际属性，如Button的缩放，Button的位置与大小属性值都改变了。而且Property Animation不止可以应用于View，还可以应用于任何对象。Property Animation只是表示一个值在一段时间内的改变，当值改变时要做什么事情完全是你自己决定的。 在Property Animation中，可以对动画应用以下属性：



**26、ListView的优化**        

1. ListView需要设置adapter,它的item是通过adapter的方法getView(int position, View convertView, ViewGroup parent)获得的。

2. ListView中只有第一屏的item需要新建，它的引用会被存在RecycleBin对象内，在拖动时后面的item实际上是重从了之前创建的item。

3. 根据上述，ListView在需要显示item时，最开始第一屏时，getView(int position, View convertView, ViewGroup parent )的第二个参数为null,显示第二屏或者回滚显示第一屏时，getView(int position, View convertView, ViewGroup parent )第二个参数是一个原来缓存的item,我们只需要在getView中把它内部数据更新即可。

4. 如果item结构比较复杂，在更新一个已有的item内部数据的时候，查找item内部每一个元素也需要占用不少资源，所以，可以把这些内部元素的引用缓存起来，直接对其赋值，最有效的方法是把这些引用存到对应的item中，比较好的方法是使用setTag()方法。


**27、invalidate与postvalidate区别**   

Android中实现view的更新有两组方法，一组是invalidate，另一组是postInvalidate，其中前者是在UI线程自身中使用，而后者在非UI线程中使用。 

Android提供了Invalidate方法实现界面刷新，但是Invalidate不能直接在线程中调用，因为他是违背了单线程模型：Android UI操作并不是线程安全的，并且这些操作必须在UI线程中调用。 

参考：[android中Invalidate和postInvalidate的区别 - tt_mc - 博客园](http://www.cnblogs.com/tt_mc/archive/2012/01/30/2332023.html)

[为什么说android UI操作不是线程安全的 - LVXIANGAN的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/lvxiangan/article/details/39504145)



**28、Relativelayout与LinearLayout性能，绘制上的区别**            

使用 LinearLayout 容易产生多层嵌套的布局，这会降低布局的性能。而RelativeLayout从使用上来讲，通常层级结构都比较扁平，使用LinearLayout的情况可以用一个RelativeLayout 来替换，以降低布局的层级。另外，RelativeLayout的使用要更灵活一些，作为根布局更容易满足各种情况。这应该就是Google在根布局中使用RelativeLayout的原因。

Relativelayout性能更好，更灵活。因为使用LinearLayout 容易产生多层嵌套的布局结构，而因为Relativelayout的灵活性的优点，可以降低布局的嵌套层级，优化性能，因此当嵌套多时推荐使用RelativeLayout。

RelativeLayout与LinearLayout都继承于ViewGroup，而ViewGroup实现了android.view.ViewParent和android.view.ViewManager接口，赋予了其装载控件和管理子控件的能力。例如ViewParent中的requestLayout()与ViewManager中的addView(View view, ViewGroup.LayoutParams params)

ViewGroup的作用是组织和管理它的子View，即对子View进行布局，让子View绘制自身并对它们的大小、边距进行约束等。ViewGroup管理View的基本过程是onMeasure()->onLayout()，RelativeLayout与LinearLayout对子View绘制与布局的区别就大部分就在这两个函数中


LinearLayout的onMeasure：                  
onMeasure的作用是遍历所有子View，对其大小进行测量。            

接下来是RelativeLayout               
与LinearLayout相同，RelativeLayout同样继承于ViewGroup，同样需要经过onMeasure与onLayout。

首先是onMeasure：                    
当第一次执行onMeasure或者执行requestLayout后，需要调用sortChildren方法，根据添加顺序对所有的子View进行排序，横着一次(mSortedHorizontalChildren)，竖着一次（mSortedVerticalChildren），然后对两个序列进行检查，通过依赖图静态类DependencyGraph中的getSortedViews方法根据依赖关系进行排序。          


因为对于LinearLayout，他在onMeasure方法中对子View是遍历测量的，所以在层级关系比较复杂，尤其是LinearLayout嵌套的页面，底层子View就会被测量好多次！遍历测绘是很耗费性能的一件事。
如果你没看过源码，那么我想你在应用LinearLayout和RelativeLayout的时候总会遇到这样一种情况，RelativeLayout在布局中上面的子View无法引用下面的子View，而LinearLayout可以，其实，就是这个道理。


**参考：**[LinearLayout与RelativeLayout异同深入探讨 - - 博客频道 - CSDN.NET](http://blog.csdn.net/oShunz/article/details/50425844)


**29、RXJava与EventBus的区别与使用场景**        

EventBus比较适合仅仅当做组件间的通讯工具使用，主要用来传递消息。使用EventBus可以避免搞出一大推的interface，仅仅是为了实现组件间的通讯，而不得不去实现那一推的接口，eventbus基于反射

RxJava,函数响应式编程,基于观察者模式，使用的场景确实异步数据流的处理,异步操作很关键的一点是程序的简洁性，因为在调度过程比较复杂的情况下，异步代码经常会既难写也难被读懂。 Android 创造的 AsyncTask 和Handler ，其实都是为了让异步代码更加简洁。RxJava 的优势也是简洁，但它的简洁的与众不同之处在于，随着程序逻辑变得越来越复杂，它依然能够保持简洁。



**Fragment与Activity，Fragment消息传递，Fragment切换状态保存**          

**Fragment间的通信**

现实中我们经常想要一个Fragment跟另一个Fragment进行通信，例如，要基于一个用户事件来改变内容。所有的Fragment间的通信都是通过跟关联的Activity来完成的。另个Fragment不应该直接通信。也就是说Fragment间不直接通信，通过Activity转一下，按java常规，转一下多是使用Interface实现的。

定义Interface

为了让Fragment跟它的Activity通信，你可以在Fragment类中定义一个接口，并在它所属的Activity中实现该接口。Fragment在它的onAttach()方法执行期间捕获该接口的实现，然后就可以调用接口方法，以便跟Activity通信。

```
public class HeadlinesFragment extends ListFragment {
    OnHeadlineSelectedListener mCallback;

    // Container Activity must implement this interface
    public interface OnHeadlineSelectedListener {
        public void onArticleSelected(int position);
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        
        // This makes sure that the container activity has implemented
        // the callback interface. If not, it throws an exception
        try {
            mCallback = (OnHeadlineSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        }
    }
    
    ...
}

@Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        // Send the event to the host activity
        mCallback.onArticleSelected(position);
    }
```


把消息传递给另一个Fragment
通过使用findFragmentById()方法捕获Fragment实例，宿主Activity可以把消息发送给该Fragment，然后直接调用该Fragment的公共方法。
例如，上面的示例，Activty通过Interface的实现方法，传递数据到另一个Fragment。


```
public static class MainActivity extends Activity
        implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...

    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article

        ArticleFragment articleFrag = (ArticleFragment)
                getSupportFragmentManager().findFragmentById(R.id.article_fragment);

        if (articleFrag != null) {
            // If article frag is available, we're in two-pane layout...

            // Call a method in the ArticleFragment to update its content
            articleFrag.updateArticleView(position);
        } else {
            // Otherwise, we're in the one-pane layout and must swap frags...

            // Create fragment and give it an argument for the selected article
            ArticleFragment newFragment = new ArticleFragment();
            Bundle args = new Bundle();
            args.putInt(ArticleFragment.ARG_POSITION, position);
            newFragment.setArguments(args);
        
            FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

            // Replace whatever is in the fragment_container view with this fragment,
            // and add the transaction to the back stack so the user can navigate back
            transaction.replace(R.id.fragment_container, newFragment);
            transaction.addToBackStack(null);

            // Commit the transaction
            transaction.commit();
        }
    }
}
```

**参考链接**        
[Android UI开发第二十六篇——Fragment间的通信 - 张兴业的博客 - 博客频道 - CSDN.NET](http://blog.csdn.net/xyz_lmn/article/details/8631195)


**Fragment切换时如何保存状态**

- 如果用的Fragment嵌套的是ViewPager的话就简单点，直接mViewpager.setOffscreenPageLimit(4);

解决方法二：解决方法，在fragment onCreateView 里缓存View: 

```
private View rootView;// 缓存Fragment view

  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container,
      Bundle savedInstanceState)
  {
    Log.i(TAG, "onCreateView");

    if (rootView == null)
    {
      rootView = inflater.inflate(R.layout.fragment_1, null);
    }
    // 缓存的rootView需要判断是否已经被加过parent，如果有parent需要从parent删除，要不然会发生这个rootview已经有parent的错误。
    ViewGroup parent = (ViewGroup) rootView.getParent();
    if (parent != null)
    {
      parent.removeView(rootView);
    }
    return rootView;
  }
```

参考：[关于fragment的缓存 - 一站到底s的博客 - 博客频道 - CSDN.NET](http://blog.csdn.net/yanxiangxue/article/details/52043328)