---
layout: post
title: Activity启动过程源码解析
date: 2016-8-15
categories: blog
tags: [源码解析]
description: 源码解析
---


### Activity框架和管理结构
 
Activity管理的核心是AcitivityManagerService，是一个独立的进程；
 
ActiveThread是每一个应用程序所在进程的主线程，循环的消息处理；
 
ActiveThread与AcitivityManagerService的通信是属于进程间通信，使用binder机制；

![](http://pic002.cnblogs.com/images/2012/328668/2012040716432354.jpg)



### Activity启动过程
 
以启动一个应用程序startActivity为例看一下代码执行的大概流程：

![](http://pic002.cnblogs.com/images/2012/328668/2012040716433927.jpg)

可将其分为6个过程：
 
1 使用代理模式启动到ActivityManagerService中执行；
 
2 创建ActivityRecord到mHistory记录中；
 
3 通过socket通信到Zgote相关类创建process；
 
4 通过ApplicatonThread与ActivityManagerService建立通信；
 
5 ActivityManagerService通知ActiveThread启动Activity的创建；
 
6 ActivityThread创建Activity加入到mActivities中并开始调度Activity执行；



### ActivityThread的创建过程

当我们第一次启动一个应用程序组件时，如果该应用没有任何组件存在，那么系统首先会创建该应用程序的ActivityThread

ActivityThread的创建过程是很复杂的，当程序进程第一次启动时，系统会在主线程中调用ActivityThread的main（）方法，从而完成ActivityThread的创建。

ActivityThread是什么？网上有很多说法：ActivityThread是应用程序的主线程，其实这种说法不准确。先来看看ActivityThread的官方说明：

```
/**
 * This manages the execution of the main thread in an
 * application process, scheduling and executing activities,
 * broadcasts, and other operations on it as the activity
 * manager requests.
 *
 * {@hide}
 */
public final class ActivityThread {
```


可以看到ActivityThread本身并不是一个线程。它负责管理主线程在进程中的执行，调度和执行Activity、BroadCast以及其他ActivityManagerService（以下简称AMS）请求的操作。具体来说，就是ActivityThread接收AMS的请求，负责应用程序的管理，主要是回调应用程序的生命周期函数。准确的说法是ActivityThread是一个运行在主线程的类，负责完成主线程的逻辑。

系统在创建进程并初始化主线程的时候，会调用ActivityThread的main()方法，代码如下：

```
/**
 *当启动一个组件时,如果该组件的应用程序进程没有创建，则创建该组件的进程，并为该进程创建
 *主线程，同时调用该方法
 */
public static void main(String[] args) {
    SamplingProfilerIntegration.start();

    // CloseGuard defaults to true and can be quite spammy.  We
    // disable it here, but selectively enable it later (via
    // StrictMode) on debug builds, but using DropBox, not logs.
    CloseGuard.setEnabled(false);

    Environment.initForCurrentUser();

    // Set the reporter for event logging in libcore
    EventLogger.setReporter(new EventLoggingReporter());

    Security.addProvider(new AndroidKeyStoreProvider());

    // Make sure TrustedCertificateStore looks in the right place for CA certificates
    final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
    TrustedCertificateStore.setDefaultUserDirectory(configDir);

    Process.setArgV0("<pre-initialized>");

    //为主线程创建Looper
    Looper.prepareMainLooper();

    //创建ActivityThread
    ActivityThread thread = new ActivityThread();

    //将ApplicationThread传递给AMS,通知AMS ActivityThread已经创建完毕
    thread.attach(false);

    //初始化主线程的Handler，以后我们可以看到，对应用组件的的生命周期的回调是
    //通过Handler发消息完成的
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    //加载AsycTask，并且创建其内部的Handler。
    AsyncTask.init();

    if (false) {
        Looper.myLooper().setMessageLogging(new
                LogPrinter(Log.DEBUG, "ActivityThread"));
    }

    //开启主线程的消息循环。
    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

可以看到main方法是该类的一个静态方法，主要完成以下操作：

- 为主线程创建Looper
- 创建ActivityThread
- 将ApplicationThread传递给AMS,通知AMS ActivityThread已经创建完毕
- 初始化主线程的Handler
- 加载AsycTask，并且创建其内部的Handler。
- 开启主线程的消息循环。


下面我们具体分析。

先看看 Looper.prepareMainLooper();

```
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```

不难发现，它在主线程中创建了一个Looper对象。

紧接着，ActivityThread thread = new ActivityThread();创建了ActivityThread,并且会导致 mH和ApplicationThread的初始化。

然后，看看thread.attach(false)：

```
private void  attach(boolean system) {
    sCurrentActivityThread = this;
    mSystemThread = system;
    if (!system) {
        ViewRootImpl.addFirstDrawHandler(new Runnable() {
            @Override
            public void run() {
                ensureJitEnabled();
            }
        });
        android.ddm.DdmHandleAppName.setAppName("<pre-initialized>",
                                                UserHandle.myUserId());
        RuntimeInit.setApplicationObject(mAppThread.asBinder());
        final IActivityManager mgr = ActivityManagerNative.getDefault();
        try {
            //将ApplicationThread传递给AMS
            mgr.attachApplication(mAppThread);
        } catch (RemoteException ex) {
            // Ignore
        }
        // Watch for getting close to heap limit.
        BinderInternal.addGcWatcher(new Runnable() {
            @Override public void run() {
                if (!mSomeActivitiesChanged) {
                    return;
                }
                Runtime runtime = Runtime.getRuntime();
                long dalvikMax = runtime.maxMemory();
                long dalvikUsed = runtime.totalMemory() - runtime.freeMemory();
                if (dalvikUsed > ((3*dalvikMax)/4)) {
                    if (DEBUG_MEMORY_TRIM) Slog.d(TAG, "Dalvik max=" + (dalvikMax/1024)
                            + " total=" + (runtime.totalMemory()/1024)
                            + " used=" + (dalvikUsed/1024));
                    mSomeActivitiesChanged = false;
                    try {
                        mgr.releaseSomeActivities(mAppThread);
                    } catch (RemoteException e) {
                    }
                }
            }
        });
    } else {
        // Don't set application object here -- if the system crashes,
        // we can't display an alert, we just want to die die die.
        android.ddm.DdmHandleAppName.setAppName("system_process",
                UserHandle.myUserId());
        try {
            mInstrumentation = new Instrumentation();
            ContextImpl context = ContextImpl.createAppContext(
                    this, getSystemContext().mPackageInfo);
            //创建Application对象
            mInitialApplication = context.mPackageInfo.makeApplication(true, null);
            //调用Application的onCreate()方法
            mInitialApplication.onCreate();
        } catch (Exception e) {
            throw new RuntimeException(
                    "Unable to instantiate Application():" + e.toString(), e);
        }
    }

    //添加配置Callback
    ViewRootImpl.addConfigCallback(new ComponentCallbacks2() {
        @Override
        public void onConfigurationChanged(Configuration newConfig) {
            synchronized (mResourcesManager) {
                // We need to apply this change to the resources
                // immediately, because upon returning the view
                // hierarchy will be informed about it.
                if (mResourcesManager.applyConfigurationToResourcesLocked(newConfig, null)) {
                    // This actually changed the resources!  Tell
                    // everyone about it.
                    if (mPendingConfiguration == null ||
                            mPendingConfiguration.isOtherSeqNewer(newConfig)) {
                        mPendingConfiguration = newConfig;
                        //
                        sendMessage(H.CONFIGURATION_CHANGED, newConfig);
                    }
                }
            }
        }
        @Override
        public void onLowMemory() {
        }
        @Override
        public void onTrimMemory(int level) {
        }
    });
}
```

可以看出 attach()主要完成以下几个步骤:

- 建立ApplicationThread和AMS之间的联系
- 创建Application对象并调用onCreat()方法，可以看到，Application对象，每个进程都会有且仅初始化一次。
- 添加ConfigurationCallback。

再看看AyncTask.init():

```
/** @hide Used to force static handler to be created. */
public static void init() {
    sHandler.getLooper();
}
```


这个方法时一个静态方法，作用是强制创建AsyncTask内部的Handler。由于异步任务是在线程池中执行任务，所以，必须使用Handler实现子线程和主线程之间的通讯。

最后 Looper.loop()方法为主线程开启消息循环。

至此，ActivityThread创建完毕，接下来，AMS就可以通过ApplicationThead来控制组件的生命周期。


**参考链接**       
[android 源码剖析之-------ActivityThread的创建过程 - 程序园](http://www.voidcn.com/blog/chenglei_56/article/p-5001873.html)


### Activity启动过程

ActivityThread用来管理四大组件。其中，ApplicationThread是供AMS调用ActivityThread远程方法的Binder类，AMS管理Activity正是通过ApplicationThread调用ActivityThread的相应方法完成的。我们知道通过Binder进行IPC的方法调用，是执行在服务端的Binder线程池中的，而不是主线程中，但是应用程序组件的生命周期方法执行在主线程，如何实现子线程中回调生命周期函数呢？答案是通过ActivityThread里的mH，它是一个Handler，由它来回调组件的生命周期回调。

下面，以Activity的创建为例，从源码的角度分析一下整个过程。其他的过程与启动过程类似。

当我们使用startActivity（）方法启动一个Activity时，会经过以下路径：

Activity.startActivity（） --->Activity.startActivityForResult(）

在Activity.startActivityForResult(）里，执行如下操作：

```
if (mParent == null) {

        //通过Instrumentation启动Activity
        Instrumentation.ActivityResult ar =
            mInstrumentation.execStartActivity(
                this, mMainThread.getApplicationThread(), mToken, this,
                intent, requestCode, options);
        if (ar != null) {
            mMainThread.sendActivityResult(
                mToken, mEmbeddedID, requestCode, ar.getResultCode(),
                ar.getResultData());
        }
        if (requestCode >= 0) {
            // If this start is requesting a result, we can avoid making
            // the activity visible until the result is received.  Setting
            // this code during onCreate(Bundle savedInstanceState) or onResume() will keep the
            // activity hidden during this time, to avoid flickering.
            // This can only be done when a result is requested because
            // that guarantees we will get information back when the
            // activity is finished, no matter what happens to it.
            mStartedActivity = true;
        }

        final View decor = mWindow != null ? mWindow.peekDecorView() : null;
        if (decor != null) {
            decor.cancelPendingInputEvents();
        }
        // TODO Consider clearing/flushing other event sources and events for child windows.
    } else {
        if (options != null) {
            mParent.startActivityFromChild(this, intent, requestCode, options);
        } else {
            // Note we want to go through this method for compatibility with
            // existing applications that may have overridden it.
            mParent.startActivityFromChild(this, intent, requestCode);
        }
    }
    if (options != null && !isTopOfTask()) {
        mActivityTransitionState.startExitOutTransition(this, options);
    }
}
```

可以看到，最终是通过Instrumentation.execStartActivity()来启动Activity的，再看看Instrumentation.execStartActivity()


```
public ActivityResult execStartActivity(
    Context who, IBinder contextThread, IBinder token, Fragment target,
    Intent intent, int requestCode, Bundle options) {
    IApplicationThread whoThread = (IApplicationThread) contextThread;
    if (mActivityMonitors != null) {
        synchronized (mSync) {
            final int N = mActivityMonitors.size();
            for (int i=0; i<N; i++) {
                final ActivityMonitor am = mActivityMonitors.get(i);
                if (am.match(who, null, intent)) {
                    am.mHits++;
                    if (am.isBlocking()) {
                        return requestCode >= 0 ? am.getResult() : null;
                    }
                    break;
                }
            }
        }
    }
    try {
        intent.migrateExtraStreamToClipData();
        intent.prepareToLeaveProcess();

        //Activity的真正启动过程是ActivityManagerNative完成的，本质上是调用AMS里面的方法启动
        int result = ActivityManagerNative.getDefault()
            .startActivity(whoThread, who.getBasePackageName(), intent,
                    intent.resolveTypeIfNeeded(who.getContentResolver()),
                    token, target != null ? target.mWho : null,
                    requestCode, 0, null, options);
        //检查Activity是否被成功创建
        checkStartActivityResult(result, intent);
    } catch (RemoteException e) {
    }
    return null;
}
```

可以看到Activity的真正启动过程是ActivityManagerNative完成的，而ActivityManagerService是ActivityManagerNative的实现类，通过对ServiceManager进行分析ActivityManagerNative.getDefault()得到的的确是AMS，这里就不做分析，后面会做详细的解析。

同时，我们注意到checkStartActivityResult(result, intent);这是用来对返回结果做检查：

```
public static void checkStartActivityResult(int res, Object intent) {
    if (res >= ActivityManager.START_SUCCESS) {
        return;
    }

    switch (res) {
        case ActivityManager.START_INTENT_NOT_RESOLVED:
        case ActivityManager.START_CLASS_NOT_FOUND:
            if (intent instanceof Intent && ((Intent)intent).getComponent() != null)
                throw new ActivityNotFoundException(
                        "Unable to find explicit activity class "
                        + ((Intent)intent).getComponent().toShortString()
                        + "; have you declared this activity in your AndroidManifest.xml?");
            throw new ActivityNotFoundException(
                    "No Activity found to handle " + intent);
        case ActivityManager.START_PERMISSION_DENIED:
            throw new SecurityException("Not allowed to start activity "
                    + intent);
        case ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT:
            throw new AndroidRuntimeException(
                    "FORWARD_RESULT_FLAG used while also requesting a result");
        case ActivityManager.START_NOT_ACTIVITY:
            throw new IllegalArgumentException(
                    "PendingIntent is not an activity");
        case ActivityManager.START_NOT_VOICE_COMPATIBLE:
            throw new SecurityException(
                    "Starting under voice control not allowed for: " + intent);
        default:
            throw new AndroidRuntimeException("Unknown error code "
                    + res + " when starting " + intent);
    }
}
```

那些异常是不是很熟悉。

接下来看看ActivityManagerService.startActivity()，会发现会走下面的调用逻辑：

startActivity（） ----

```
startActivityAsUser() --->ActivityStackSupervisor.startActivityMayWait()--->ActivityStackSupervisor.startActivityLocked()---> ActivityStackSupervisor.startActivityUncheckedLocked--->ActivityStack.resumeTopActivityLocked()--->ActivityStack.resumeTopActivityInnerLocked()--->ActivityStackSupervisor.startSpecificActivityLocked---- ActivityStackSupervisor.realStartActivityLocked
```

可以看到执行逻辑从ActivityManagerService到ActivityStackSupervisor，到ActivityStack,最后又到了ActivityStackSupervisor。

下面看看ActivityStackSupervisor.realStartActivityLocked：

这方法很长，看看下面这行代码：

```
app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
                System.identityHashCode(r), r.info, new Configuration(mService.mConfiguration),
                r.compat, r.task.voiceInteractor, app.repProcState, r.icicle, r.persistentState,
                results, newIntents, !andResume, mService.isNextTransitionForward(),
                profilerInfo);
```

可以发现，创建Activity最终是由app.thread来完成的，而app.thread是一个IApplicationThread，它是一个Binder接口，声明了很多与Activity和Service回调相关的方法。

IApplicationThread的实现类是ActivityThread中的内部类ApplicationThread，它继承自抽象类ApplicationThreadNative,而这个ApplicationThread对象正是实现AMS和ActivityThread实现IPC的媒介。

再看看ApplicationThread.scheduleLaunchActivity()是怎么处理的。

```
public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
            ActivityInfo info, Configuration curConfig, CompatibilityInfo compatInfo,
            IVoiceInteractor voiceInteractor, int procState, Bundle state,
            PersistableBundle persistentState, List<ResultInfo> pendingResults,
            List<Intent> pendingNewIntents, boolean notResumed, boolean isForward,
            ProfilerInfo profilerInfo) {

        updateProcessState(procState, false);

        ActivityClientRecord r = new ActivityClientRecord();

        r.token = token;
        r.ident = ident;
        r.intent = intent;
        r.voiceInteractor = voiceInteractor;
        r.activityInfo = info;
        r.compatInfo = compatInfo;
        r.state = state;
        r.persistentState = persistentState;

        r.pendingResults = pendingResults;
        r.pendingIntents = pendingNewIntents;

        r.startsNotResumed = notResumed;
        r.isForward = isForward;

        r.profilerInfo = profilerInfo;

        updatePendingConfiguration(curConfig);

        //最终通过Handler发消息，回调Activity的生命周期方法
        sendMessage(H.LAUNCH_ACTIVITY, r);
    }
```


可以看到主要完成两项任务，一个是利用AMS传递的信息创建一个ActivityClientRecord 对象，第二个是发送一个创建Activity的消息

最终我们发现是这样发消息的：

```
private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
    if (DEBUG_MESSAGES) Slog.v(
        TAG, "SCHEDULE " + what + " " + mH.codeToString(what)
        + ": " + arg1 + " / " + obj);
    Message msg = Message.obtain();
    msg.what = what;
    msg.obj = obj;
    msg.arg1 = arg1;
    msg.arg2 = arg2;
    if (async) {
        msg.setAsynchronous(true);
    }
    mH.sendMessage(msg);
}
```

它是通过mH发送了一条消息，mH其实是一个Handler。那么问题来了，为什么要这么做呢?因为ApplicationThread.scheduleLaunchActivity()是在应用程序的Binder'线程池中运行的，而Activity运行在主线程，因而要使用Handler实现工作线程到主线程的切换。

```
public void handleMessage(Message msg) {
        if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
        switch (msg.what) {
            case LAUNCH_ACTIVITY: {
                Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
                final ActivityClientRecord r = (ActivityClientRecord) msg.obj;

                r.packageInfo = getPackageInfoNoCheck(
                        r.activityInfo.applicationInfo, r.compatInfo);
                //回调
                handleLaunchActivity(r, null);
                Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
            } break;
```


在H里面，可以看到，启动Activity的操作是由handleLaunchActivity(r, null)完成的。


```
private void handleLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    // If we are getting ready to gc after going to the background, well
    // we are back active so skip it.
    unscheduleGcIdler();
    mSomeActivitiesChanged = true;

    if (r.profilerInfo != null) {
        mProfiler.setProfiler(r.profilerInfo);
        mProfiler.startProfiling();
    }

    // Make sure we are running with the most recent config.
    handleConfigurationChanged(null, null);

    if (localLOGV) Slog.v(
        TAG, "Handling launch of " + r);
    //回调，完成Activity的创建
    Activity a = performLaunchActivity(r, customIntent);

    if (a != null) {
        r.createdConfig = new Configuration(mConfiguration);
        Bundle oldState = r.state;
        //
        handleResumeActivity(r.token, false, r.isForward,
                !r.activity.mFinished && !r.startsNotResumed);

        if (!r.activity.mFinished && r.startsNotResumed) {
```

可以看到创建Activity最终是由performLaunchActivity(r, customIntent)完成的。

这个方法主要完成以下几个方面的内容：

从ActivityClientRecord中拿到Activity的组件信息


```
ActivityInfo aInfo = r.activityInfo; 
if (r.packageInfo == null) { 
r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo, Context.CONTEXT_INCLUDE_CODE); 
}

ComponentName component = r.intent.getComponent();
if (component == null) {
    component = r.intent.resolveActivity(
        mInitialApplication.getPackageManager());
    r.intent.setComponent(component);
}

if (r.activityInfo.targetActivity != null) {
    component = new ComponentName(r.activityInfo.packageName,
            r.activityInfo.targetActivity);
}
```

通过Instrumentation的newActivity方法创建Activity实例

其实Instrumentation是通过Java的反射机制创建Activity的。


```
Activity activity = null;
    try {
        java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
        activity = mInstrumentation.newActivity(
                cl, component.getClassName(), r.intent);
        StrictMode.incrementExpectedActivityCount(activity.getClass());
        r.intent.setExtrasClassLoader(cl);
        r.intent.prepareToEnterProcess();
        if (r.state != null) {
            r.state.setClassLoader(cl);
        }
    } catch (Exception e) {
        if (!mInstrumentation.onException(activity, e)) {
            throw new RuntimeException(
                "Unable to instantiate activity " + component
                + ": " + e.toString(), e);
        }
    }
```

创建Application对象，

Application app = r.packageInfo.makeApplication(false, mInstrumentation);

可以知道，在一个进程中，如果Application对象已经存在，是不会再创建Application对象的。

调用Activity的attach方法，主要是创建Window

通过Instrumentation的callmInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);调用Activity的oncreate()方法。

在onCreate()方法里，我们通过调用setContentView方法，为DecorView里的内容栏设置了布局
在onCreate（）方法完成之后，一个Activity就创建成功了。

继续看performLaunchActivity()这个方法：


```
if (!r.activity.mFinished) {
                activity.performStart();
                r.stopped = false;
     }
在调用onCreate()之后，会去执行 activity.performStart()，这里会调用：

mInstrumentation.callActivityOnStart(this);
而这个方法里会调用：

public void callActivityOnStart(Activity activity) {
    activity.onStart();
}
从而在执行onCreate之后，会执行onStart（）方法；

同样，在onStart之后，会判断需要不需要执行OnRestoreInstanceState方法：

//调用OnRestoreInstanceState方法
if (!r.activity.mFinished) {
    if (r.isPersistable()) {
        if (r.state != null || r.persistentState != null) {
            mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state,
                    r.persistentState);
        }
    } else if (r.state != null) {
        mInstrumentation.callActivityOnRestoreInstanceState(activity, r.state);
    }
}
```

在performLaunchActivity()方法返回后，这是，DecorView还处于不可见的状态，而且不能没有获得焦点。这是因为DecorView还没有关联WMS

在handleLaunchActivity()方法里，performLaunchActivity()方法执行完毕之后，接着执行：

```
if (a != null) {
        r.createdConfig = new Configuration(mConfiguration);
        Bundle oldState = r.state;
        //回调activity的onResume方法。
        handleResumeActivity(r.token, false, r.isForward,
                !r.activity.mFinished && !r.startsNotResumed);
在handleResumeActivity（）里首先执行这段代码：

ActivityClientRecord r = performResumeActivity(token, clearHide);
顾名思义，这最终导致onResume方法被调用

紧接着，会执行这段代码：

if (r.activity.mVisibleFromClient) {
                //将decorView添加到WMS中
                r.activity.makeVisible();
  }
我们在看看activity.makeVisible()

void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
```


可以看到，最终在这里DecorView被添加到WSM里，从而能够接受用户的输入，并响应窗口事件。

至此，Activity的启动过程分析完毕。

**参考链接**   
[Android源码剖析之-----Activity的启动过程 - 程序园](http://www.voidcn.com/blog/chenglei_56/article/p-5001874.html)


### 总结  

启动一个activity的过程

(1) 我们app中是使用了Activity的startActivity方法，具体调用的是ContextImpl的同名函数。

(2) ContextImpl中会调用Instrumentation的execStartActivity方法。

(3) Instrumentation通过aidl进行跨进程通信，最终调用AMS的startActivity方法。

(4) 在系统进程中AMS中先判断权限，然后通过调用ActivityStackSupervisor和ActivityStack进行一系列的交互用来确定Activity栈的使用方式。

(5) 通过ApplicationThread进行跨进程通信，转回到app进程。

(6) 通过ActivityThread中的H（一个handler）传递消息，最终调用Instrumentation来创建一个Activity。




