---
layout: post
title: SystemServer源码解析
date: 2016-8-17
categories: blog
tags: [源码解析]
description: 源码解析
---

Android系统中各个进程的先后顺序为：

init进程 –-> Zygote进程 –> SystemServer进程 –>应用进程

其中Zygote进程由init进程启动，SystemServer进程和应用进程由Zygote进程启动。

本文依据6.0源码，主要分析SystemServer进程的启动流程。注意，是启动流程，不是启动过程。启动过程的解析见前文。SystemServer进程的作用是启动各种核心服务，例如Installer、ActivityManagerService、WindowManagerService、PowerManagerService等等。这些服务会在开机时启动。由于加载的服务很多，相对很耗时，所以Android系统在开机时速度很慢，但是一次加载之后，方便后续所有应用程序调用，所以这个代价还是非常值得的。由于加载的服务实在太多，故本文不可能分析所有的服务。分析完主要流程后，会适当举例分析，其余服务还请自行查看。从Android Zygote启动流程源码解析一文中可以看到：Zygote进程fork出SystemServer进程后，通过反射调用SystemServer#main()。以此为切入点，一步步分析。


### 启动流程概览

源码位置：frameworks/base/services/java/com/android/server/SystemServer.java 

**SystemServer#main()**          

```
 /**
     * The main entry point from zygote.
     */
    public static void main(String[] args) {
        new SystemServer().run();
    }

    public SystemServer() {
        // Check for factory test mode.
        mFactoryTestMode = FactoryTest.getMode();
    }

    private void run() {
        ...
        // 创建主线程Looper
        Looper.prepareMainLooper();
        // 加载android_servers.so
        System.loadLibrary("android_servers");
        // 创建Context对象
        createSystemContext();
        // 创建SystemServiceManager对象
        mSystemServiceManager = new SystemServiceManager(mSystemContext);
        // 将刚创建的SystemServiceManager对象添加进LocalServices属性sLocalServiceObjects
        LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);

        // Start services.
        try {
            // 启动系统引导相关服务
            startBootstrapServices();
            // 启动系统核心服务
            startCoreServices();
            // 启动应用或系统相关的服务
            startOtherServices();
        } catch (Throwable ex) {...}
        ...
        // Loop forever.
        Looper.loop();
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

上面截取了最关键的代码，也是本文重点要分析的内容。可以看到，在Main（）方法中实例化之后直接调用了run()方法。在run()方法中首先获取主线程的Looper对象，这也就意味着SystemServer后续是可以获取主线程中的消息。下面先分析createSystemContext()。

#### 创建Context对象

SystemServer#createSystemContext()


```
 private void createSystemContext() {
        ActivityThread activityThread = ActivityThread.systemMain();
        mSystemContext = activityThread.getSystemContext();
        mSystemContext.setTheme(android.R.style.Theme_DeviceDefault_Light_DarkActionBar);
    }
```

在createSystemContext()中首先获取了一个ActivityThread对象，紧接着根据ActivityThread对象获取了mSystemContext，mSystemContext是Context对象。这可不得了。我们知道，ActivityThread其实就是所谓的主线程，看来ActivityThread#systemMain()这个方法大有搞头。跟进。

源码位置：frameworks/base/core/java/android/app/ActivityThread.java 

**ActivityThread#systemMain()**

