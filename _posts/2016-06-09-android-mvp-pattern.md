---
layout: post
title: MVP模式学习
date: 2016-6-9
categories: blog
tags: [设计模式]
description: MVP模式学习
---

MVP模式（Model-View-Presenter）可以说是MVC模式（Model-View-Controller）在Android开发上的一种变种、进化模式。后者大家可能比较熟悉，就算不熟悉也可能或多或少地在自己的项目中用到过。要介绍MVP模式，就不得不先说说MVC模式。

### MVC模式

MVC模式的结构分为三部分，实体层的Model，视图层的View，以及控制层的Controller。

![](http://7xih5c.com1.z0.glb.clouddn.com/15-10-11/13126761.jpg)


- 其中View层其实就是程序的UI界面，用于向用户展示数据以及接收用户的输入      
- 而Model层就是JavaBean实体类，用于保存实例数据         
- Controller控制器用于更新UI界面和数据实例              

例如，View层接受用户的输入，然后通过Controller修改对应的Model实例；同时，当Model实例的数据发生变化的时候，需要修改UI界面，可以通过Controller更新界面。（View层也可以直接更新Model实例的数据，而不用每次都通过Controller，这样对于一些简单的数据更新工作会变得方便许多。）

举个简单的例子，现在要实现一个飘雪的动态壁纸，可以给雪花定义一个实体类Snow，里面存放XY轴坐标数据，View层当然就是SurfaceView（或者其他视图），为了实现雪花飘的效果，可以启动一个后台线程，在线程里不断更新Snow实例里的坐标值，这部分就是Controller的工作了，Controller里还要定时更新SurfaceView上面的雪花。进一步的话，可以在SurfaceView上监听用户的点击，如果用户点击，只通过Controller对触摸点周围的Snow的坐标值进行调整，从而实现雪花在用户点击后出现弹开等效果。具体的MVC模式请自行Google。


### MVP模式

在Android项目中，Activity和Fragment占据了大部分的开发工作。如果有一种设计模式（或者说代码结构）专门是为优化Activity和Fragment的代码而产生的，你说这种模式重要不？这就是MVP设计模式。

按照MVC的分层，Activity和Fragment（后面只说Activity）应该属于View层，用于展示UI界面，以及接收用户的输入，此外还要承担一些生命周期的工作。Activity是在Android开发中充当非常重要的角色，特别是TA的生命周期的功能，所以开发的时候我们经常把一些业务逻辑直接写在Activity里面，这非常直观方便，代价就是Activity会越来越臃肿，超过1000行代码是常有的事，而且如果是一些可以通用的业务逻辑（比如用户登录），写在具体的Activity里就意味着这个逻辑不能复用了。如果有进行代码重构经验的人，看到1000+行的类肯定会有所顾虑。因此，Activity不仅承担了View的角色，还承担了一部分的Controller角色，这样一来V和C就耦合在一起了，虽然这样写方便，但是如果业务调整的话，要维护起来就难了，而且在一个臃肿的Activity类查找业务逻辑的代码也会非常蛋疼，所以看起来有必要在Activity中，把View和Controller抽离开来，而这就是MVP模式的工作了。


![](http://7xih5c.com1.z0.glb.clouddn.com/15-10-11/2114527.jpg)


**MVP模式的核心思想**

MVP把Activity中的UI逻辑抽象成View接口，把业务逻辑抽象成Presenter接口，Model类还是原来的Model。

这就是MVP模式，现在这样的话，Activity的工作的简单了，只用来响应生命周期，其他工作都丢到Presenter中去完成。从上图可以看出，Presenter是Model和View之间的桥梁，为了让结构变得更加简单，View并不能直接对Model进行操作，这也是MVP与MVC最大的不同之处。

**MVP模式的作用**

- 分离了视图逻辑和业务逻辑，降低了耦合      
- Activity只处理生命周期的任务，代码变得更加简洁     
- 视图逻辑和业务逻辑分别抽象到了View和Presenter的接口中去，提高代码的可阅读性        
- Presenter被抽象成接口，可以有多种具体的实现，所以方便进行单元测试     
- 把业务逻辑抽到Presenter中去，避免后台线程引用着Activity导致Activity的资源无法被系统回收从而引起内存泄露和OOM  

**其中最重要的有三点**

- Activity 代码变得更加简洁         
- 方便进行单元测试             
- 避免 Activity 的内存泄露   


### MVP模式的使用

![](http://7xih5c.com1.z0.glb.clouddn.com/15-10-12/94032090.jpg)


**首先来看两个Base接口类**            
BaseView与BasePresenter，两类分别是所有View和Presenter的基类。

```
public interface BaseView<T> {
    // 规定View必须要实现setPresenter方法，则View中保持对Presenter的引用。
    void setPresenter(T presenter);
}
``` 

setPresenter的调用时机是presenter实现类的构造函数中，如此View中的事件请求便通过调用presenter来实现。

```
public interface BasePresenter {
    // 规定Presenter必须要实现start方法。
    void start();
}
```

该方法的作用是Presenter开始获取数据并调用View的方法来刷新界面，其调用时机是在Fragment类的onResume方法中。


**定义了契约类（接口)** 

使用契约类来统一管理view与presenter的所有的接口，这种方式使得view与presenter中有哪些功能，一目了然，维护起来也很方便。

```
public interface GirlContract {
    interface View extends BaseView<Presenter> {
        public void showMore(@Nullable List list);

        public void finishRefresh();

        public void showSnackBar();
    }

    interface Presenter extends BasePresenter {

    }
}
```


**Activity或Fragment在mvp中的作用**      

Activity在项目中是一个全局的控制者，负责创建view以及presenter实例，并将二者联系起来。TaskDetailActivity 的onCreate()回调中创建TaskDetailPresenter 实例，TaskDetailPresenter 的构造函数中实现了View和Presenter的关联.

在Fragment或Activity中实现View接口，实例化Present实现类。


```
private GirlContract.Presenter mPresenter = new GirlPresenter(this);
```


```
public GirlPresenter(GirlContract.View baseView) {
        this.mView = baseView;
        init();
    }
```


**通过以上的分析，我们可以看到：**

- 工程的整体架构和代码结构非常清晰（不再是所有的业务和逻辑都糅合在Activity、Fragment里了），易于理解和上手。    
- 由于将UI代码与业务代码进行了拆分，整体的可测试性非常的好，UI层和业务层可以分别进行单元测试。   
- 由于架构的引入，虽然代码量有了一定的上升，但是由于界限非常清晰，各个类和层的职责都非常明确且单一，后期的扩展，维护都会更加容易。    

但以上毕竟是架构的Sample，是为了说明架构思想，因此有些地方我们在实际运用中需要注意：数据库和网络的操作都应该放在工作线程，用户回退后需要取消网络请求、回调接口置为null等。



### 参考链接

[ANDROID MVP模式 简单易懂的介绍方式 - MOE STUDIO - 萌の軌跡・願いの叶う場所](http://kaedea.com/2015/10/11/android-mvp-pattern/)


[Android官方MVP架构项目解析 - 简书](http://www.jianshu.com/p/389c9ae1a82c)

