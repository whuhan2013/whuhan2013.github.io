---
layout: post
title: Android之事件分发机制
date: 2016-5-24
categories: blog
tags: [android]
description: Android之事件分发机制
---   

### 本文主要包括以下内容

1. view的事件分发
2. viewGroup的事件分发


### 首先来看两张图  

![](http://img.blog.csdn.net/20160328204531512)

![](http://img.blog.csdn.net/20160328204550137)


在执行touch事件时

1. 首先执行dispatchTouchEvent方法，执行事件分发。

2. 再执行onInterceptTouchEvent方法，判断是否中断事件，返回true时中断，执行自己的onTouchEvnet方法.

3. 最后执行onTouchEvent方法，处理事件

呈U字型，拦截时从父View到子View,onTouchEvent时，则是从下到上。

### View的事件分发

不管是DOWN，MOVE，UP都会按照下面的顺序执行：       
1、dispatchTouchEvent                          
2、 setOnTouchListener的onTouch                   
3、onTouchEvent              


### 其中  

如果我们设置了setOnTouchListener，并且return true，那么View自己的onTouchEvent就不会被执行了


### 总结

1、整个View的事件转发流程是：
View.dispatchEvent->View.setOnTouchListener->View.onTouchEvent

在dispatchTouchEvent中会进行OnTouchListener的判断，如果OnTouchListener不为null且返回true，则表示事件被消费，onTouchEvent不会被执行；否则执行onTouchEvent。

2、onTouchEvent中的DOWN,MOVE,UP     

- DOWN时：        
a、首先设置标志为PREPRESSED，设置mHasPerformedLongPress=false ;然后发出一个115ms后的mPendingCheckForTap；              
b、如果115ms内没有触发UP，则将标志置为PRESSED，清除PREPRESSED标志，同时发出一个延时为500-115ms的，检测长按任务消息；                      
c、如果500ms内（从DOWN触发开始算），则会触发LongClickListener:            

此时如果LongClickListener不为null，则会执行回调，同时如果LongClickListener.onClick返回true，才把mHasPerformedLongPress设置为true;否则mHasPerformedLongPress依然为false;

- MOVE时：
主要就是检测用户是否划出控件，如果划出了：              
115ms内，直接移除mPendingCheckForTap；           
115ms后，则将标志中的PRESSED去除，同时移除长按的检查：removeLongPressCallback();    

- UP时：        
a、如果115ms内，触发UP，此时标志为PREPRESSED，则执行UnsetPressedState，setPressed(false);会把setPress转发下去，可以在View中复写dispatchSetPressed方法接收；                
b、如果是115ms-500ms间，即长按还未发生，则首先移除长按检测，执行onClick回调；                
c、如果是500ms以后，那么有两种情况：                  
i.设置了onLongClickListener，且onLongClickListener.onClick返回true，则点击事件OnClick事件无法触发；             
ii.没有设置onLongClickListener或者onLongClickListener.onClick返回false，则点击事件OnClick事件依然可以触发；                
d、最后执行mUnsetPressedState.run()，将setPressed传递下去，然后将PRESSED标识去除；                    

最后问个问题，然后再运行个例子结束：
1、setOnLongClickListener和setOnClickListener是否只能执行一个
不是的，只要setOnLongClickListener中的onClick返回false，则两个都会执行；返回true则会屏幕setOnClickListener          



###  Android ViewGroup事件分发机制


大体的事件流程为：              
MyLinearLayout的dispatchTouchEvent -> MyLinearLayout的onInterceptTouchEvent -> MyButton的dispatchTouchEvent ->Mybutton的onTouchEvent 

可以看出，在View上触发事件，最先捕获到事件的为View所在的ViewGroup，然后才会到View自身~


1、ACTION_DOWN中，ViewGroup捕获到事件，然后判断是否拦截，如果没有拦截，则找到包含当前x,y坐标的子View，赋值给mMotionTarget，然后调用  mMotionTarget.dispatchTouchEvent

2、ACTION_MOVE中，ViewGroup捕获到事件，然后判断是否拦截，如果没有拦截，则直接调用mMotionTarget.dispatchTouchEvent(ev)

3、ACTION_UP中，ViewGroup捕获到事件，然后判断是否拦截，如果没有拦截，则直接调用mMotionTarget.dispatchTouchEvent(ev)
当然了在分发之前都会修改下坐标系统，把当前的x，y分别减去child.left 和 child.top ，然后传给child;


### 关于拦截


### 如何拦截

复写ViewGroup的onInterceptTouchEvent方法：


```
@Override
    public boolean onInterceptTouchEvent(MotionEvent ev)
    {
        int action = ev.getAction();
        switch (action)
        {
        case MotionEvent.ACTION_DOWN:
            //如果你觉得需要拦截
            return true ; 
        case MotionEvent.ACTION_MOVE:
            //如果你觉得需要拦截
            return true ; 
        case MotionEvent.ACTION_UP:
            //如果你觉得需要拦截
            return true ; 
        }
        
        return false;
    }
```


默认是不拦截的，即返回false；如果你需要拦截，只要return true就行了，这要该事件就不会往子View传递了，并且如果你在DOWN retrun true ，则DOWN,MOVE,UP子View都不会捕获事件；如果你在MOVE return true , 则子View在MOVE和UP都不会捕获事件。
原因很简单，当onInterceptTouchEvent(ev) return true的时候，会把mMotionTarget 置为null ; 


### 如何不被拦截


如果ViewGroup的onInterceptTouchEvent(ev) 当ACTION_MOVE时return true ，即拦截了子View的MOVE以及UP事件；           
此时子View希望依然能够响应MOVE和UP时该咋办呢？            
Android给我们提供了一个方法：                requestDisallowInterceptTouchEvent(boolean) 用于设置是否允许拦截，我们在子View的dispatchTouchEvent中直接这么写：     


```
@Override
    public boolean dispatchTouchEvent(MotionEvent event)
    {
        getParent().requestDisallowInterceptTouchEvent(true);  
        int action = event.getAction();

        switch (action)
        {
        case MotionEvent.ACTION_DOWN:
            Log.e(TAG, "dispatchTouchEvent ACTION_DOWN");
            break;
        case MotionEvent.ACTION_MOVE:
            Log.e(TAG, "dispatchTouchEvent ACTION_MOVE");
            break;
        case MotionEvent.ACTION_UP:
            Log.e(TAG, "dispatchTouchEvent ACTION_UP");
            break;

        default:
            break;
        }
        return super.dispatchTouchEvent(event);
    }
```


getParent().requestDisallowInterceptTouchEvent(true);  这样即使ViewGroup在MOVE的时候return true，子View依然可以捕获到MOVE以及UP事件。

### 注
如果ViewGroup在onInterceptTouchEvent(ev)  ACTION_DOWN里面直接return true了，那么子View是木有办法的捕获事件的~~~


### 如果没有找到合适的子View


1、ACTION_DOWN的时候，子View.dispatchTouchEvent(ev)返回的为false ; 

则不处理，向上传递，由父view处理  

### 总结

关于代码流程上面已经总结过了~             
1、如果ViewGroup找到了能够处理该事件的View，则直接交给子View处理，自己的onTouchEvent不会被触发；      
2、可以通过复写onInterceptTouchEvent(ev)方法，拦截子View的事件（即return true），把事件交给自己处理，则会执行自己对应的onTouchEvent方法               
3、子View可以通过调用                 getParent().requestDisallowInterceptTouchEvent(true);  阻止ViewGroup对其MOVE或者UP事件进行拦截；          


好了，那么实际应用中能解决哪些问题呢？                
比如你需要写一个类似slidingmenu的左侧隐藏menu，主Activity上有个Button、ListView或者任何可以响应点击的View，你在当前View上死命的滑动，菜单栏也出不来；因为MOVE事件被子View处理了~ 你需要这么做：在ViewGroup的dispatchTouchEvent中判断用户是不是想显示菜单，如果是，则在onInterceptTouchEvent(ev)拦截子View的事件；自己进行处理，这样自己的onTouchEvent就可以顺利展现出菜单栏了~


### 参考链接

[Android View 事件分发机制 源码解析 （上） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/38960443)

[Android ViewGroup事件分发机制 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/39102591)



