---
layout: post
title: Android中mesure过程详解
date: 2016-5-18
categories: blog
tags: [自定义控件]
description: Android中mesure过程详解
---   

### 本文主要包括以下内容

1. DisplayMetrics
2. measure过程 

### 利用DisplayMetrics获取屏幕信息   

```
// 获得屏幕宽度
        WindowManager wm = (WindowManager) context
                .getSystemService(Context.WINDOW_SERVICE);
        DisplayMetrics outMetrics = new DisplayMetrics();
        wm.getDefaultDisplay().getMetrics(outMetrics);
        mScreenWitdh = outMetrics.widthPixels;
```

### 参考链接

[Android 中的DisplayMetrics类的用法 - yujian_bing的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/yujian_bing/article/details/8264780)

我们在编写layout的xml文件时会碰到layout_width和layout_height两个属性，对于这两个属性我们有三种选择：赋值成具体的数值，match_parent或者wrap_content，而measure过程就是用来处理match_parent或者wrap_content，假如layout中规定所有View的layout_width和layout_height必须赋值成具体的数值，那么measure其实是没有必要的，但是google在设计Android的时候考虑加入match_parent或者wrap_content肯定是有原因的，它们会使得布局更加灵活。
 

首先我们来看几个关键的函数和参数：

``` 
      1、public final void measue(int widthMeasureSpec, int heightMeasureSpec);
 
      2、protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)；
 
      3、protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec)
 
      4、protected void measureChild(View child, int parentWidthMeasureSpec, int parentHeightMeasureSpec)
 
      5、protected void measureChildWithMargins(View child,int parentWidthMeasureSpec, int widthUsed, int parentHeightMeasureSpec, int heightUsed)
```

 
接着我们来看View类中measure和onMeasure函数的源码：


```
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        if ((mPrivateFlags & FORCE_LAYOUT) == FORCE_LAYOUT ||
                widthMeasureSpec != mOldWidthMeasureSpec ||
                heightMeasureSpec != mOldHeightMeasureSpec) {

            // first clears the measured dimension flag
            mPrivateFlags &= ~MEASURED_DIMENSION_SET;

            if (ViewDebug.TRACE_HIERARCHY) {
                ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_MEASURE);
            }

            // measure ourselves, this should set the measured dimension flag back
            onMeasure(widthMeasureSpec, heightMeasureSpec);

            // flag not set, setMeasuredDimension() was not invoked, we raise
            // an exception to warn the developer
            if ((mPrivateFlags & MEASURED_DIMENSION_SET) != MEASURED_DIMENSION_SET) {
                throw new IllegalStateException("onMeasure() did not set the"
                        + " measured dimension by calling"
                        + " setMeasuredDimension()");
            }

            mPrivateFlags |= LAYOUT_REQUIRED;
        }

        mOldWidthMeasureSpec = widthMeasureSpec;
        mOldHeightMeasureSpec = heightMeasureSpec;
    }
```


 由于函数原型中有final字段，那么measure根本没打算被子类继承，也就是说measure的过程是固定的，而measure中调用了onMeasure函数，因此真正有变数的是onMeasure函数，onMeasure的默认实现很简单，源码如下：

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```


onMeasure默认的实现仅仅调用了setMeasuredDimension，setMeasuredDimension函数是一个很关键的函数，它对View的成员变量mMeasuredWidth和mMeasuredHeight变量赋值，而measure的主要目的就是对View树中的每个View的mMeasuredWidth和mMeasuredHeight进行赋值，一旦这两个变量被赋值，则意味着该View的测量工作结束。


```
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
        mMeasuredWidth = measuredWidth;
        mMeasuredHeight = measuredHeight;

        mPrivateFlags |= MEASURED_DIMENSION_SET;
    }
```


对于非ViewGroup的View而言，通过调用上面默认的measure——>onMeasure，即可完成View的测量，当然你也可以重载onMeasure，并调用setMeasuredDimension来设置任意大小的布局，但一般不这么做，因为这种做法太“专政”，至于为何“专政”，读完本文就会明白。
 
  对于ViewGroup的子类而言，往往会重载onMeasure函数负责其children的measure工作，重载时不要忘记调用setMeasuredDimension来设置自身的mMeasuredWidth和mMeasuredHeight。如果我们在layout的时候不需要依赖子视图的大小，那么不重载onMeasure也可以，但是必须重载onLayout来安排子视图的位置，这在下一篇博客中会介绍。  
 
  再来看下measue(int widthMeasureSpec, int heightMeasureSpec)中的两个参数， 这两个参数分别是父视图提供的测量规格，当父视图调用子视图的measure函数对子视图进行测量时，会传入这两个参数，通过这两个参数以及子视图本身的LayoutParams来共同决定子视图的测量规格，在ViewGroup的measureChildWithMargins函数中体现了这个过程，稍后会介绍。
 
 MeasureSpec参数的值为int型，分为高32位和低16为，高32位保存的是specMode，低16位表示specSize，specMode分三种：
 
  1、MeasureSpec.UNSPECIFIED,父视图不对子视图施加任何限制，子视图可以得到任意想要的大小；
 
  2、MeasureSpec.EXACTLY，父视图希望子视图的大小是specSize中指定的大小；
 
  3、MeasureSpec.AT_MOST，子视图的大小最多是specSize中的大小。
 
  以上施加的限制只是父视图“希望”子视图的大小按MeasureSpec中描述的那样，但是子视图的具体大小取决于多方面的。
 
  ViewGroup中定义了measureChildren, measureChild,  measureChildWithMargins来对子视图进行测量，measureChildren内部只是循环调用measureChild，measureChild和measureChildWithMargins的区别就是是否把margin和padding也作为子视图的大小，我们主要分析measureChildWithMargins的执行过程：


```
 protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    } 
```

 
 
总的来看该函数就是对父视图提供的measureSpec参数进行了调整（结合自身的LayoutParams参数），然后再来调用child.measure()函数，具体通过函数getChildMeasureSpec来进行参数调整，过程如下：
 

```
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = 0;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = 0;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    } 
```

getChildMeasureSpec的总体思路就是通过其父视图提供的MeasureSpec参数得到specMode和specSize，并根据计算出来的specMode以及子视图的childDimension（layout_width和layout_height中定义的）来计算自身的measureSpec，如果其本身包含子视图，则计算出来的measureSpec将作为调用其子视图measure函数的参数，同时也作为自身调用setMeasuredDimension的参数，如果其不包含子视图则默认情况下最终会调用onMeasure的默认实现，并最终调用到setMeasuredDimension，而该函数的参数正是这里计算出来的。
 

1)不论父View 的specMode是EXACTLY、UNSPECIFIED、AT_MOST的哪一种，如果子view在xml文件里面把layout_width或者layout_height这是了具体
  的dimen值、或者在代码里调用相应的方法为view的宽度或者高度设置了具体的值，那么此时widthSpec的mode 或者heightSpec的mode就是EXACLTY,size就等于layout_width或者
  layout_height的值。此时子view的specMode值不受父View的specMode的影响。
  
2)当父View的specMode为EXACTLY的时候：父View强加给子View一个确切的大小,有如下两种情况
    2.1)子View的layout_width或者layout_height设置为MATCH_PARENT的时候，子View的specMode为EXACTLY             
    2.2)子View的layout_width或者layout_height设置为       WRAP_CONTENT的时候，子View的specMode为AT_MOST               
    2.3)不论子view为match_parent或者wrap_content，resultSize都等于父类的size.                
    
3)当父view的specMode为AT_MOST的时候：父View强加给一个最大的size给子view，最大的size也就是父view的size           
    3.1)此时不论子view的为match_parent或者wrap_content，子view的specMode都为AT_MOST                                      
    3.2)resultSize的大小被设置为父view的大小             
    
4)当父view的specMode为UNSPECIFIED的时候：            
    4.1) 此时不论子view的为match_parent或者wrap_content，子view的specMode都为UNSPECIFIED                                
    4.3)此时reusltSize = 0                                        
    
5)从对UNSPECIFIED分析可以知道,在1)情况之外的情况下，要是想让子view自己决定自己的宽度或者高度的大小，MeasureSpec.makeMeasureSpec(0,UNSPECIFIED)这么调用来获取宽度和高度
的spec

6)根据方法的参数以及方法的实现可以知道，视图的大小是由父视图和子视图共同决定的，前提是子视图没有设置具体的dimen值
 
 
 总结：从上面的描述看出，决定权最大的就是View的设计者，因为设计者可以通过调用setMeasuredDimension决定视图的最终大小，例如调用setMeasuredDimension（100， 100）将视图的mMeasuredWidth和mMeasuredHeight设置为100，100，那么父视图提供的大小以及程序员在xml中设置的layout_width和layout_height将完全不起作用，当然良好的设计一般会根据子视图的measureSpec来设置mMeasuredWidth和mMeasuredHeight的大小，已尊重程序员的意图。


### 通过以下方法可以强制计算当前view的宽高，原因不明

```
int w = View.MeasureSpec.makeMeasureSpec(0,
                    View.MeasureSpec.UNSPECIFIED);
            int h = View.MeasureSpec.makeMeasureSpec(0,
                    View.MeasureSpec.UNSPECIFIED);
            view.measure(w, h);
            mChildHeight = view.getMeasuredHeight();
            mChildWidth = view.getMeasuredWidth();
```


### 参考链接

[获取view的高度和宽度(在onCreate方法中) - Listening_music的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/listening_music/article/details/7210146)


### 本文主要参考 

[Android中mesure过程详解 -- - 那些人追过的年 - 博客园](http://www.cnblogs.com/xilinch/archive/2012/10/24/2737178.html)

[MeasureSpec学习 - 转 - yuhailong626的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/yuhailong626/article/details/20639217)

[MeasureSpec的简单说明 - 凌云 - 博客频道 - CSDN.NET](http://blog.csdn.net/chunqiuwei/article/details/44515345)

![](http://img.blog.csdn.net/20150321142254322?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2h1bnFpdXdlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)