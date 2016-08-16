---
layout: post
title: Android启动过程源码解析
date: 2016-8-16
categories: blog
tags: [源码解析]
description: 源码解析
---


### Android系统启动过程

首先Android框架架构图：

![](http://pic002.cnblogs.com/images/2010/50949/2010123010192073.jpg)

Linux内核启动之后就到Android Init进程，进而启动Android相关的服务和应用。

启动的过程如下图所示：

![](http://hi.csdn.net/attachment/201005/14/0_1273850759wbAp.gif)

### Android启动过程

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


#### Init进程的启动

init进程，它是一个由内核启动的用户级进程。内核自行启动（已经被载入内存，开始运行，

并已初始化所有的设备驱动程序和数据结构等）之后，就通过启动一个用户级程序init的方式，完成引导进程。init始终是第一个进程。

启动过程就是代码init.c中main函数执行过程：system\core\init\init.c

在函数中执行了：文件夹建立，挂载，rc文件解析，属性设置，启动服务，执行动作，socket监听……

下面看两个重要的过程：rc文件解析和服务启动。


**rc文件解析**      

.rc文件是Android使用的初始化脚本文件 （System/Core/Init/readme.txt中有描述：

four broad classes of statements which are Actions, Commands, Services, and Options.）

其中Command 就是系统支持的一系列命令，如：export，hostname，mkdir，mount，等等，其中一部分是 linux 命令，

还有一些是 android 添加的，如：class_start <serviceclass>： 启动服务，class_stop <serviceclass>：关闭服务，等等。

其中Options是针对 Service 的选项的。

系统初始化要触发的动作和要启动的服务及其各自属性都在rc脚本文件中定义。 具体看一下启动脚本：\system\core\rootdir\init.rc

在解析rc脚本文件时，将相应的类型放入各自的List中.


** 服务启动**

文件解析完成之后将service放入到service_list中。

启动ServiceManager

ServiceManager用来管理系统中所有的binder service，不管是本地的c++实现的还是java语言实现的都需要

这个进程来统一管理，最主要的管理就是，注册添加服务，获取服务。所有的Service使用前都必须先在servicemanager中进行注册。


#### Zygote进程的启动         

Zygote进程是Android和Java世界的开创者。在Android系统中，所有的应用进程和SystemServer进程都是由Zygote进程fork而来。其重要性由此可见一斑。虽然Zygote进程相当于Android系统的根进程，但是事实上它也是由Linux系统的init进程启动的。各个进程的先后顺序为：

init进程 –-> Zygote进程 –> SystemServer进程 –>应用进程

其中Zygote进程由init进程启动，SystemServer进程和应用进程由Zygote进程启动。本文依据6.0源码，主要分析Zygote进程的启动流程。init进程在启动Zygote进程时会调用ZygoteInit#main()。以此为切入点，一步步分析。

源码位置：frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

**流程概览**

ZygoteInit#main();

```
public static void main(String argv[]) {
    try {
        // 设置DDMS可用
        RuntimeInit.enableDdms();
        // 初始化启动参数
        boolean startSystemServer = false;
        String socketName = "zygote";
        String abiList = null;
        for (int i = 1; i < argv.length; i++) {
            if ("start-system-server".equals(argv[i])) {
                startSystemServer = true;
            } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                abiList = argv[i].substring(ABI_LIST_ARG.length());
            } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                socketName = argv[i].substring(SOCKET_NAME_ARG.length());
            } else {
                throw new RuntimeException("Unknown command line argument: " + argv[i]);
            }
        }
        // 注册socket
        registerZygoteSocket(socketName);
        // 预加载各种资源
        preload();
        ...
        if (startSystemServer) {
            // 启动SystemServer进程
            startSystemServer(abiList, socketName);
        }
        // 监听socket,启动新的应用进程。--后文会讲
        runSelectLoop(abiList);
        closeServerSocket();
    } catch (MethodAndArgsCaller caller) {
        // 通过反射调用SystemServer#main()--后文会讲
        caller.run();
    } catch (RuntimeException ex) {
        closeServerSocket();
        throw ex;
    }
}
```

上面是个大概流程，下面会依据源码一步步解释。设置DDMS可用之后初始化各种参数，在此之后注册为Zygote进程注册Socket，预加载各种资源，但这些都不是重点！同学们，重点在于startSystemServer(abiList, socketName)(手敲黑板状)！下面简单贴下registerZygoteSocket(socketName)和preload()源码，不感兴趣的同学可直接略过下面两段代码。


**ZygoteInit#registerZygoteSocket()**       

```
private static void registerZygoteSocket(String socketName) {
    if (sServerSocket == null) {
        int fileDesc;
        final String fullSocketName = ANDROID_SOCKET_PREFIX + socketName;
        ...
        FileDescriptor fd = new FileDescriptor();
        fd.setInt$(fileDesc);
        // 不是使用IP和端口、而是使用fd创建socket
        sServerSocket = new LocalServerSocket(fd);
        ...
    }
}
```

**ZygoteInit#registerZygoteSocket()**      

```
static void preload() {
    preloadClasses(); // 加载所需的各种class文件
    preloadResources(); // 加载资源文件
    preloadOpenGL(); // 初始化OpenGL
    preloadSharedLibraries(); // 加载系统Libraries
    preloadTextResources(); //加载文字资源
    WebViewFactory.prepareWebViewInZygote(); // 初始化WebView
}
```

**启动SystemServer进程**    

```
private static boolean startSystemServer(String abiList, String socketName) throws MethodAndArgsCaller, RuntimeException {
    long capabilities = posixCapabilitiesAsBits(
        OsConstants.CAP_BLOCK_SUSPEND,
        OsConstants.CAP_KILL,
        ...
    );
    /* Hardcoded command line to start the system server */
    String args[] = {
        "--setuid=1000",
        "--setgid=1000",
        "--setgroups=1001,1002,1003,1004,1005,1006,1007,1008,1009,1010,1018,1021,1032,3001,3002,3003,3006,3007",
        "--capabilities=" + capabilities + "," + capabilities,
        "--nice-name=system_server",
        "--runtime-args",
        "com.android.server.SystemServer",
    };
    ZygoteConnection.Arguments parsedArgs = null;

    int pid;

    try {
        parsedArgs = new ZygoteConnection.Arguments(args);
        // 打开系统调试属性
        ZygoteConnection.applyDebuggerSystemProperty(parsedArgs);
        ZygoteConnection.applyInvokeWithSystemProperty(parsedArgs);

        // 请求fork SystemServer进程
        pid = Zygote.forkSystemServer(
                parsedArgs.uid, parsedArgs.gid,
                parsedArgs.gids,
                parsedArgs.debugFlags,
                null,
                parsedArgs.permittedCapabilities,
                parsedArgs.effectiveCapabilities);
    } catch (IllegalArgumentException ex) {
        throw new RuntimeException(ex);
    }

    // pid为0表示子进程，即SystemServer进程，从此SystemServer进程与Zygote进程分道扬镳
    if (pid == 0) {
        if (hasSecondZygote(abiList)) {
            waitForSecondaryZygote(socketName);
        }

        handleSystemServerProcess(parsedArgs);
    }

    return true;
}
```

前面一大段都在构造参数，直接进到try中的代码块。首先根据args数组构造了一个ZygoteConnection.Arguments，然后根据parsedArgs对象的各种参数调用Zygote#forkSyatemServer()方法fork出第一个子进程，也就是SystemServer进程。最后通过执行handleSystemServerProcess反射调用SystemServer#main()。可以看到，这段代码最主要的作用就是fork出SystemServer进程。这里还看不出反射调用的具体细节，下文会一一分析。

首先看下构造ZygoteConnection.Arguments对象时，具体都做了哪些工作，尤其关注Zygote#forkSystemServer()中几个参数的值。

源码位置：frameworks/base/core/java/com/android/internal/os/ZygoteConnection$Arguments.java

```
 Arguments(String args[]) throws IllegalArgumentException {
        parseArgs(args);
    }

    private void parseArgs(String args[]) throws IllegalArgumentException {
        int curArg = 0;
        boolean seenRuntimeArgs = false;

        for ( /* curArg */ ; curArg < args.length; curArg++) {
            String arg = args[curArg];

            if (arg.equals("--")) {
                curArg++;
                break;
            } else if (arg.startsWith("--setuid=")) {
                if (uidSpecified) {
                    throw new IllegalArgumentException("Duplicate arg specified");
                }
                uidSpecified = true;
                uid = Integer.parseInt(arg.substring(arg.indexOf('=') + 1));
            } else if (arg.startsWith("--setgid=")) {
                if (gidSpecified) {
                gidSpecified = true;
                gid = Integer.parseInt(arg.substring(arg.indexOf('=') + 1));
            } else if (arg.startsWith("--target-sdk-version=")) {
                targetSdkVersionSpecified = true;
                targetSdkVersion = Integer.parseInt(arg.substring(arg.indexOf('=') + 1));
            } 
            ...
              else if (arg.equals("--runtime-args")) {
                seenRuntimeArgs = true;
            } else if (arg.startsWith("--capabilities=")) {
                capabilitiesSpecified = true;
                String capString = arg.substring(arg.indexOf('=')+1);
                String[] capStrings = capString.split(",", 2);
                if (capStrings.length == 1) {
                    effectiveCapabilities = Long.decode(capStrings[0]);
                    permittedCapabilities = effectiveCapabilities;
                } else {
                    permittedCapabilities = Long.decode(capStrings[0]);
                    effectiveCapabilities = Long.decode(capStrings[1]);
                }
            } else if (arg.startsWith("--setgroups=")) {
                String[] params = arg.substring(arg.indexOf('=') + 1).split(",");
                gids = new int[params.length];
                for (int i = params.length - 1; i >= 0 ; i--) {
                    gids[i] = Integer.parseInt(params[i]);
                }
            } else if (arg.startsWith("--nice-name=")) {
                niceName = arg.substring(arg.indexOf('=') + 1);
            } else {
                break;
            }
        }

        // 保存没有被解析的参数
        remainingArgs = new String[args.length - curArg];
        System.arraycopy(args, curArg, remainingArgs, 0, remainingArgs.length);    
    }
```

对比传入的args数组，可以发现：parsedArgs.uid=1000、parsedArgs.gid=1000、parsedArgs.gids={"1001","1002",..."3007"}、parsedArgs.gid=1000、parsedArgs.niceName=system_server、parsedArgs.seenRuntimeArgs=true。如果中途结束，保存未解析的参数至remainingArgs数组。 
获得Arguments对象之后，就开始请求创建SystemServer进程。

源码位置：frameworks/base/core/java/com/android/internal/os/ZygoteInit.java

**ZygoteInit#handleSystemServerProcess()**

```
private static void handleSystemServerProcess(
        ZygoteConnection.Arguments parsedArgs)
        throws ZygoteInit.MethodAndArgsCaller {

    closeServerSocket();

    if (parsedArgs.niceName != null) {
        Process.setArgV0(parsedArgs.niceName);
    }
    ...
    // 默认为null
    if (parsedArgs.invokeWith != null) {
        ...
    } else {
        ...
        RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl);
    }

    /* should never reach here */
}
```

由Zygote创建的子进程默认拥有Zygote进程的Socket对象，而子进程又用不上，所以先调用closeServerSocket()关闭它。上一段参数解析时写道：parsedArgs.niceName=system_server，在这里调用Process.setArgV0()设置进程名为：system_server。由于parsedArgs.invokeWith属性默认为null，最后调用RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, cl)来进一步启动SystemServer，这里的参数parsedArgs.remainingArgs就是上文中保存没有被解析对象的数组。

源码位置：frameworks/base/core/java/com/android/internal/os/RuntimeInit.java

**RuntimeInit#zygoteInit()**

```
public static final void zygoteInit(int targetSdkVersion, String[] argv, ClassLoader classLoader) throws ZygoteInit.MethodAndArgsCaller {
    // 重定向Log输出
    redirectLogStreams();
    //初始化运行环境
    commonInit();
    //启动Binder线程池
    nativeZygoteInit();
    //调用程序入口函数
    applicationInit(targetSdkVersion, argv, classLoader);
}
```


**RuntimeInit#applicationInit()** 


```
private static void applicationInit(int targetSdkVersion, String[] argv, ClassLoader classLoader)
        throws ZygoteInit.MethodAndArgsCaller {
    // 初始化虚拟机环境
    VMRuntime.getRuntime().setTargetHeapUtilization(0.75f);
    VMRuntime.getRuntime().setTargetSdkVersion(targetSdkVersion);
    final Arguments args;
    try {
        args = new Arguments(argv);
    } catch (IllegalArgumentException ex) {
        return;
    }

    // Remaining arguments are passed to the start class's static main
    invokeStaticMain(args.startClass, args.startArgs, classLoader);
}
```


**RuntimeInit#invokeStaticMain()** 

```
private static void invokeStaticMain(String className, String[] argv, ClassLoader classLoader)
        throws ZygoteInit.MethodAndArgsCaller {
    Class<?> cl;

    try {
        cl = Class.forName(className, true, classLoader);
    } catch (ClassNotFoundException ex) {
        throw new RuntimeException("Missing class when invoking static main " +    className, ex);
    }

    Method m;
    try {
        // 获取main方法
        m = cl.getMethod("main", new Class[] { String[].class });
    } catch (NoSuchMethodException ex) {
        throw new RuntimeException("Missing static main on " + className, ex);
    } catch (SecurityException ex) {
        throw new RuntimeException("Problem getting static main on " + className, ex);
    }
    // 判断修饰符
    int modifiers = m.getModifiers();
    if (! (Modifier.isStatic(modifiers) && Modifier.isPublic(modifiers))) {
        throw new RuntimeException("Main method is not public and static on " + className);
    }

    throw new ZygoteInit.MethodAndArgsCaller(m, argv);
}
```

这里传入的className就是com.android.server.SystemServer，然后获取main方法，接着判断修饰符，必须是static而且必须是public类型。最有意思的莫过于做完这一切之后，抛出了个MethodAndArgsCaller异常。辛苦辛苦各种初始化，各种变着法儿的调用，最后你居然给我抛个异常！！先别急，这个异常在Zygote#main()方法中捕获。这么做的作用是清除应用程序进程创建过程的调用栈。

```
public static void main(String argv[]) {
    try {
        ...
        startSystemServer(abiList, socketName);
        ...
    } catch (MethodAndArgsCaller caller) {
        caller.run();
    }
}
```


跟进MethodAndArgsCaller#run()，感觉要出大事情！！


```
public void run() {
        try {
            mMethod.invoke(null, new Object[] { mArgs });
        } catch (IllegalAccessException ex) {
            throw new RuntimeException(ex);
        } catch (InvocationTargetException ex) {
            Throwable cause = ex.getCause();
            if (cause instanceof RuntimeException) {
                throw (RuntimeException) cause;
            } else if (cause instanceof Error) {
                throw (Error) cause;
            }
            throw new RuntimeException(ex);
        }
    }
```

可以看到在这里通过反射调用了com.android.server.SystemServer#main(String[] args)。至此，Zygote进程fork出SystemServer进程，并成功调用SystemServer#main()。

现在SystemServer进程也创建了，main方法也调用了。Zygote进程的使命就此完结了吗？上文我们说道：所有的应用进程和SystemServer进程都是由Zygote进程fork而来。现在有关SystemServer进程的已经告一段落，那有关应用进程呢？

让我们再次回到ZygoteInit#main()

```
public static void main(String argv[]) {
      ...
      startSystemServer(abiList, socketName);
      runSelectLoop(abiList);
      closeServerSocket();
}
```

main方法中前面所有的代码好像都和应用进程没有关系，最后一行又是关闭socket，看来和应用进程相关的设置都在runSelectLoop()中，跟进。


#### 监听Socket,启动应用进程

ZygoteInit#runSelectLoop()、ZygoteInit#acceptCommandPeer()

```
private static void runSelectLoop(String abiList) throws MethodAndArgsCaller {
    ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
    ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();
    FileDescriptor[] fdArray = new FileDescriptor[4];
    fds.add(sServerSocket.getFileDescriptor());
    peers.add(null);
    int loopCount = GC_LOOP_COUNT;
    while (true) {
        int index;
        if (loopCount <= 0) {
            gc();
            loopCount = GC_LOOP_COUNT;
        } else {
            loopCount--;
        }

        try {
            fdArray = fds.toArray(fdArray);
            index = selectReadable(fdArray);
        } catch (IOException ex) {
            throw new RuntimeException("Error in select()", ex);
        }

        if (index < 0) {
            throw new RuntimeException("Error in select()");
        } else if (index == 0) {
            ZygoteConnection newPeer = acceptCommandPeer(abiList);
            peers.add(newPeer);
            fds.add(newPeer.getFileDescriptor());
        } else {
            boolean done;
            done = peers.get(index).runOnce();

            if (done) {
                peers.remove(index);
                fds.remove(index);
            }
        }
    }
}

private static ZygoteConnection acceptCommandPeer(String abiList) {
    try {
        return new ZygoteConnection(sServerSocket.accept(), abiList);
    } catch (IOException ex) {
        throw new RuntimeException("IOException during accept()", ex);
    }
}
```

这里有个死循环，一直监听socket，然后调用ZygoteConnection#runOnce()，从函数名runOnce上感觉真相就要呼之欲出了，跟进。

源码位置：frameworks/base/core/java/com/android/internal/os/ZygoteConnection.java

**ZygoteConnection#runOnce()**


```
boolean runOnce() throws ZygoteInit.MethodAndArgsCaller {
    String args[];
    args = readArgumentList();
    parsedArgs = new Arguments(args);
     try {
        ...
        pid = Zygote.forkAndSpecialize(parsedArgs.uid, parsedArgs.gid, parsedArgs.gids,
                parsedArgs.debugFlags, rlimits, parsedArgs.mountExternal, parsedArgs.seInfo,
                parsedArgs.niceName, fdsToClose, parsedArgs.instructionSet,
                parsedArgs.appDataDir);
    } catch (ErrnoException ex) {
        logAndPrintError(newStderr, "Exception creating pipe", ex);
    } catch (IllegalArgumentException ex) {
        logAndPrintError(newStderr, "Invalid zygote arguments", ex);
    } catch (ZygoteSecurityException ex) {
        logAndPrintError(newStderr,
                "Zygote security policy prevents request: ", ex);
    }

    try {
        if (pid == 0) {
            // in child
            IoUtils.closeQuietly(serverPipeFd);
            serverPipeFd = null;
            handleChildProc(parsedArgs, descriptors, childPipeFd, newStderr);

            // should never get here, the child is expected to either
            // throw ZygoteInit.MethodAndArgsCaller or exec().
            return true;
        } else {
            // in parent...pid of < 0 means failure
            IoUtils.closeQuietly(childPipeFd);
            childPipeFd = null;
            return handleParentProc(pid, descriptors, serverPipeFd, parsedArgs);
        }
    } finally {
        IoUtils.closeQuietly(childPipeFd);
        IoUtils.closeQuietly(serverPipeFd);
    }
}
```

和启动SystemServer进程类似。这里调用Zygote#forkAndSpecialize()创建应用进程，而参数parsedArgs是通过socket一行行读出来的。详见

**ZygoteConnection#readArgumentList()**

```
private String[] readArgumentList() throws IOException {
    int argc;
    try {
        String s = mSocketReader.readLine();
        if (s == null) {
            return null;
        }
        argc = Integer.parseInt(s);
    } catch (NumberFormatException ex) {
        throw new IOException("invalid wire format");
    }
    if (argc > MAX_ZYGOTE_ARGC) {
        throw new IOException("max arg count exceeded");
    }
    String[] result = new String[argc];
    for (int i = 0; i < argc; i++) {
        result[i] = mSocketReader.readLine();
        if (result[i] == null) {
            // We got an unexpected EOF.
            throw new IOException("truncated request");
        }
    }
    return result;
}
```

因为还没有看发送Socket消息的源码，这里斗胆猜测：应该是uid、gid、niceName等参数。

通过Socket读取完各种参数之后，调用ZygoteConnection#handleChildProc()，创建完应用程序进程之后就该调用应用程序的入口方法了。跟进。

```
private void handleChildProc(Arguments parsedArgs, FileDescriptor[] descriptors, FileDescriptor pipeFd, PrintStream newStderr)
        throws ZygoteInit.MethodAndArgsCaller {
    // 关闭从Zygote进程复制过来的Socket连接  
    closeSocket();
    ZygoteInit.closeServerSocket();
    if (parsedArgs.niceName != null) {
        Process.setArgV0(parsedArgs.niceName);
    }
    ...
    RuntimeInit.zygoteInit(parsedArgs.targetSdkVersion, parsedArgs.remainingArgs, null /* classLoader */);
}
```

最后调用RuntimeInit#zygoteInit()，后面的就和SystemServer启动流程类似。感兴趣的同学自行查看。


**总结一下Zygote启动流程：**

- 初始化DDMS

- 注册Zygote进程的Socket

- 加载class、resource、OpenGL、WebView等各种资源

- fork出SystemServer进程

- 启动SystemServer进程

- 调用runSelectLoop()一直监听Socket信息

- 收到创建应用程序Socket消息，调用ZygoteConnection#runOnce()。在runOnce（）中调用Zygote#forkAndSpecialize()创建应用进程

- 启动应用进程

![](http://pic002.cnblogs.com/images/2012/328668/2012090311451831.jpg)

参考链接：

[Android Zygote启动流程源码解析 - 一口仨馍 - 博客频道 - CSDN.NET](http://blog.csdn.net/qq_17250009/article/details/52061171)


**最后**

到这里系统ApplicationFramework层的XxxServiceManager准备就绪，可以开始跑上层应用了，我们的第一个上层应用HomeLauncher。

HomeActivity又是如何启动的呢？

Activity的启动必然和ActivityManagerService有关，我们需要去看看

ActivityManagerService.systemReady( )中都干了些什么。



**Home界面启动**


```
public void systemReady(final Runnable goingCallback) {
　　　　……
　　　　//ready callback
       if (goingCallback != null)
              goingCallback.run();

       synchronized (this) {
              // Start up initial activity.
              // ActivityStack mMainStack;
              mMainStack.resumeTopActivityLocked(null);
       }
……

}

final boolean resumeTopActivityLocked(ActivityRecord prev) {
　　// Find the first activity that is not finishing.
　　ActivityRecord next = topRunningActivityLocked(null);
　　if (next == null) {
　　　　// There are no more activities!  Let's just start up the
　　　　// Launcher...
　　　　if (mMainStack) {
　　　　　　//ActivityManagerService mService;
　　　　　　return mService.startHomeActivityLocked();
　　　　}
　　}
　　……
}
```

![](http://pic002.cnblogs.com/images/2012/328668/2012082816045188.jpg)


**参考链接**         

[Android系统启动过程 - __Shadow - 博客园](http://www.cnblogs.com/bastard/archive/2012/08/28/2660389.html)