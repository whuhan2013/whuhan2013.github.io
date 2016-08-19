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

```
public static ActivityThread systemMain() {
        ...
        ActivityThread thread = new ActivityThread();
        thread.attach(true);
        return thread;
    }
```

在ActivityThread#systemMain()中直接new了一个ActivityThread对象，然后调用ActivityThread#attach()方法。跟进

```
 ActivityThread() {
        mResourcesManager = ResourcesManager.getInstance();
    }

    private void attach(boolean system) {
        sCurrentActivityThread = this;
        mSystemThread = system;
        if (!system) {
           ...
        } else {
            ...
                mInstrumentation = new Instrumentation();
                ContextImpl context = ContextImpl.createAppContext(this, getSystemContext().mPackageInfo);
                mInitialApplication = context.mPackageInfo.makeApplication(true, null);
                mInitialApplication.onCreate();
           ...
        }
       ...
```

可以看到，调用ContextImpl的静态方法createAppContext()获取一个ContextImpl对象，然后调用LoadedApk（mPackageInfo是LoadedApk的一个对象）#makeApplication()创建了Application，最后调用Application#onCreate()方法。SystemServer也是一个Android进程，由此可以看出，进程创建后最先被调用的是ActivityThread#attach()，其次才是Application#onCreate()。


### 创建SystemServiceManager

回到SystemServer#run()。


```
  private void run() {
        ...
         createSystemContext();
        // 创建SystemServiceManager对象
        mSystemServiceManager = new SystemServiceManager(mSystemContext);
        // 将刚创建的SystemServiceManager对象添加进LocalServices属性sLocalServiceObjects
        LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
        ...
    }
```

获取到Context对象后，创建mSystemServiceManager用于管理各种系统服务。接下来就开始启动各种系统服务。文章有限，这里以启动Installer、ActivityManagerService和WindowManagerService为例进行分析，其余还请自行查看


### 启动各种服务

ActivityThread#startBootstrapServices()

```
 startBootstrapServices();
  startCoreServices();
  startOtherServices();
```

**启动Installer**    

```
    private void startBootstrapServices() {
        Installer installer = mSystemServiceManager.startService(Installer.class);
    }
```

通过SystemServiceManager#startService()方法，就启动了Installer。跟进。

源码位置：frameworks/base/services/core/java/com/android/server/SystemServerManager.java 

**SystemServiceManager#startService()**

```
 public <T extends SystemService> T startService(Class<T> serviceClass) {
        if (!SystemService.class.isAssignableFrom(serviceClass)) {
            throw new RuntimeException("Failed to create " + name
                    + ": service must extend " + SystemService.class.getName());
        }
        final T service;
        Constructor<T> constructor = serviceClass.getConstructor(Context.class);
        service = constructor.newInstance(mContext);
        mServices.add(service);
        service.onStart();
        return service;
    }
```

传入的泛型必须是抽象类SystemService的子类，并且通过反射得到实例。最后添加进ArrayList<SystemService> mServices中以便管理。最后调用泛型Service中抽象父类的抽象方法onStart()的具体实现。跟进。


源码位置：frameworks/base/services/core/java/com/android/server/pm/Installer.java 

**Installer#onStart()**

```
  public void onStart() {
        mInstaller.waitForConnection();
    }
```

mInstaller是InstallerConnection对象。跟进。

源码位置：frameworks/base/services/core/java/com/android/internal/os/InstallerConnection.java 

**InstallerConnection#waitForConnection()**

```
  public void waitForConnection() {
        for (;;) {
            if (execute("ping") >= 0) {
                return;
            }
            Slog.w(TAG, "installd not ready");
            SystemClock.sleep(1000);
        }
    }
```

这里有个死循环，说明只有execute()方法执行成功并且返回值大于0之后才会继续启动其他服务。否则一直卡在这里。跟进。


```
  public int execute(String cmd) {
        String res = transact(cmd);
        try {
            return Integer.parseInt(res);
        } catch (NumberFormatException ex) {
            return -1;
        }
    }

    public synchronized String transact(String cmd) {
        if (!connect()) {
            return "-1";
        }

        if (!writeCommand(cmd)) {
            if (!connect() || !writeCommand(cmd)) {
                return "-1";
            }
        }
        final int replyLength = readReply();
        if (replyLength > 0) {
            String s = new String(buf, 0, replyLength);
            return s;
        } else {
            return "-1";
        }
    }

    private boolean connect() {
        if (mSocket != null) {
            return true;
        }
        Slog.i(TAG, "connecting...");
        try {
            mSocket = new LocalSocket();

            LocalSocketAddress address = new LocalSocketAddress("installd",
                    LocalSocketAddress.Namespace.RESERVED);

            mSocket.connect(address);

            mIn = mSocket.getInputStream();
            mOut = mSocket.getOutputStream();
        } catch (IOException ex) {
            disconnect();
            return false;
        }
        return true;
    }

    private boolean writeCommand(String cmdString) {
        final byte[] cmd = cmdString.getBytes();
        final int len = cmd.length;
        if ((len < 1) || (len > buf.length)) {
            return false;
        }

        buf[0] = (byte) (len & 0xff);
        buf[1] = (byte) ((len >> 8) & 0xff);
        try {
            mOut.write(buf, 0, 2);
            mOut.write(cmd, 0, len);
        } catch (IOException ex) {
            Slog.e(TAG, "write error");
            disconnect();
            return false;
        }
        return true;
    }
```


为了方便查看，连续贴了三个相关的方法。稍微显得有些长，没关系，我们一个个去分析。首先调用connect()方法通过Socket连接installd服务端。接着调用writeCommand()方法写入要发送的数据，这里参数cmdString是ping，所以len=4。最后通过readReply()方法，读取服务端返回的数据，转换成String形式返回。之后启动其它服务。


**启动ActivityManagerService**      

```
  private void startBootstrapServices() {
        mActivityManagerService = mSystemServiceManager.startService(
        ActivityManagerService.Lifecycle.class).getService();
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
        mActivityManagerService.setInstaller(installer);
    }
```


这里和启动Installer不同，startService()传入的是ActivityManagerService.Lifecycle.class，之后调用startService()返回值的getService()方法。跟进。

源码位置：frameworks/base/services/core/java/com/android/servicer/am/ActivityManagerService.java 

**ActivityManagerService$Lifecycle**


```
 public static final class Lifecycle extends SystemService {
        private final ActivityManagerService mService;

        public Lifecycle(Context context) {
            super(context);
            mService = new ActivityManagerService(context);
        }

        @Override
        public void onStart() {
            mService.start();
        }

        public ActivityManagerService getService() {
            return mService;
        }
    }
```

Lifecycle是ActivityManagerService的一个final类型的静态内部类。在分析SystemServiceManager#startService()说道：通过反射得到实例。最后添加进ArrayList<SystemService> mServices中以便管理。最后调用泛型Service中抽象父类的抽象方法onStart()的具体实现。所以这里首先会实例化ActivityManagerService对象，然后调用Lifecycle#onStart()。在Lifecycle#onStart()中调用ActivityManagerService#start()。最后getService()返回实例化过的ActivityManagerService对象。由于ActivityManagerService#start()相关代码太多，这里就不详细展开了。有时间单独写一篇博文解析。


**启动WindowManagerService** 

```
  private void startOtherServices() {
          wm = WindowManagerService.main(context, inputManager,
                    mFactoryTestMode != FactoryTest.FACTORY_TEST_LOW_LEVEL,
                    !mFirstBoot, mOnlyCore);
    }
```

这里和启动Installer、ActivityManagerService又有些不同，直接调用WindowManagerService#main()。真的是简单粗暴。跟进。


```
public static WindowManagerService main(final Context context,
            final InputManagerService im,
            final boolean haveInputMethods, final boolean showBootMsgs,
            final boolean onlyCore) {
        final WindowManagerService[] holder = new WindowManagerService[1];
        DisplayThread.getHandler().runWithScissors(new Runnable() {
            @Override
            public void run() {
                holder[0] = new WindowManagerService(context, im,
                        haveInputMethods, showBootMsgs, onlyCore);
            }
        }, 0);
        return holder[0];
    }
```


**参考链接** 

[Android SystemServer启动流程源码解析 - 一口仨馍 - 博客频道 - CSDN.NET](http://blog.csdn.net/qq_17250009/article/details/52143652)


