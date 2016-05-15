---
layout: post
title: Android之自定义ViewGroup
date: 2016-5-14
categories: blog
tags: [自定义控件]
description: Android之自定义ViewGroup
---   

 
### 概述

在写代码之前，我必须得问几个问题： 

### 1、ViewGroup的职责是啥？

ViewGroup相当于一个放置View的容器，并且我们在写布局xml的时候，会告诉容器（凡是以layout为开头的属性，都是为用于告诉容器的），我们的宽度（layout_width）、高度（layout_height）、对齐方式（layout_gravity）等；当然还有margin等；于是乎，ViewGroup的职能为：给childView计算出建议的宽和高和测量模式 ；决定childView的位置；为什么只是建议的宽和高，而不是直接确定呢，别忘了childView宽和高可以设置为wrap_content，这样只有childView才能计算出自己的宽和高。

### 2、View的职责是啥？

View的职责，根据测量模式和ViewGroup给出的建议的宽和高，计算出自己的宽和高；同时还有个更重要的职责是：在ViewGroup为其指定的区域内绘制自己的形态。

### 3、ViewGroup和LayoutParams之间的关系？

大家可以回忆一下，当在LinearLayout中写childView的时候，可以写layout_gravity，layout_weight属性；在RelativeLayout中的childView有layout_centerInParent属性，却没有layout_gravity，layout_weight，这是为什么呢？这是因为每个ViewGroup需要指定一个LayoutParams，用于确定支持childView支持哪些属性，比如LinearLayout指定LinearLayout.LayoutParams等。如果大家去看LinearLayout的源码，会发现其内部定义了LinearLayout.LayoutParams，在此类中，你可以发现weight和gravity的身影。


### 2、View的3种测量模式

上面提到了ViewGroup会为childView指定测量模式，下面简单介绍下三种测量模式：
EXACTLY：表示设置了精确的值，一般当childView设置其宽、高为精确值、match_parent时，ViewGroup会将其设置为EXACTLY；          
AT_MOST：表示子布局被限制在一个最大值内，一般当childView设置其宽、高为wrap_content时，ViewGroup会将其设置为AT_MOST；     
UNSPECIFIED：表示子布局想要多大就多大，一般出现在AadapterView的item的heightMode中、ScrollView的childView的heightMode中；此种模式比较少见。           

注：上面的每一行都有一个一般，意思上述不是绝对的，对于childView的mode的设置还会和ViewGroup的测量mode有一定的关系；当然了，这是第一篇自定义ViewGroup，而且绝大部分情况都是上面的规则，所以为了通俗易懂，暂不深入讨论其他内容。


### 3、从API角度进行浅析
上面叙述了ViewGroup和View的职责，下面从API角度进行浅析。
View的根据ViewGroup传人的测量值和模式，对自己宽高进行确定（onMeasure中完成），然后在onDraw中完成对自己的绘制。
ViewGroup需要给View传入view的测量值和模式（onMeasure中完成），而且对于此ViewGroup的父布局，自己也需要在onMeasure中完成对自己宽和高的确定。此外，需要在onLayout中完成对其childView的位置的指定。

### 4、完整的例子

1、决定该ViewGroup的LayoutParams
对于我们这个例子，我们只需要ViewGroup能够支持margin即可，那么我们直接使用系统的MarginLayoutParams

```
@Override
    public ViewGroup.LayoutParams generateLayoutParams(AttributeSet attrs)
    {
        return new MarginLayoutParams(getContext(), attrs);
    }

    @Override
    protected ViewGroup.LayoutParams generateDefaultLayoutParams()
    {
        Log.e(TAG, "generateDefaultLayoutParams");
        return new MarginLayoutParams(LayoutParams.MATCH_PARENT,
                LayoutParams.MATCH_PARENT);
    }

    @Override
    protected ViewGroup.LayoutParams generateLayoutParams(
            ViewGroup.LayoutParams p)
    {
        Log.e(TAG, "generateLayoutParams p");
        return new MarginLayoutParams(p);
    }
```

重写父类的该方法，返回MarginLayoutParams的实例，这样就为我们的ViewGroup指定了其LayoutParams为MarginLayoutParams。


### onMeasure


在onMeasure中计算childView的测量值以及模式，以及设置自己的宽和高：

```
/** 
     * 计算所有ChildView的宽度和高度 然后根据ChildView的计算结果，设置自己的宽和高 
     */  
    @Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)  
    {  
        /** 
         * 获得此ViewGroup上级容器为其推荐的宽和高，以及计算模式 
         */  
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);  
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);  
        int sizeWidth = MeasureSpec.getSize(widthMeasureSpec);  
        int sizeHeight = MeasureSpec.getSize(heightMeasureSpec);  
  
  
        // 计算出所有的childView的宽和高  
        measureChildren(widthMeasureSpec, heightMeasureSpec);  
        /** 
         * 记录如果是wrap_content是设置的宽和高 
         */  
        int width = 0;  
        int height = 0;  
  
        int cCount = getChildCount();  
  
        int cWidth = 0;  
        int cHeight = 0;  
        MarginLayoutParams cParams = null;  
  
        // 用于计算左边两个childView的高度  
        int lHeight = 0;  
        // 用于计算右边两个childView的高度，最终高度取二者之间大值  
        int rHeight = 0;  
  
        // 用于计算上边两个childView的宽度  
        int tWidth = 0;  
        // 用于计算下面两个childiew的宽度，最终宽度取二者之间大值  
        int bWidth = 0;  
  
        /** 
         * 根据childView计算的出的宽和高，以及设置的margin计算容器的宽和高，主要用于容器是warp_content时 
         */  
        for (int i = 0; i < cCount; i++)  
        {  
            View childView = getChildAt(i);  
            cWidth = childView.getMeasuredWidth();  
            cHeight = childView.getMeasuredHeight();  
            cParams = (MarginLayoutParams) childView.getLayoutParams();  
  
            // 上面两个childView  
            if (i == 0 || i == 1)  
            {  
                tWidth += cWidth + cParams.leftMargin + cParams.rightMargin;  
            }  
  
            if (i == 2 || i == 3)  
            {  
                bWidth += cWidth + cParams.leftMargin + cParams.rightMargin;  
            }  
  
            if (i == 0 || i == 2)  
            {  
                lHeight += cHeight + cParams.topMargin + cParams.bottomMargin;  
            }  
  
            if (i == 1 || i == 3)  
            {  
                rHeight += cHeight + cParams.topMargin + cParams.bottomMargin;  
            }  
  
        }  
          
        width = Math.max(tWidth, bWidth);  
        height = Math.max(lHeight, rHeight);  
  
        /** 
         * 如果是wrap_content设置为我们计算的值 
         * 否则：直接设置为父容器计算的值 
         */  
        setMeasuredDimension((widthMode == MeasureSpec.EXACTLY) ? sizeWidth  
                : width, (heightMode == MeasureSpec.EXACTLY) ? sizeHeight  
                : height);  
    }  
```

0-14行，获取该ViewGroup父容器为其设置的计算模式和尺寸，大多情况下，只要不是wrap_content，父容器都能正确的计算其尺寸。所以我们自己需要计算如果设置为wrap_content时的宽和高，如何计算呢？那就是通过其childView的宽和高来进行计算。

17行，通过ViewGroup的measureChildren方法为其所有的孩子设置宽和高，此行执行完成后，childView的宽和高都已经正确的计算过了

43-71行，根据childView的宽和高，以及margin，计算ViewGroup在wrap_content时的宽和高。

80-82行，如果宽高属性值为wrap_content，则设置为43-71行中计算的值，否则为其父容器传入的宽和高。


### onLayout对其所有childView进行定位（设置childView的绘制区域）


```
// abstract method in viewgroup  
    @Override  
    protected void onLayout(boolean changed, int l, int t, int r, int b)  
    {  
        int cCount = getChildCount();  
        int cWidth = 0;  
        int cHeight = 0;  
        MarginLayoutParams cParams = null;  
        /** 
         * 遍历所有childView根据其宽和高，以及margin进行布局 
         */  
        for (int i = 0; i < cCount; i++)  
        {  
            View childView = getChildAt(i);  
            cWidth = childView.getMeasuredWidth();  
            cHeight = childView.getMeasuredHeight();  
            cParams = (MarginLayoutParams) childView.getLayoutParams();  
  
            int cl = 0, ct = 0, cr = 0, cb = 0;  
  
            switch (i)  
            {  
            case 0:  
                cl = cParams.leftMargin;  
                ct = cParams.topMargin;  
                break;  
            case 1:  
                cl = getWidth() - cWidth - cParams.leftMargin  
                        - cParams.rightMargin;  
                ct = cParams.topMargin;  
  
                break;  
            case 2:  
                cl = cParams.leftMargin;  
                ct = getHeight() - cHeight - cParams.bottomMargin;  
                break;  
            case 3:  
                cl = getWidth() - cWidth - cParams.leftMargin  
                        - cParams.rightMargin;  
                ct = getHeight() - cHeight - cParams.bottomMargin;  
                break;  
  
            }  
            cr = cl + cWidth;  
            cb = cHeight + ct;  
            childView.layout(cl, ct, cr, cb);  
        }  
  
    }  
```

### 参考链接

[Android 手把手教您自定义ViewGroup（一） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/38339817)

[教你搞定Android自定义ViewGroup - 简书](http://www.jianshu.com/p/138b98095778)

