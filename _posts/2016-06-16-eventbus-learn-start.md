---
layout: post
title: EventBus学习入门
date: 2016-6-16
categories: blog
tags: [android]
description: EventBus学习入门
---


**EventBus Features**  

What makes greenrobot’s EventBus unique, are its features:

- Simple yet powerful: EventBus is a tiny library with an API that is super easy to learn. Nevertheless, your software architecture may great benefit by decoupling components: Subscribers do not have know about senders, when using events.
- Battle tested: EventBus is one of the most used Android libraries: thousands of apps use EventBus including very popular ones. Over a billion app installs speak for themselves.
- High Performance: Especially on Android, performance matters. EventBus was profiled and optimized a lot; probably making it the fastest solution of its kind.
- Convenient Annotation based API (without sacrificing performance): Simply put the @Subscribe annotation to your subscriber methods. Because of a build time indexing of annotations, EventBus does not need to do annotation reflection during your app’s run time, which is very slow on Android.
- Android main thread delivery: When interacting with the UI, EventBus can deliver events in the main thread regardless how an event was posted.
- Background thread delivery: If your subscriber does long running tasks, EventBus can also use background threads to avoid UI blocking.
- Event & Subscriber inheritance: In EventBus, the object oriented paradigm apply to event and subscriber classes. Let’s say event class A is the superclass of B. Posted events of type B will also be posted to subscribers interested in A. Similarly the inheritance of subscriber classes are considered.
- Zero configuration: You can get started immediately using a default EventBus instance available from anywhere in your code.
- Configurable: To tweak EventBus to your requirements, you can adjust its behavior using the builder pattern.

![](https://github.com/greenrobot/EventBus/raw/master/EventBus-Publish-Subscribe.png)


### Quickly Start


EventBus’ API is as easy as 1-2-3. Before we get started with the 3 basic steps, let’s add the EventBus depency to your Gradle script (make sure you are using the latest version):

```
 compile 'org.greenrobot:eventbus:3.0.0'
```

**Step 1: Define events**

Events are POJO (plain old Java object) without any specific requirements.

```
public class MessageEvent {
    public final String message;

    public MessageEvent(String message) {
        this.message = message;
    }
}
```

**Step 2: Prepare subscribers**

Subscribers implement event handling methods (also called “subscriber methods”) that will be called when an event is posted. These are defined with the @Subscribe annotation. Please note that with EventBus 3 the method name can be chosen freely (no naming conventions like in EventBus 2).

```
// This method will be called when a MessageEvent is posted
@Subscribe
public void onMessageEvent(MessageEvent event){
    Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
}

// This method will be called when a SomeOtherEvent is posted
@Subscribe
public void handleSomethingElse(SomeOtherEvent event){
    doSomethingWith(event);
}
```


Subscribers also need to register and unregister themselves to the bus. Only while subscribers are registered, they will receive events. In Android, Activities and Fragments usually bind according to their life cycle:


```
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

@Override
public void onStop() {
   EventBus.getDefault().unregister(this);
    super.onStop();
}
```


**Step 3: Post events** 

Post an event from any part of your code. All currently registered subscribers matching the event type will receive it.

```
EventBus.getDefault().post(new MessageEvent("Hello everyone!"));
```   


### Delivery Threads (ThreadMode)  


EventBus can handle threading for you: events can be posted in threads different from the posting thread. A common use case is dealing with UI changes. In Android, UI changes must be done in the UI (main) thread. On the other hand, networking, or any time consuming task, must not run on the main thread. EventBus helps you to deal with those tasks and synchronize with the UI thread (without having to delve into thread transitions, using AsyncTask, etc).

In EventBus, you may define the thread that will call the event handling method by using one of the four ThreadModes.

**ThreadMode: POSTING**

Subscribers will be called in the same thread posting the event. This is the default. Event delivery is done synchronously and all subscribers will have been called once the posting is done. This ThreadMode implies the least overhead because it avoids thread switching completely. Thus this is the recommended mode for simple tasks that are known to complete is a very short time without requiring the main thread. Event handlers using this mode should return quickly to avoid blocking the posting thread, which may be the main thread. Example:

```
// Called in the same thread (default)

@Subscribe(threadMode = ThreadMode.POSTING) // ThreadMode is optional here
public void onMessage(MessageEvent event) {
    log(event.message);
}
```


**ThreadMode: MAIN**

Subscribers will be called in Android’s main thread (sometimes referred to as UI thread). If the posting thread is the main thread, event handler methods will be called directly (synchronously like described for ThreadMode.POSTING). Event handlers using this mode must return quickly to avoid blocking the main thread. Example:

```
// Called in Android UI's main thread
@Subscribe(threadMode = ThreadMode.MAIN)
public void onMessage(MessageEvent event) {
    textField.setText(event.message);
}
```

**ThreadMode: BACKGROUND**

Subscribers will be called in a background thread. If posting thread is not the main thread, event handler methods will be called directly in the posting thread. If the posting thread is the main thread, EventBus uses a single background thread that will deliver all its events sequentially. Event handlers using this mode should try to return quickly to avoid blocking the background thread.

```
// Called in the background thread
@Subscribe(threadMode = ThreadMode.BACKGROUND)
public void onMessage(MessageEvent event){
    saveToDisk(event.message);
}
```

**ThreadMode: ASYNC**

Event handler methods are called in a separate thread. This is always independent from the posting thread and the main thread. Posting events never wait for event handler methods using this mode. Event handler methods should use this mode if their execution might take some time, e.g. for network access. Avoid triggering a large number of long running asynchronous handler methods at the same time to limit the number of concurrent threads. EventBus uses a thread pool to efficiently reuse threads from completed asynchronous event handler notifications.

```
// Called in a separate thread
@Subscribe(threadMode = ThreadMode.ASYNC)
public void onMessage(MessageEvent event){
    backend.send(event.message);
}
```

### References  

[Delivery Threads (ThreadMode) | Open Source by greenrobot](http://greenrobot.org/eventbus/documentation/delivery-threads-threadmode/)

[Android EventBus实战 没听过你就out了 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/40794879)

[快速Android开发系列通信篇之EventBus - AngelDevil - 博客园](http://www.cnblogs.com/angeldevil/p/3715934.html)

[Android EventBus源码解析 带你深入理解EventBus - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/40920453)

