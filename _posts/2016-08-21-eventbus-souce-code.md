---
layout: post
title: EventBus源码解析
date: 2016-8-21
categories: blog
tags: [源码解析]
description: EventBus
---

#### 概述
一般使用EventBus的组件类，类似下面这种方式：

```
public class SampleComponent extends Fragment
{

    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        EventBus.getDefault().register(this);
    }

    public void onEventMainThread(param)
    {
    }
    
    public void onEventPostThread(param)
    {
        
    }
    
    public void onEventBackgroundThread(param)
    {
        
    }
    
    public void onEventAsync(param)
    {
        
    }
    
    @Override
    public void onDestroy()
    {
        super.onDestroy();
        EventBus.getDefault().unregister(this);
    }
    
}
```

大多情况下，都会在onCreate中进行register，在onDestory中进行unregister ；

看完代码大家或许会有一些疑问：

1、代码中还有一些以onEvent开头的方法，这些方法是干嘛的呢？              
在回答这个问题之前，我有一个问题，你咋不问register（this）是干嘛的呢？其实register(this)就是去当前类，遍历所有的方法，找到onEvent开头的然后进行存储。现在知道onEvent开头的方法是干嘛的了吧。

2、那onEvent后面的那些MainThread应该是什么标志吧？            
嗯，是的，onEvent后面可以写四种，也就是上面出现的四个方法，决定了当前的方法最终在什么线程运行，怎么运行，可以参考上一篇博客或者细细往下看。     

既然register了，那么肯定得说怎么调用是吧。

EventBus.getDefault().post(param);  

调用很简单，一句话，你也可以叫发布，只要把这个param发布出去，EventBus会在它内部存储的方法中，进行扫描，找到参数匹配的，就使用反射进行调用。                    
现在有没有觉得，撇开专业术语：其实EventBus就是在内部存储了一堆onEvent开头的方法，然后post的时候，根据post传入的参数，去找到匹配的方法，反射调用之。                        
那么，我告诉你，它内部使用了Map进行存储，键就是参数的Class类型。知道是这个类型，那么你觉得根据post传入的参数进行查找还是个事么？                  

![](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/event-bus/event-bus/image/register-flow-chart.png)

![](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/event-bus/event-bus/image/post-flow-chart.png)


#### register              

EventBus.getDefault().register(this);    

首先：             
EventBus.getDefault()其实就是个单例，和我们传统的getInstance一个意思：    

```
 /** Convenience singleton for apps using a process-wide EventBus instance. */
    public static EventBus getDefault() {
        if (defaultInstance == null) {
            synchronized (EventBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new EventBus();
                }
            }
        }
        return defaultInstance;
    }
```

使用了双重判断的方式，防止并发的问题，还能极大的提高效率。
然后register应该是一个普通的方法，我们去看看：
register公布给我们使用的有4个：

```
 public void register(Object subscriber) {
        register(subscriber, DEFAULT_METHOD_NAME, false, 0);
    }
 public void register(Object subscriber, int priority) {
        register(subscriber, DEFAULT_METHOD_NAME, false, priority);
    }
public void registerSticky(Object subscriber) {
        register(subscriber, DEFAULT_METHOD_NAME, true, 0);
    }
public void registerSticky(Object subscriber, int priority) {
        register(subscriber, DEFAULT_METHOD_NAME, true, priority);
    }
```


本质上就调用了同一个：

```
private synchronized void register(Object subscriber, String methodName, boolean sticky, int priority) {
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass(),
                methodName);
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod, sticky, priority);
        }
    }
```

四个参数                 

- subscriber 是我们扫描类的对象，也就是我们代码中常见的this;          
- methodName 这个是写死的：“onEvent”，用于确定扫描什么开头的方法，可见我们的类中都是以这个开头。
- sticky 这个参数，解释源码的时候解释，暂时不用管
- priority 优先级，优先级越高，在调用的时候会越先调用。

下面开始看代码：

```
List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass(),methodName);
```

调用内部类SubscriberMethodFinder的findSubscriberMethods方法，传入了subscriber 的class，以及methodName，返回一个List<SubscriberMethod>。
那么不用说，肯定是去遍历该类内部所有方法，然后根据methodName去匹配，匹配成功的封装成SubscriberMethod，最后返回一个List。下面看代码：

```
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass, String eventMethodName) {
        String key = subscriberClass.getName() + '.' + eventMethodName;
        List<SubscriberMethod> subscriberMethods;
        synchronized (methodCache) {
            subscriberMethods = methodCache.get(key);
        }
        if (subscriberMethods != null) {
            return subscriberMethods;
        }
        subscriberMethods = new ArrayList<SubscriberMethod>();
        Class<?> clazz = subscriberClass;
        HashSet<String> eventTypesFound = new HashSet<String>();
        StringBuilder methodKeyBuilder = new StringBuilder();
        while (clazz != null) {
            String name = clazz.getName();
            if (name.startsWith("java.") || name.startsWith("javax.") || name.startsWith("android.")) {
                // Skip system classes, this just degrades performance
                break;
            }

            // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
            Method[] methods = clazz.getMethods();
            for (Method method : methods) {
                String methodName = method.getName();
                if (methodName.startsWith(eventMethodName)) {
                    int modifiers = method.getModifiers();
                    if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                        Class<?>[] parameterTypes = method.getParameterTypes();
                        if (parameterTypes.length == 1) {
                            String modifierString = methodName.substring(eventMethodName.length());
                            ThreadMode threadMode;
                            if (modifierString.length() == 0) {
                                threadMode = ThreadMode.PostThread;
                            } else if (modifierString.equals("MainThread")) {
                                threadMode = ThreadMode.MainThread;
                            } else if (modifierString.equals("BackgroundThread")) {
                                threadMode = ThreadMode.BackgroundThread;
                            } else if (modifierString.equals("Async")) {
                                threadMode = ThreadMode.Async;
                            } else {
                                if (skipMethodVerificationForClasses.containsKey(clazz)) {
                                    continue;
                                } else {
                                    throw new EventBusException("Illegal onEvent method, check for typos: " + method);
                                }
                            }
                            Class<?> eventType = parameterTypes[0];
                            methodKeyBuilder.setLength(0);
                            methodKeyBuilder.append(methodName);
                            methodKeyBuilder.append('>').append(eventType.getName());
                            String methodKey = methodKeyBuilder.toString();
                            if (eventTypesFound.add(methodKey)) {
                                // Only add if not already found in a sub class
                                subscriberMethods.add(new SubscriberMethod(method, threadMode, eventType));
                            }
                        }
                    } else if (!skipMethodVerificationForClasses.containsKey(clazz)) {
                        Log.d(EventBus.TAG, "Skipping method (not public, static or abstract): " + clazz + "."
                                + methodName);
                    }
                }
            }
            clazz = clazz.getSuperclass();
        }
        if (subscriberMethods.isEmpty()) {
            throw new EventBusException("Subscriber " + subscriberClass + " has no public methods called "
                    + eventMethodName);
        } else {
            synchronized (methodCache) {
                methodCache.put(key, subscriberMethods);
            }
            return subscriberMethods;
        }
    }
```

呵，代码还真长；不过我们直接看核心部分：          
22行：看到没，clazz.getMethods();去得到所有的方法：           
23-62行：就开始遍历每一个方法了，去匹配封装了。                
25-29行：分别判断了是否以onEvent开头，是否是public且非static和abstract方法，是否是一个参数。如果都复合，才进入封装的部分。             
32-45行：也比较简单，根据方法的后缀，来确定threadMode，threadMode是个枚举类型：就四种情况。        
最后在54行：将method, threadMode, eventType传入构造了：new SubscriberMethod(method, threadMode, eventType)。添加到List，最终放回。                
注意下63行：clazz = clazz.getSuperclass();可以看到，会扫描所有的父类，不仅仅是当前类。          
继续回到register:         

```
for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod, sticky, priority);
        }
```

for循环扫描到的方法，然后去调用suscribe方法。

```
 // Must be called in synchronized block
    private void subscribe(Object subscriber, SubscriberMethod subscriberMethod, boolean sticky, int priority) {
        subscribed = true;
        Class<?> eventType = subscriberMethod.eventType;
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod, priority);
        if (subscriptions == null) {
            subscriptions = new CopyOnWriteArrayList<Subscription>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            for (Subscription subscription : subscriptions) {
                if (subscription.equals(newSubscription)) {
                    throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                            + eventType);
                }
            }
        }

        // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
        // subscriberMethod.method.setAccessible(true);

        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || newSubscription.priority > subscriptions.get(i).priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }

        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<Class<?>>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);

        if (sticky) {
            Object stickyEvent;
            synchronized (stickyEvents) {
                stickyEvent = stickyEvents.get(eventType);
            }
            if (stickyEvent != null) {
                // If the subscriber is trying to abort the event, it will fail (event is not tracked in posting state)
                // --> Strange corner case, which we don't take care of here.
                postToSubscription(newSubscription, stickyEvent, Looper.getMainLooper() == Looper.myLooper());
            }
        }
    }
```

我们的subscriberMethod中保存了method, threadMode, eventType，上面已经说了；

4-17行：根据subscriberMethod.eventType，去subscriptionsByEventType去查找一个CopyOnWriteArrayList<Subscription> ，如果没有则创建。
顺便把我们的传入的参数封装成了一个：Subscription（subscriber, subscriberMethod, priority）；
这里的subscriptionsByEventType是个Map，key：eventType ； value：CopyOnWriteArrayList<Subscription> ； 这个Map其实就是EventBus存储方法的地方，一定要记住！

22-28行：实际上，就是添加newSubscription；并且是按照优先级添加的。可以看到，优先级越高，会插到在当前List的前面。

30-35行：根据subscriber存储它所有的eventType ； 依然是map；key：subscriber ，value：List<eventType> ;知道就行，非核心代码，主要用于isRegister的判断。

37-47行：判断sticky；如果为true，从stickyEvents中根据eventType去查找有没有stickyEvent，如果有则立即发布去执行。stickyEvent其实就是我们post时的参数。
postToSubscription这个方法，我们在post的时候会介绍。

到此，我们register就介绍完了。

你只要记得一件事：扫描了所有的方法，把匹配的方法最终保存在subscriptionsByEventType（Map，key：eventType ； value：CopyOnWriteArrayList<Subscription> ）中；

eventType是我们方法参数的Class，Subscription中则保存着subscriber, subscriberMethod（method, threadMode, eventType）, priority；包含了执行改方法所需的一切。


#### post
register完毕，知道了EventBus如何存储我们的方法了，下面看看post它又是如何调用我们的方法的。
再看源码之前，我们猜测下：register时，把方法存在subscriptionsByEventType；那么post肯定会去subscriptionsByEventType去取方法，然后调用。

下面看源码：

```
 /** Posts the given event to the event bus. */
    public void post(Object event) {
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        eventQueue.add(event);

        if (postingState.isPosting) {
            return;
        } else {
            postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                while (!eventQueue.isEmpty()) {
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
```

currentPostingThreadState是一个ThreadLocal类型的，里面存储了PostingThreadState；PostingThreadState包含了一个eventQueue和一些标志位。

```
 private final ThreadLocal<PostingThreadState> currentPostingThreadState = new ThreadLocal<PostingThreadState>() {
        @Override
        protected PostingThreadState initialValue() {
            return new PostingThreadState();
        }
    }
```

把我们传入的event，保存到了当前线程中的一个变量PostingThreadState的eventQueue中。         
10行：判断当前是否是UI线程。                                                                     
16-18行：遍历队列中的所有的event，调用postSingleEvent（eventQueue.remove(0), postingState）方法。     

这里大家会不会有疑问，每次post都会去调用整个队列么，那么不会造成方法多次调用么？
可以看到第7-8行，有个判断，就是防止该问题的，isPosting=true了，就不会往下走了。

下面看postSingleEvent

```
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
        Class<? extends Object> eventClass = event.getClass();
        List<Class<?>> eventTypes = findEventTypes(eventClass);
        boolean subscriptionFound = false;
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            CopyOnWriteArrayList<Subscription> subscriptions;
            synchronized (this) {
                subscriptions = subscriptionsByEventType.get(clazz);
            }
            if (subscriptions != null && !subscriptions.isEmpty()) {
                for (Subscription subscription : subscriptions) {
                    postingState.event = event;
                    postingState.subscription = subscription;
                    boolean aborted = false;
                    try {
                        postToSubscription(subscription, event, postingState.isMainThread);
                        aborted = postingState.canceled;
                    } finally {
                        postingState.event = null;
                        postingState.subscription = null;
                        postingState.canceled = false;
                    }
                    if (aborted) {
                        break;
                    }
                }
                subscriptionFound = true;
            }
        }
        if (!subscriptionFound) {
            Log.d(TAG, "No subscribers registered for event " + eventClass);
            if (eventClass != NoSubscriberEvent.class && eventClass != SubscriberExceptionEvent.class) {
                post(new NoSubscriberEvent(this, event));
            }
        }
    }
```

将我们的event，即post传入的实参；以及postingState传入到postSingleEvent中。   

2-3行：根据event的Class，去得到一个List<Class<?>>；其实就是得到event当前对象的Class，以及父类和接口的Class类型；主要用于匹配，比如你传入Dog extends Animal，他会把Animal也装到该List中。

6-31行：遍历所有的Class，到subscriptionsByEventType去查找subscriptions；哈哈，熟不熟悉，还记得我们register里面把方法存哪了不？
是不是就是这个Map;

12-30行：遍历每个subscription,依次去调用postToSubscription(subscription, event, postingState.isMainThread);
这个方法就是去反射执行方法了，大家还记得在register，if(sticky)时，也会去执行这个方法。

下面看它如何反射执行：

```
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        switch (subscription.subscriberMethod.threadMode) {
        case PostThread:
            invokeSubscriber(subscription, event);
            break;
        case MainThread:
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            } else {
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case BackgroundThread:
            if (isMainThread) {
                backgroundPoster.enqueue(subscription, event);
            } else {
                invokeSubscriber(subscription, event);
            }
            break;
        case Async:
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }
```

前面已经说过subscription包含了所有执行需要的东西，大致有:subscriber, subscriberMethod（method, threadMode, eventType）, priority；
那么这个方法：第一步根据threadMode去判断应该在哪个线程去执行该方法；
case PostThread:

```
 void invokeSubscriber(Subscription subscription, Object event) throws Error {
            subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
  ｝
```

直接反射调用；也就是说在当前的线程直接调用该方法；   

- case MainThread:             
首先去判断当前如果是UI线程，则直接调用；否则： mainThreadPoster.enqueue(subscription, event);把当前的方法加入到队列，然后直接通过handler去发送一个消息，在handler的handleMessage中，去执行我们的方法。说白了就是通过Handler去发送消息，然后执行的。
- case BackgroundThread:                 
如果当前非UI线程，则直接调用；如果是UI线程，则将任务加入到后台的一个队列，最终由Eventbus中的一个线程池去调用
executorService = Executors.newCachedThreadPool();。
- case Async:                 
将任务加入到后台的一个队列，最终由Eventbus中的一个线程池去调用；线程池与BackgroundThread用的是同一个。

这么说BackgroundThread和Async有什么区别呢？                    
BackgroundThread中的任务，一个接着一个去调用，中间使用了一个布尔型变量handlerActive进行的控制。
Async则会动态控制并发。

到此，我们完整的源码分析就结束了，总结一下：register会把当前类中匹配的方法，存入一个map，而post会根据实参去map查找进行反射调用。分析这么久，一句话就说完了~~

其实不用发布者，订阅者，事件，总线这几个词或许更好理解，以后大家问了EventBus，可以说，就是在一个单例内部维持着一个map对象存储了一堆的方法；post无非就是根据参数去查找方法，进行反射调用


#### 其余方法   

介绍了register和post；大家获取还能想到一个词sticky，在register中，如何sticky为true，会去stickyEvents去查找事件，然后立即去post；    

那么这个stickyEvents何时进行保存事件呢？       
其实evevntbus中，除了post发布事件，还有一个方法也可以    

```
 public void postSticky(Object event) {
        synchronized (stickyEvents) {
            stickyEvents.put(event.getClass(), event);
        }
        // Should be posted after it is putted, in case the subscriber wants to remove immediately
        post(event);
    }
```

和post功能类似，但是会把方法存储到stickyEvents中去；
大家再去看看EventBus中所有的public方法，无非都是一些状态判断，获取事件，移除事件的方法；没什么好介绍的，基本见名知意。


**参考链接**        

[Android EventBus源码解析 带你深入理解EventBus - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/40920453)

[EventBus源码解析](http://a.codekk.com/detail/Android/Trinea/EventBus%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)


### EventBus3.0源码解析           

![](http://img.blog.csdn.net/20160710021031385)

可以看到，发布者(Publisher)使用post()方法将Event发送到Event Bus，而后Event Bus自动将Event发送到多个订阅者(Subcriber)。这里需要注意两个地方：

（1）一个发布者可以对应多个订阅者。

（2）3.0以前订阅者的订阅方法为onEvent()、onEventMainThread()、onEventBackgroundThread()和onEventAsync()。在Event Bus3.0之后统一采用注解@Subscribe的形式


注解标签Subscribe

对注解不了解的同学可以看下这篇[博客](http://www.cnblogs.com/yydcdut/p/4646454.html)。

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface Subscribe {
    ThreadMode threadMode() default ThreadMode.POSTING;

    /**
     * If true, delivers the most recent sticky event (posted with
     * {@link EventBus#postSticky(Object)}) to this subscriber (if event available).
     */
    boolean sticky() default false;

    /** Subscriber priority to influence the order of event delivery.
     * Within the same delivery thread ({@link ThreadMode}), higher priority subscribers will receive events before
     * others with a lower priority. The default priority is 0. Note: the priority does *NOT* affect the order of
     * delivery among subscribers with different {@link ThreadMode}s! */
    int priority() default 0;
}

public enum ThreadMode {

    POSTING,

    MAIN,

    BACKGROUND,

    ASYNC
}
```


注解Subscribe在运行时解析，且只能加在METHOD上。其中有三个方法，threadMode()返回类型ThreadMode为枚举类型，默认值为POSTING，sticky()默认返回false,priority()默认返回0。


关于register与post的详细过程可以参见：

[Android EventBus3.0使用及源码解析 ](http://blog.csdn.net/qq_17250009/article/details/51872731)