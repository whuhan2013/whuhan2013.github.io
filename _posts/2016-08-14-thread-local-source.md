---
layout: post
title: ThreadLocal源码解析
date: 2016-8-14
categories: blog
tags: [源码解析]
description: 源码解析
---

**ThreadLocal是什么？**   

> Implements a thread-local storage, that is, a variable for which each thread has its own value. All threads share the same ThreadLocal object, but each sees a different value when accessing it, and changes made by one thread do not affect the other threads. The implementation supports null values. 
实现了每个线程的自有变量的存储。所有线程共享同一个ThreadLocal对象，但是每个线程只能访问自己所存储的变量，并且线程之间做的改动互不影响。此实现支持null变量的存储。（非逐词翻译）


通过这个描述，我们可以知道一些信息：

- ThreadLocal是一个线程的内部存储类，通过它我们可以在指定的线程中存储信息。
- 每个线程之间的信息是独立且封闭的。


#### ThreadLocal的用途

ThreadLocal的作用是实现每个线程的自有变量的存储，这个“自有变量”具体是什么当然要根据不同的需求来定，但是在Android的源码中，这个自有变量常常是Looper。

这个Looper是什么呢？这里就不得不提到Android的消息机制了。Android的消息机制是指Handler的运行机制以及Handler所附带的MessageQueue和Looper的工作过程。简单来讲，就是Handler发送Message到MessageQueue中，而Looper不断的从MessageQueue中循环摘取Message，并进行进一步的解析处理。

但是Looper只是个简单的类而已，它虽然提供了循环处理方面的成员函数loop()，却不能自己凭空地运行起来，而只能寄身于某个真实的线程。那么Looper是怎么和线程建立联系的呢？这个时候ThreadLocal就参与其中了。

我们来看一下Looper的部分源码：

```
private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        //调用ThreadLocal的set()方法将Looper存进去
        sThreadLocal.set(new Looper(quitAllowed));
}



public static void loop() {
        //调用myLooper()方法得到Looper，我们接着看一下myLooper()方法
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        …… ……
}



public static @Nullable Looper myLooper() {
        //调用ThreadLocal的get()方法得到刚才存入的Looper对象
        return sThreadLocal.get();
}
```


#### ThreadLocal用法   

它的用法其实挺简单的，暴露出来的方法一共只有三个:

- get()：返回调用线程中变量的当前值
- set(T value)：设置调用线程中变量的值
- remove()：移除调用线程的当前变量    

用的话也是这三个方法，就跟上面Looper里面的用法差不多。例子的代码如下：

```
mIntegerThreadLocal = new ThreadLocal<>();

mIntegerThreadLocal.set(0);
//在主线程里设置mIntegerThreadLocal的值为0，输出0
Log.d(TAG, "[Thread#main]mIntegerThreadLocal=" + mIntegerThreadLocal.get());

new Thread("Thread#1") {
    @Override
    public void run() {
        //设置mIntegerThreadLocal的值为1，输出1
        mIntegerThreadLocal.set(1);
        Log.d(TAG, "[Thread#1]mIntegerThreadLocal=" + mIntegerThreadLocal.get());
    }
}.start();

new Thread("Thread#2") {
    @Override
    public void run() {
        //不设置任何值，输出null
        Log.d(TAG, "[Thread#2]mIntegerThreadLocal=" + mIntegerThreadLocal.get());
    }
}.start();

new Thread("Thread#3") {
    @Override
    public void run() {
        //设置为3，然后remove，输出null
        mIntegerThreadLocal.set(3);
        mIntegerThreadLocal.remove();
        Log.d(TAG, "[Thread#3]mIntegerThreadLocal=" + mIntegerThreadLocal.get());
    }
}.start();
```


#### 源码解析        

先看一下它的构造方法：

```
public ThreadLocal() {}
```

可以的，啥都没干，空构造，这条路子走不通。


**暴露方法**

- public void set(T value)

接下来从暴露出来的方法入手，先看set()方法：

```
public void set(T value) {
    //Thread.currentThread()方法是一个native的方法，功能是返回调用这个方法的线程
    Thread currentThread = Thread.currentThread();
    Values values = values(currentThread);
    if (values == null) {
        values = initializeValues(currentThread);
    }
    values.put(this, value);
}
```

- Values

native的方法咱们先不看，知道他是返回调用线程就好了，直接看第二行。第二行出现了一个之前没见过的类：Values。接下来找找Values是何方神圣：


```
static class Values {

        /**
         * 大小是2的n次方
         */
        private static final int INITIAL_SIZE = 16;

        /**
         * 被删除的数据的占位符，因为是用object数组查询的数据，所以需要这个
         */
        private static final Object TOMBSTONE = new Object();

        /**
         * 存放数据的数组。他存放数据的形式有点像map,是ThreadLocal与value相对应
         * 
         * 长度总是2的N次方
         */
        private Object[] table;

        /** 用来将hash地址转换为索引 */
        private int mask;

        /** 当前存活数据的数目 */
        private int size;

        /** 当前失效数据的数目 */
        private int tombstones;

        /** 允许的存活数据与失效数据数目总和的最大值 */
        private int maximumLoad;

        /** 指向下一个要清理的数据 */
        private int clean;
```

原来Values是ThreadLocal的一个静态的内部类，它的功能就是存储放进来的数据以及对数据进行处理，比如清理失效数据啦，扩展存储的table啦，等等。它存储数据是用的Object数组，并且有一些值来记录当前存活的数据以及失效数据的数目，通过这些数据它可以及时的清理无效的数据并且扩展或者缩小当前table的大小，便于保持当前table的健壮性。


由于这个内部类还是很重要的，我们再往里面挖掘一下它里面的源码，看看到底是怎么一卵回事： 

![](http://ac-cnyv47la.clouddn.com/cd00dbfdce1476b0.png)

可以看到，这里面有俩构造方法： 

- Values() 
- Values(Values values)                    
并且有四个暴露出来的方法：

```
 - void put(ThreadLocal<?> key, Object value)：往table里添加一个键值对
 - void add(ThreadLocal<?> key, Object value)：也是往table里面添加键值对，但是比起put()来少了很多繁琐但是很棒的操作，下面再详述
 - Object getAfterMiss(ThreadLocal<?> key)：在首位置没找到值的时候通过这个方法来找到给定key的值
 - void remove(ThreadLocal<?> key)：删掉给定key对应的值
 ```

 **构造方法**     

 ```
 Values() {
    //初始化table
    initializeTable(INITIAL_SIZE);
    this.size = 0;
    this.tombstones = 0;
}

Values(Values fromParent) {
    this.table = fromParent.table.clone();
    this.mask = fromParent.mask;
    this.size = fromParent.size;
    this.tombstones = fromParent.tombstones;
    this.maximumLoad = fromParent.maximumLoad;
    this.clean = fromParent.clean;
    inheritValues(fromParent);
}
```

这两个构造方法一个是普通的构造，一个是类似于继承的那种，从一个父Values对象来生成新的Values，我们先看普通的这个构造。它就只是做了一些常规的初始化操作，initializeTable(int size)方法是这样的：

```
 private void initializeTable(int capacity) {
    //新建values里面的table，默认大小是32
    this.table = new Object[capacity * 2];
    //mask的默认大小是table的长度减一，也就是table中最后一个元素的索引
    this.mask = table.length - 1;
    this.clean = 0;
    //默认的最大load值是capacity的2/3，其实我不是很能理解这个值是怎么得来的
    this.maximumLoad = capacity * 2 / 3; // 2/3
}
```

void add(ThreadLocal

```
void add(ThreadLocal<?> key, Object value) {
    for (int index = key.hash & mask;; index = next(index)) {
        Object k = table[index];
        if (k == null) {
            //在index处放入一个键
            table[index] = key.reference;
            //在index的下一位放入键对应的值
            table[index + 1] = value;
            return;
        }
    }
}
```


上面的代码向我们展示了table存储数据的方式，它是以一种类似于map的方式来存储的，在index处存入map的键，在index的下一位存入键对应的值，而这个键则是ThreadLocal的引用，这里毫无问题。但是有一个地方问题则是大大的有： int index = key.hash & mask 。大家都明白这行代码的作用是获得可用的索引，可是到底是怎么获得的呢？为什么要通过这种方式来获得呢？要解决这个问题，我们要先知道何为“可用的索引”，通过分析观察，我总结出了一些条件： 

- 要是偶数。（这个很显然） 
- 不能越界。（都比table的边界长了那还了得） 
- 尽可能分散。（尽可能的新产生的索引不要是已经出现过的数，不然table的空间不能充分的利用，而且观察上面代码，会发现如果新产生的索引是已经出现过的数的话数据根本存不进去）


**void put(ThreadLocal** 

```
void put(ThreadLocal<?> key, Object value) {
    cleanUp();

    //标志第一个失效键的index
    int firstTombstone = -1;
    //获取当前index
    for (int index = key.hash & mask;; index = next(index)) {
        Object k = table[index];
        //如果键相同，就用传进来的值替换原来的值
        if (k == key.reference) {
            // Replace existing entry.
            table[index + 1] = value;
            return;
        }
        //如果键为空，就放入传进来的键值
        if (k == null) {
            //firstTombstone == -1表示firstTombstone还没有改变过，即循环到现在还没有发现失效键
            if (firstTombstone == -1) {
                //放入传进来的键值
                table[index] = key.reference;
                table[index + 1] = value;
                size++;
                return;
            }

            // 如果firstTombstone ！= -1，说明上一个循环中遇到了失效键，那么就将放入传进来的键值放入那个失效键值的位置
            table[firstTombstone] = key.reference;
            table[firstTombstone + 1] = value;
            tombstones--;
            size++;
            return;
        }
        // 如果当前键为失效键，则标记其位置，准备在其位置放入传入的键值对
        if (firstTombstone == -1 && k == TOMBSTONE) {
            firstTombstone = index;
        }
    }
}
```

可以看到，put()方法里面的逻辑其实很简单，就是在想方设法的把传进来的键值对给存进去——其中对获得的index的值进行了一些判断，以决定如何进行存储——总之是想要尽可能的节省空间。另外，值得注意的是，在遇到相同索引处存放着同一个键的时候，其采取的方式是新值换旧值。除此之外，我们可以看到，在方法的第一行其调用了另一个方法cleanUp()，这个方法又是干嘛的呢？


```
private void cleanUp() {
    if (rehash()) {
        // 如果rehash()方法返回的是true，就不需要继续clean up了
        return;
    }
    //如果table的size是0的话，也不需要clean up.
    //都没有键值在里面有什么好clean的？
    if (size == 0) {
        return;
    }

    // 默认的clean是0.
    //下一次再clean的时候是从上一次clean的地方继续
    int index = clean;
    Object[] table = this.table;
    for (int counter = table.length; counter > 0; counter >>= 1,
        index = next(index)) {
        Object k = table[index];
        //如果键是TOMBSTONE或者null的话，就可以下一次循环了
        if (k == TOMBSTONE || k == null) {
            continue;
        }

        // 这个table只有可能存储null, tombstones 和 references的键.
        @SuppressWarnings("unchecked")
        Reference<ThreadLocal<?>> reference = (Reference<ThreadLocal<?>>) k;
        // 检查键是否已经失效，如果已经失效就把它设置为失效键，同时释放它的值
        if (reference.get() == null) {
            table[index] = TOMBSTONE;
            table[index + 1] = null;
            tombstones++;
            size--;
        }
    }

    // 指向下一个index
    clean = index;
}
```

cleanUp()方法只做了一件事，就是把失效的键放上TOMBSTONE去占位，然后释放它的值。那么rehash()是干什么的其实已经很显而易见了： 

- 从字面意思来也知道是重新规划table的大小。 
- 联想cleanUp()的作用，它都已经把失效键放上TOMBSTONE，接下来呢？显然是想办法干掉这些TOMBSTONE，还我内存一个朗朗乾坤喽。

**Object getAfterMiss(ThreadLocal** 

```
public T get() {
    // 获得调用线程
    Thread currentThread = Thread.currentThread();
    //获得调用线程的Values
    Values values = values(currentThread);
    if (values != null) {
        Object[] table = values.table;
        int index = hash & values.mask;
        //如果在当前index处的键不是此线程的引用，就会去调用getAfterMiss()
        if (this.reference == table[index]) {
            return (T) table[index + 1];
        }
    } else {
        values = initializeValues(currentThread);
    }
    return (T) values.getAfterMiss(this);
}
```

它的作用就是在第一个位置没有找到此键对应的值的时候继续查询，达到获得其值的目的。但是在看它的代码的过程中，我整个人是懵逼的。为什么呢？按照我的想法，没有在第一个位置得到这个调用线程存储的信息之后，应该下一步是一个一个index的循环这个table，来找到调用线程存储的信息，如果实在找不到，就说明调用线程没有存储信息在里面，那么就应该初始化一个value给它——当然，这个初始化value的方法理应是可以重写的。初始化之后，就是用一种巧妙的方法把它存进table里面，然后返回这个value了。


可是，它的源码根本不是这样！


```
Object getAfterMiss(ThreadLocal<?> key) {
            Object[] table = this.table;
            int index = key.hash & mask;

            // If the first slot is empty, the search is over.
            if (table[index] == null) {
                Object value = key.initialValue();

                // If the table is still the same and the slot is still empty...
                if (this.table == table && table[index] == null) {
                    table[index] = key.reference;
                    table[index + 1] = value;
                    size++;

                    cleanUp();
                    return value;
                }

                // The table changed during initialValue().
                put(key, value);
                return value;
            }

            // Keep track of first tombstone. That's where we want to go back
            // and add an entry if necessary.
            int firstTombstone = -1;

            // Continue search.
            for (index = next(index);; index = next(index)) {
                Object reference = table[index];
                if (reference == key.reference) {
                    return table[index + 1];
                }

                // If no entry was found...
                if (reference == null) {
                    Object value = key.initialValue();

                    // If the table is still the same...
                    if (this.table == table) {
                        // If we passed a tombstone and that slot still
                        // contains a tombstone...
                        if (firstTombstone > -1
                                && table[firstTombstone] == TOMBSTONE) {
                            table[firstTombstone] = key.reference;
                            table[firstTombstone + 1] = value;
                            tombstones--;
                            size++;

                            // No need to clean up here. We aren't filling
                            // in a null slot.
                            return value;
                        }

                        // If this slot is still empty...
                        if (table[index] == null) {
                            table[index] = key.reference;
                            table[index + 1] = value;
                            size++;

                            cleanUp();
                            return value;
                        }
                    }

                    // The table changed during initialValue().
                    put(key, value);
                    return value;
                }

                if (firstTombstone == -1 && reference == TOMBSTONE) {
                    // Keep track of this tombstone so we can overwrite it.
                    firstTombstone = index;
                }
            }
        }
```

**void remove(ThreadLocal**      

```
void remove(ThreadLocal<?> key) {
    //先把table清理一下
    cleanUp();

    for (int index = key.hash & mask;; index = next(index)) {
        Object reference = table[index];
        //把那个引用的用TOMBSTONE占用
        if (reference == key.reference) {
            // Success!
            table[index] = TOMBSTONE;
            table[index + 1] = null;
            tombstones++;
            size--;
            return;
        }

        if (reference == null) {
            // No entry found.
            return;
        }
    }
}
```

**结语**

分析到这里，整个ThreadLocal的源码就分析的差不多了。在这里我们简单的总结一下这个类：

- 这个类之所以能够存储每个thread的信息，是因为它的内部有一个Values内部类，而Values中有一个Object数组。
- Object数组是以一种近似于map的形式来存储数据的，其中偶数位存ThreadLocal的弱引用，它的下一位存值。
- 在寻址的时候，Values采用了一种很神奇的方式——斐波拉契散列寻址
- Values里面的getAfterMiss()方法让人觉得很奇怪


**参考链接**     

[由浅入深全面剖析ThreadLocal - lypeer的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/luoyanglizi/article/details/51510233)


### Android消息机制与ThreadLocal 

Android的消息机制主要是指Handler的运行机制，Handler的运行需要底层的MessageQueue和Looper的支撑。MessageQueue的中文翻译是消息队列，顾名思义它的内部存储了一组消息，其以队列的形式对外提供插入和删除的工作，虽然叫做消息队列，但是它的内部存储结构并不是真正的队列，而是采用单链表的数据结构来存储消息列表。Looper的中文翻译为循环，在这里可以理解为消息循环，由于MessageQueue只是一个消息的存储单元，它不能去处理消息，而Looper就填补了这个功能，Looper会以无限循环的形式去查找是否有新消息，如果有的话就处理消息，否则就一直等待着


Looper中还有一个特殊的概念，那就是ThreadLocal，ThreadLocal并不是线程，它的作用是可以在每个线程中存储数据。大家知道，Handler创建的时候会采用当前线程的Looper来构造消息循环系统，那么Handler内部如何获取到当前线程的Looper呢？这就要使用ThreadLocal了，ThreadLocal可以在不同的线程之中互不干扰地存储并提供数据，通过ThreadLocal可以轻松获取每个线程的Looper。当然需要注意的是，线程是默认没有Looper的，如果需要使用Handler就必须为线程创建Looper。大家经常提到的主线程，也叫UI线程，它就是ActivityThread，ActivityThread被创建时就会初始化Looper，这也是在主线程中默认可以使用Handler的原因。

参考：[Android的消息机制之ThreadLocal的工作原理 - 任玉刚 - 博客频道 - CSDN.NET](http://blog.csdn.net/singwhatiwanna/article/details/48350919)

