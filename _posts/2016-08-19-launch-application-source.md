---
layout: post
title: 应用启动过程源码解析
date: 2016-8-19
categories: blog
tags: [源码解析]
description: 应用启动
---


- 点击桌面Launcher图标后做了哪些工作？
- 应用程序什么时候被创建的？
- Application和MainActivity的onCreate()方法什么时候被调用的？


**概述**

在Android系统中，启动四大组件中的任何一个都可以启动应用程序。但绝大部分时候我们是通过点击Launcher图标启动应用程序。本文依据Android6.0源码，从点击Launcher图标，直至解析到MainActivity#OnCreate()被调用。


### Launcher简析

Launcher也是个应用程序，不过是个特殊的应用。俗称“桌面”。通过PackageManagerService查询所有已安装的应用程序，并保存相应的图标、应用名称、包名和第一个要启动的类名等。

源码位置：frameworks/base/core/java/android/app/SystemServer.java

**LauncherActivity#onListItemClick()**

```
public abstract class LauncherActivity extends ListActivity {
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        Intent intent = intentForPosition(position);
        startActivity(intent);
    }
}
```

可以看到LauncherActivity继承自ListActivity，而ListActivity继承自Activity。在LauncherActivity#onListItemClick()方法中首先通过intentForPosition()获取了一个Intent，然后调用startActivity(intent)启动一个Activity。由于ListActivity并没有重写Activity#startActivity()方法，所以这里会直接调用Activity#startActivity()。在此之前先看下intentForPosition()是怎样构造Intent。


```
protected Intent intentForPosition(int position) {
        ActivityAdapter adapter = (ActivityAdapter) mAdapter;
        return adapter.intentForPosition(position);
    }

    private class ActivityAdapter extends BaseAdapter implements Filterable {
        public Intent intentForPosition(int position) {
            if (mActivitiesList == null) {
                return null;
            }
            Intent intent = new Intent(mIntent);
            ListItem item = mActivitiesList.get(position);
            intent.setClassName(item.packageName, item.className);
            if (item.extras != null) {
                intent.putExtras(item.extras);
            }
            return intent;
        }
    }
 ```

Intent中只是指定了即将启动的Activity。

源码位置：frameworks/base/core/java/android/app/Activity.java

**Activity#startActivity()**

```
public void startActivity(Intent intent) {
        this.startActivity(intent, null);
    }

    public void startActivity(Intent intent, @Nullable Bundle options) {
        if (options != null) {
            startActivityForResult(intent, -1, options);
        } else {
            startActivityForResult(intent, -1);
        }
    }

    /** @param requestCode If >= 0, this code will be returned inonActivityResult() when the activity exits. */
    public void startActivityForResult(Intent intent, int requestCode) {
        startActivityForResult(intent, requestCode, null);
    }

    public void startActivityForResult(Intent intent, int requestCode, @Nullable Bundle options) {
            Instrumentation.ActivityResult ar = mInstrumentation.execStartActivity(
                    this, mMainThread.getApplicationThread(), mToken, this,
                    intent, requestCode, options);
           ...
        }
    }
```


在调用的过程中可以观察到，无论调用StartActivity()还是startActivityForResult()，最终调用的都是startActivityForResult()。而且注释说明只有requestCode >= 0时onActivityResult()才会被调用。经过一系列的调用，最终调用了Instrumentation#execStartActivity()。跟进。

源码位置：frameworks/base/core/java/android/app/Instrumentation.java

**Instrumentation#execStartActivity()**

```
public ActivityResult execStartActivity(
        Context who, IBinder contextThread, IBinder token, Activity target, 
        Intent intent, int requestCode, Bundle options) {
            ...
            int result = ActivityManagerNative.getDefault()
                .startActivity(whoThread, who.getBasePackageName(), intent,
                        intent.resolveTypeIfNeeded(who.getContentResolver()),
                        token, target != null ? target.mEmbeddedID : null,
                        requestCode, 0, null, options);
            ...
    }
```

这里ActivityManagerNative.getDefault()涉及到Binder进程间通信机制。


### 调用过程解析

前方高能，可能会导致身体感到严重不适。请做好心理准备！

源码位置：frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java

**ActivityManagerService#startActivity()**

```
 @Override
    public final int startActivity(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle options) {
        return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
            resultWho, requestCode, startFlags, profilerInfo, options,
            UserHandle.getCallingUserId());
    }

    @Override
    public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
            Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
            int startFlags, ProfilerInfo profilerInfo, Bundle options, int userId) {
        // 验证CallingUid是否合法
        enforceNotIsolatedCaller("startActivity");
        // 检查限权
        userId = handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(), userId,
                false, ALLOW_FULL_ONLY, "startActivity", null);
        return mStackSupervisor.startActivityMayWait(caller, -1, callingPackage, intent,
                resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
                profilerInfo, null, null, options, false, userId, null, null);
    }
```

简单验证后开始调用StackSupervisor#startActivityMayWait()，跟进。 

源码位置：frameworks/base/services/core/java/com/android/server/am/StackSupervisor.java

**StackSupervisor#startActivityMayWait()、StackSupervisor#resolveActivity()**

```
 final int startActivityMayWait(IApplicationThread caller, int callingUid,
                                   String callingPackage, Intent intent, String resolvedType,
                                   IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
                                   IBinder resultTo, String resultWho, int requestCode, int startFlags,
                                   ProfilerInfo profilerInfo, WaitResult outResult, Configuration config,
                                   Bundle options, boolean ignoreTargetSecurity, int userId,
                                   IActivityContainer iContainer, TaskRecord inTask) {
          ...
         // 解析intent信息                         
         ActivityInfo aInfo = resolveActivity(intent, resolvedType, startFlags, profilerInfo, userId);
         // 验证pid uid(省略)
         ...
        int res = startActivityLocked(caller, intent, resolvedType, aInfo,
                voiceSession, voiceInteractor, resultTo, resultWho,
                requestCode, callingPid, callingUid, callingPackage,
                realCallingPid, realCallingUid, startFlags, options, ignoreTargetSecurity,
                componentSpecified, null, container, inTask);

        return res;                          
    }

    ActivityInfo resolveActivity(Intent intent, String resolvedType, int startFlags,
                                 ProfilerInfo profilerInfo, int userId) {
        // Collect information about the target of the Intent.
        ActivityInfo aInfo;
        try {
            ResolveInfo rInfo = AppGlobals.getPackageManager().resolveIntent(
                            intent, resolvedType,
                            PackageManager.MATCH_DEFAULT_ONLY 
                            | ActivityManagerService.STOCK_PM_FLAGS, userId);
            aInfo = rInfo != null ? rInfo.activityInfo : null;
        } catch (RemoteException e) {
            aInfo = null;
        }

        if (aInfo != null) {
            intent.setComponent(new ComponentName(aInfo.applicationInfo.packageName, aInfo.name));
            ...
        }
        return aInfo;
    }
```

在调用resolveActivity()之后会读取配置文件中包名和要启动Activity完整的路径名，并分别赋值给aInfo.applicationInfo.packageName和aInfo.name。跟进。

**StackSupervisor#startActivityLocked()**

```
 final int startActivityLocked(IApplicationThread caller,
                                  Intent intent, String resolvedType, ActivityInfo aInfo,
                                  IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
                                  IBinder resultTo, String resultWho, int requestCode,
                                  int callingPid, int callingUid, String callingPackage,
                                  int realCallingPid, int realCallingUid, int startFlags, Bundle options,
                                  boolean ignoreTargetSecurity, boolean componentSpecified, ActivityRecord[] outActivity,
                                  ActivityContainer container, TaskRecord inTask) {
        ProcessRecord callerApp = null;
        if (caller != null) {
            callerApp = mService.getRecordForAppLocked(caller);
            if (callerApp != null) {
                callingPid = callerApp.pid;
                callingUid = callerApp.info.uid;
            } else {
                err = ActivityManager.START_PERMISSION_DENIED;
            }
        }
        ...
        ActivityRecord r = new ActivityRecord(mService, callerApp, callingUid, callingPackage,
                intent, resolvedType, aInfo, mService.mConfiguration, resultRecord, resultWho,
                requestCode, componentSpecified, voiceSession != null, this, container, options);
        // 检查intent中的flag信息
        ...
        // 检查限权
        ...
        ...
        err = startActivityUncheckedLocked(r, sourceRecord, voiceSession, voiceInteractor,
                startFlags, true, options, inTask);
        return err;

    }
```

这个方法近300行代码，我们只看关键的部分就好了。首先获取调用者的pid、uid等信息，然后保存在callerApp中。目标Activity的信息保存在 r 中。跟进。

**StackSupervisor#startActivityUncheckedLocked()**


```
final int startActivityUncheckedLocked(final ActivityRecord r, ActivityRecord sourceRecord,
        IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor, int startFlags,
        boolean doResume, Bundle options, TaskRecord inTask) {
        ...
        int launchFlags = intent.getFlags();
        // 对intent的flag各种判断
        ...
        ActivityRecord notTop = (launchFlags & Intent.FLAG_ACTIVITY_PREVIOUS_IS_TOP) != 0 ? r : null;
        ...
                ActivityStack sourceStack;
        if (sourceRecord != null) {
            if (sourceRecord.finishing) {
                if ((launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
                    Slog.w(TAG, "startActivity called from finishing " + sourceRecord
                            + "; forcing " + "Intent.FLAG_ACTIVITY_NEW_TASK for: " + intent);
                    launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
                    newTaskInfo = sourceRecord.info;
                    newTaskIntent = sourceRecord.task.intent;
                }
                sourceRecord = null;
                sourceStack = null;
            } else {
                sourceStack = sourceRecord.task.stack;
            }
        } else {
            sourceStack = null;
        }
        ...
        if (((launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0 &&
                (launchFlags & Intent.FLAG_ACTIVITY_MULTIPLE_TASK) == 0)
                || launchSingleInstance || launchSingleTask) {
            if (inTask == null && r.resultTo == null) {
                // See if there is a task to bring to the front.  If this is
                // a SINGLE_INSTANCE activity, there can be one and only one
                // instance of it in the history, and it is always in its own
                // unique task, so we do a special search.
                ActivityRecord intentActivity = !launchSingleInstance ?
                        findTaskLocked(r) : findActivityLocked(intent, r.info);
        ...
        }
        ...
        // Should this be considered a new task?
        if (r.resultTo == null && inTask == null && !addingToTask
                && (launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
            newTask = true;
            targetStack = computeStackFocus(r, newTask);
            targetStack.moveToFront("startingNewTask");

            if (reuseTask == null) {
                // 设任务栈
                r.setTask(targetStack.createTaskRecord(getNextTaskId(),
                        newTaskInfo != null ? newTaskInfo : r.info,
                        newTaskIntent != null ? newTaskIntent : intent,
                        voiceSession, voiceInteractor, !launchTaskBehind /* toTop */),
                        taskToAffiliate);
            } 
            ...
        }
        ...
        targetStack.startActivityLocked(r, newTask, doResume, keepCurTransition, options);
        return ActivityManager.START_SUCCESS;
    }
```


这个方法600+行，而且好多逻辑判断，看的几近崩溃。不过还好，关键的代码已经被过滤出来了～首先确定几个变量。前面都没有设置过intent的flag，所以launchFlags = 0x00000000，但是在执行launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK之后。launchFlags = Intent.FLAG_ACTIVITY_NEW_TASK。notTop = null。由于没有要接收返回信息的Activity，所以r.resultTo == null。由于没有设置Activity的启动模式为SingleInstance，所以launchSingleInstance = false，即会执行findTaskLocked(r)。这个方法的作用是查找历史所有stack中有没有目标Activity，很显然返回值为null，所以下面很大一段判断逻辑都不会执行。大约200行，爽～。既然没有相应的任务栈，肯定要创建的。在上文中创建任务栈对应的方法为targetStack.createTaskRecord()。最后执行ActivityStack#startActivityLocked()，跟进。

源码位置：frameworks/base/services/core/java/com/android/server/am/ActivityStack.java

**ActivityStack#startActivityLocked()**

```
final void startActivityLocked(ActivityRecord r, boolean newTask,
            boolean doResume, boolean keepCurTransition, Bundle options) {
        ...
        // newTask = true
        if(!newTask){
            ...
        }
        task = r.task;
        // 将ActivityRecord置于栈顶
        task.addActivityToTop(r);
        // 将任务栈置于前台
        task.setFrontOfTask();
        ...
        if (doResume) {
            mStackSupervisor.resumeTopActivitiesLocked(this, r, options);
        }
    }
```

跟进。

源码位置：frameworks/base/services/core/java/com/android/server/am/StackSupervisor.java

**StackSupervisor#resumeTopActivitiesLocked()**

```
 boolean resumeTopActivitiesLocked(ActivityStack targetStack, ActivityRecord target,
                                      Bundle targetOptions) {
        if (targetStack == null) {
            targetStack = mFocusedStack;
        }
        // Do targetStack first.
        boolean result = false;
        if (isFrontStack(targetStack)) {
            result = targetStack.resumeTopActivityLocked(target, targetOptions);
        }

        for (int displayNdx = mActivityDisplays.size() - 1; displayNdx >= 0; --displayNdx) {
            final ArrayList<ActivityStack> stacks = mActivityDisplays.valueAt(displayNdx).mStacks;
            for (int stackNdx = stacks.size() - 1; stackNdx >= 0; --stackNdx) {
                final ActivityStack stack = stacks.get(stackNdx);
                if (stack == targetStack) {
                    // Already started above.
                    continue;
                }
                if (isFrontStack(stack)) {
                    stack.resumeTopActivityLocked(null);
                }
            }
        }
        return result;
    }
```

这个方法的作用是将Launcher应用程序所在的任务栈执行resumeTopActivityLocked()方法。跟进。

源码位置：frameworks/base/services/core/java/com/android/server/am/ActivityStack.java

**ActivityStack#resumeTopActivityLocked()**

```
final boolean resumeTopActivityLocked(ActivityRecord prev, Bundle options) {
        if (mStackSupervisor.inResumeTopActivity) {
            // Don't even start recursing.
            return false;
        }

        boolean result = false;
        try {
            // Protect against recursion.
            mStackSupervisor.inResumeTopActivity = true;
            if (mService.mLockScreenShown == ActivityManagerService.LOCK_SCREEN_LEAVING) {
                mService.mLockScreenShown = ActivityManagerService.LOCK_SCREEN_HIDDEN;
                mService.updateSleepIfNeededLocked();
            }
            result = resumeTopActivityInnerLocked(prev, options);
        } finally {
            mStackSupervisor.inResumeTopActivity = false;
        }
        return result;
    }
```

跟进。

**ActivityStack#resumeTopActivityInnerLocked()**

```
private boolean resumeTopActivityInnerLocked(ActivityRecord prev, Bundle options) {
        ...
        // If the top activity is the resumed one, nothing to do.
        if (mResumedActivity == next && next.state == ActivityState.RESUMED &&
                    mStackSupervisor.allResumedActivitiesComplete()) {
            ...
            return false;
        }
        ...
        if (mResumedActivity != null) {
            if (DEBUG_STATES) Slog.d(TAG_STATES,
                    "resumeTopActivityLocked: Pausing " + mResumedActivity);
            pausing |= startPausingLocked(userLeaving, false, true, dontWaitForPause);
        }
        ...
        mStackSupervisor.startSpecificActivityLocked(next, true, true);
        ...
    }
```


**实在是太长了，后面还有很多**

详情见：[Android Launcher启动应用程序流程源码解析 - 一口仨馍 - 博客频道 - CSDN.NET](http://blog.csdn.net/qq_17250009/article/details/52204918)


**总结**

- Launcher通过Binder机制通知AMS启动一个Activity
- AMS使Launcher栈最顶端Activity进入onPause状态
- AMS通知Process使用Socket和Zygote进程通信，请求创建一个新进程
- Zygote收到Socket请求，fork出一个进程，并调用ActivityThread#main()
- ActivityThread通过Binder通知AMS启动应用程序
- AMS通知ActivityStackSupervisor真正的启动Activity
- ActivityStackSupervisor通知ApplicationThread启动Activity
- ApplicationThread发消息给ActivityThread，需要启动一个Activity
- ActivityThread收到消息之后，通知LoadedApk创建Applicaition，并且调用其onCteate()方法
- ActivityThread装载目标Activity类，并调用Activity#attach()
- ActivityThread通知Instrumentation调用Activity#onCreate()
- Instrumentation调用Activity#performCreate()，在Activity#performCreate()中调用自身onCreate()方法








