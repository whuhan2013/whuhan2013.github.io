---
layout: post
title: Android实现支持缩放平移图片
date: 2016-5-18
categories: blog
tags: [自定义控件]
description: Android实现支持缩放平移图片
---   

### 本文主要用到了以下知识点  

1. Matrix
2. GestureDetector 能够捕捉到长按、双击
3. ScaleGestureDetector 用于检测缩放的手势

### 自由的缩放


需求：当图片加载时，将图片在屏幕中居中；图片宽或高大于屏幕的，缩小至屏幕大小；自由对图片进行方法或缩小；


```
package com.zhy.view;  
  
import android.content.Context;  
import android.graphics.Matrix;  
import android.graphics.drawable.Drawable;  
import android.util.AttributeSet;  
import android.util.Log;  
import android.view.MotionEvent;  
import android.view.ScaleGestureDetector;  
import android.view.ScaleGestureDetector.OnScaleGestureListener;  
import android.view.View;  
import android.view.View.OnTouchListener;  
import android.view.ViewTreeObserver;  
import android.widget.ImageView;  
  
public class ZoomImageView extends ImageView implements OnScaleGestureListener,  
        OnTouchListener, ViewTreeObserver.OnGlobalLayoutListener  
  
{  
    private static final String TAG = ZoomImageView.class.getSimpleName();  
      
    public static final float SCALE_MAX = 4.0f;  
    /** 
     * 初始化时的缩放比例，如果图片宽或高大于屏幕，此值将小于0 
     */  
    private float initScale = 1.0f;  
  
    /** 
     * 用于存放矩阵的9个值 
     */  
    private final float[] matrixValues = new float[9];  
  
    private boolean once = true;  
  
    /** 
     * 缩放的手势检测 
     */  
    private ScaleGestureDetector mScaleGestureDetector = null;  
  
    private final Matrix mScaleMatrix = new Matrix();  
  
    public ZoomImageView(Context context)  
    {  
        this(context, null);  
    }  
  
    public ZoomImageView(Context context, AttributeSet attrs)  
    {  
        super(context, attrs);  
        super.setScaleType(ScaleType.MATRIX);  
        mScaleGestureDetector = new ScaleGestureDetector(context, this);  
        this.setOnTouchListener(this);  
    }  
  
    @Override  
    public boolean onScale(ScaleGestureDetector detector)  
    {  
        float scale = getScale();  
        float scaleFactor = detector.getScaleFactor();  
  
        if (getDrawable() == null)  
            return true;  
  
        /** 
         * 缩放的范围控制 
         */  
        if ((scale < SCALE_MAX && scaleFactor > 1.0f)  
                || (scale > initScale && scaleFactor < 1.0f))  
        {  
            /** 
             * 最大值最小值判断 
             */  
            if (scaleFactor * scale < initScale)  
            {  
                scaleFactor = initScale / scale;  
            }  
            if (scaleFactor * scale > SCALE_MAX)  
            {  
                scaleFactor = SCALE_MAX / scale;  
            }  
            /** 
             * 设置缩放比例 
             */  
            mScaleMatrix.postScale(scaleFactor, scaleFactor, getWidth() / 2,  
                    getHeight() / 2);  
            setImageMatrix(mScaleMatrix);  
        }  
        return true;  
  
    }  
  
    @Override  
    public boolean onScaleBegin(ScaleGestureDetector detector)  
    {  
        return true;  
    }  
  
    @Override  
    public void onScaleEnd(ScaleGestureDetector detector)  
    {  
    }  
  
    @Override  
    public boolean onTouch(View v, MotionEvent event)  
    {  
        return mScaleGestureDetector.onTouchEvent(event);  
  
    }  
  
      
    /** 
     * 获得当前的缩放比例 
     *  
     * @return 
     */  
    public final float getScale()  
    {  
        mScaleMatrix.getValues(matrixValues);  
        return matrixValues[Matrix.MSCALE_X];  
    }  
  
    @Override  
    protected void onAttachedToWindow()  
    {  
        super.onAttachedToWindow();  
        getViewTreeObserver().addOnGlobalLayoutListener(this);  
    }  
  
    @SuppressWarnings("deprecation")  
    @Override  
    protected void onDetachedFromWindow()  
    {  
        super.onDetachedFromWindow();  
        getViewTreeObserver().removeGlobalOnLayoutListener(this);  
    }  
  
    @Override  
    public void onGlobalLayout()  
    {  
        if (once)  
        {  
            Drawable d = getDrawable();  
            if (d == null)  
                return;  
            Log.e(TAG, d.getIntrinsicWidth() + " , " + d.getIntrinsicHeight());  
            int width = getWidth();  
            int height = getHeight();  
            // 拿到图片的宽和高  
            int dw = d.getIntrinsicWidth();  
            int dh = d.getIntrinsicHeight();  
            float scale = 1.0f;  
            // 如果图片的宽或者高大于屏幕，则缩放至屏幕的宽或者高  
            if (dw > width && dh <= height)  
            {  
                scale = width * 1.0f / dw;  
            }  
            if (dh > height && dw <= width)  
            {  
                scale = height * 1.0f / dh;  
            }  
            // 如果宽和高都大于屏幕，则让其按按比例适应屏幕大小  
            if (dw > width && dh > height)  
            {  
                scale = Math.min(dw * 1.0f / width, dh * 1.0f / height);  
            }  
            initScale = scale;  
            // 图片移动至屏幕中心  
                        mScaleMatrix.postTranslate((width - dw) / 2, (height - dh) / 2);  
            mScaleMatrix  
                    .postScale(scale, scale, getWidth() / 2, getHeight() / 2);  
            setImageMatrix(mScaleMatrix);  
            once = false;  
        }  
  
    }  
  
}  
```


1、我们在onGlobalLayout的回调中，根据图片的宽和高以及屏幕的宽和高，对图片进行缩放以及移动至屏幕的中心。如果图片很小，那就正常显示，不放大了~                   
2、我们让OnTouchListener的MotionEvent交给ScaleGestureDetector进行处理          

```
@Override  
    public boolean onTouch(View v, MotionEvent event)  
    {  
        return mScaleGestureDetector.onTouchEvent(event);  
  
    }  
```


3、在onScale的回调中对图片进行缩放的控制，首先进行缩放范围的判断，然后设置mScaleMatrix的scale值
现在的效果：


![](http://img.blog.csdn.net/20140922151414046)



### 存在问题：
1、缩放的中心点，我们设置是固定的，屏幕中间                
2、放大后，无法移动~         


### 设置缩放中心

1、单纯的设置缩放中心             
仅仅是设置中心很简单，直接修改下中心点 ：

```
/** 
             * 设置缩放比例 
             */  
            mScaleMatrix.postScale(scaleFactor, scaleFactor,  
                    detector.getFocusX(), detector.getFocusX());  
            setImageMatrix(mScaleMatrix);  
```


但是，随意的中心点放大、缩小，会导致图片的位置的变化，最终导致，图片宽高大于屏幕时，图片与屏幕间出现白边；图片小于屏幕，但是不居中。

2、控制缩放时图片显示的范围
所以我们在缩放的时候需要手动控制下范围：


```
/** 
     * 在缩放时，进行图片显示范围的控制 
     */  
    private void checkBorderAndCenterWhenScale()  
    {  
  
        RectF rect = getMatrixRectF();  
        float deltaX = 0;  
        float deltaY = 0;  
  
        int width = getWidth();  
        int height = getHeight();  
  
        // 如果宽或高大于屏幕，则控制范围  
        if (rect.width() >= width)  
        {  
            if (rect.left > 0)  
            {  
                deltaX = -rect.left;  
            }  
            if (rect.right < width)  
            {  
                deltaX = width - rect.right;  
            }  
        }  
        if (rect.height() >= height)  
        {  
            if (rect.top > 0)  
            {  
                deltaY = -rect.top;  
            }  
            if (rect.bottom < height)  
            {  
                deltaY = height - rect.bottom;  
            }  
        }  
        // 如果宽或高小于屏幕，则让其居中  
        if (rect.width() < width)  
        {  
            deltaX = width * 0.5f - rect.right + 0.5f * rect.width();  
        }  
        if (rect.height() < height)  
        {  
            deltaY = height * 0.5f - rect.bottom + 0.5f * rect.height();  
        }  
        Log.e(TAG, "deltaX = " + deltaX + " , deltaY = " + deltaY);  
  
        mScaleMatrix.postTranslate(deltaX, deltaY);  
  
    }  
  
    /** 
     * 根据当前图片的Matrix获得图片的范围 
     *  
     * @return 
     */  
    private RectF getMatrixRectF()  
    {  
        Matrix matrix = mScaleMatrix;  
        RectF rect = new RectF();  
        Drawable d = getDrawable();  
        if (null != d)  
        {  
            rect.set(0, 0, d.getIntrinsicWidth(), d.getIntrinsicHeight());  
            matrix.mapRect(rect);  
        }  
        return rect;  
    }  
```


### 自由的进行移动  

```
@Override  
    public boolean onTouch(View v, MotionEvent event)  
    {  
        mScaleGestureDetector.onTouchEvent(event);  
  
        float x = 0, y = 0;  
        // 拿到触摸点的个数  
        final int pointerCount = event.getPointerCount();  
        // 得到多个触摸点的x与y均值  
        for (int i = 0; i < pointerCount; i++)  
        {  
            x += event.getX(i);  
            y += event.getY(i);  
        }  
        x = x / pointerCount;  
        y = y / pointerCount;  
  
        /** 
         * 每当触摸点发生变化时，重置mLasX , mLastY  
         */  
        if (pointerCount != lastPointerCount)  
        {  
            isCanDrag = false;  
            mLastX = x;  
            mLastY = y;  
        }  
          
  
        lastPointerCount = pointerCount;  
  
        switch (event.getAction())  
        {  
        case MotionEvent.ACTION_MOVE:  
            Log.e(TAG, "ACTION_MOVE");  
            float dx = x - mLastX;  
            float dy = y - mLastY;  
              
            if (!isCanDrag)  
            {  
                isCanDrag = isCanDrag(dx, dy);  
            }  
            if (isCanDrag)  
            {  
                RectF rectF = getMatrixRectF();  
                if (getDrawable() != null)  
                {  
                    isCheckLeftAndRight = isCheckTopAndBottom = true;  
                    // 如果宽度小于屏幕宽度，则禁止左右移动  
                    if (rectF.width() < getWidth())  
                    {  
                        dx = 0;  
                        isCheckLeftAndRight = false;  
                    }  
                    // 如果高度小雨屏幕高度，则禁止上下移动  
                    if (rectF.height() < getHeight())  
                    {  
                        dy = 0;  
                        isCheckTopAndBottom = false;  
                    }  
                    mScaleMatrix.postTranslate(dx, dy);  
                    checkMatrixBounds();  
                    setImageMatrix(mScaleMatrix);  
                }  
            }  
            mLastX = x;  
            mLastY = y;  
            break;  
  
        case MotionEvent.ACTION_UP:  
        case MotionEvent.ACTION_CANCEL:  
            Log.e(TAG, "ACTION_UP");  
            lastPointerCount = 0;  
            break;  
        }  
  
        return true;  
    }  
```


首先我们拿到触摸点的数量，然后求出多个触摸点的平均值，设置给我们的mLastX , mLastY ， 然后在移动的时候，得到dx ,dy 进行范围检查以后，调用mScaleMatrix.postTranslate进行设置偏移量，当然了，设置完成以后，还需要再次校验一下，不能把图片移动的与屏幕边界出现白边，校验完成后，调用setImageMatrix.

这里：需要注意一下，我们没有复写ACTION_DOWM，是因为，ACTION_DOWN在多点触控的情况下，只要有一个手指按下状态，其他手指按下不会再次触发ACTION_DOWN，但是多个手指以后，触摸点的平均值会发生很大变化，所以我们没有用到ACTION_DOWN。每当触摸点的数量变化，我们就会跟新当前的mLastX,mLastY.
下面是上面用到的两个私有方法，一个用于检查边界，一个用于判断是否是拖动的操作：

```
/** 
     * 移动时，进行边界判断，主要判断宽或高大于屏幕的 
     */  
    private void checkMatrixBounds()  
    {  
        RectF rect = getMatrixRectF();  
  
        float deltaX = 0, deltaY = 0;  
        final float viewWidth = getWidth();  
        final float viewHeight = getHeight();  
        // 判断移动或缩放后，图片显示是否超出屏幕边界  
        if (rect.top > 0 && isCheckTopAndBottom)  
        {  
            deltaY = -rect.top;  
        }  
        if (rect.bottom < viewHeight && isCheckTopAndBottom)  
        {  
            deltaY = viewHeight - rect.bottom;  
        }  
        if (rect.left > 0 && isCheckLeftAndRight)  
        {  
            deltaX = -rect.left;  
        }  
        if (rect.right < viewWidth && isCheckLeftAndRight)  
        {  
            deltaX = viewWidth - rect.right;  
        }  
        mScaleMatrix.postTranslate(deltaX, deltaY);  
    }  
  
    /** 
     * 是否是推动行为 
     *  
     * @param dx 
     * @param dy 
     * @return 
     */  
    private boolean isCanDrag(float dx, float dy)  
    {  
        return Math.sqrt((dx * dx) + (dy * dy)) >= mTouchSlop;  
    }  
```


这样，我们就可以快乐的放大、缩小加移动了~~~

![](http://img.blog.csdn.net/20140922222445644)





### 双击放大与缩小

谈到双击事件，我们的GestureDetector终于要登场了，这哥们可以捕获双击事件~~                           
1、GestureDetector的使用                             
因为GestureDetector设置监听器的话，方法一大串，而我们只需要onDoubleTap这个回调，所以我们准备使用它的一个内部类SimpleOnGestureListener，对接口的其他方法实现了空实现。
不过还有几个问题需要讨论下，才能开始我们的代码：

1、我们双击尺寸如何变化？                  
我是这样的，根据当前的缩放值，如果是小于2的，我们双击直接到变为原图的2倍；如果是2,4之间的，我们双击直接为原图的4倍；其他状态也就是4倍，双击后还原到最初的尺寸。                           
如果你觉得这样不合适，可以根据自己的爱好调整。

2、我们双击变化，需要一个动画~~比如我们上例的演示图，图片很大，全屏显示的时候initScale=0.5左后，如果双击后变为2，也就是瞬间大了四倍，没有一个过渡的效果的话，给用户的感觉会特别差。所以，我们准备使用postDelay执行一个Runnable，Runnable中再次根据的当然的缩放值继续执行。                    
首先我们在构造方法中，完成对GestureDetector的初始化，以及设置onDoubleTap监听                  


```
public ZoomImageView(Context context, AttributeSet attrs)  
    {  
        super(context, attrs);  
        mScaleGestureDetector = new ScaleGestureDetector(context, this);  
        mGestureDetector = new GestureDetector(context,  
                new SimpleOnGestureListener()  
                {  
                    @Override  
                    public boolean onDoubleTap(MotionEvent e)  
                    {  
                        if (isAutoScale == true)  
                            return true;  
  
                        float x = e.getX();  
                        float y = e.getY();  
                        Log.e("DoubleTap", getScale() + " , " + initScale);  
                        if (getScale() < SCALE_MID)  
                        {  
                            ZoomImageView.this.postDelayed(  
                                    new AutoScaleRunnable(SCALE_MID, x, y), 16);  
                            isAutoScale = true;  
                        } else if (getScale() >= SCALE_MID  
                                && getScale() < SCALE_MAX)  
                        {  
                            ZoomImageView.this.postDelayed(  
                                    new AutoScaleRunnable(SCALE_MAX, x, y), 16);  
                            isAutoScale = true;  
                        } else  
                        {  
                            ZoomImageView.this.postDelayed(  
                                    new AutoScaleRunnable(initScale, x, y), 16);  
                            isAutoScale = true;  
                        }  
  
                        return true;  
                    }  
                });  
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();  
        super.setScaleType(ScaleType.MATRIX);  
        this.setOnTouchListener(this);  
    }  
```

1、当双击的时候，首先判断是否正在自动缩放，如果在，直接retrun ;   
2、然后就进入了我们的if，如果当然是scale小于2，则通过view.发送一个Runnable进行执行；其他类似；                    
下面看我们的Runnable的代码： 


```
/** 
     * 自动缩放的任务 
     *  
     * @author zhy 
     *  
     */  
    private class AutoScaleRunnable implements Runnable  
    {  
        static final float BIGGER = 1.07f;  
        static final float SMALLER = 0.93f;  
        private float mTargetScale;  
        private float tmpScale;  
  
        /** 
         * 缩放的中心 
         */  
        private float x;  
        private float y;  
  
        /** 
         * 传入目标缩放值，根据目标值与当前值，判断应该放大还是缩小 
         *  
         * @param targetScale 
         */  
        public AutoScaleRunnable(float targetScale, float x, float y)  
        {  
            this.mTargetScale = targetScale;  
            this.x = x;  
            this.y = y;  
            if (getScale() < mTargetScale)  
            {  
                tmpScale = BIGGER;  
            } else  
            {  
                tmpScale = SMALLER;  
            }  
  
        }  
  
        @Override  
        public void run()  
        {  
            // 进行缩放  
            mScaleMatrix.postScale(tmpScale, tmpScale, x, y);  
            checkBorderAndCenterWhenScale();  
            setImageMatrix(mScaleMatrix);  
  
            final float currentScale = getScale();  
            //如果值在合法范围内，继续缩放  
            if (((tmpScale > 1f) && (currentScale < mTargetScale))  
                    || ((tmpScale < 1f) && (mTargetScale < currentScale)))  
            {  
                ZoomImageView.this.postDelayed(this, 16);  
            } else//设置为目标的缩放比例  
            {  
                final float deltaScale = mTargetScale / currentScale;  
                mScaleMatrix.postScale(deltaScale, deltaScale, x, y);  
                checkBorderAndCenterWhenScale();  
                setImageMatrix(mScaleMatrix);  
                isAutoScale = false;  
            }  
  
        }  
    }  
```

代码写完了，我们依然需要把我们的event传给它，依然是在onTouch方法：

```
@Override  
    public boolean onTouch(View v, MotionEvent event)  
    {  
        if (mGestureDetector.onTouchEvent(event))  
            return true;  
```


![](http://img.blog.csdn.net/20140922233209467)




### 与ViewPager一起使用时的事件冲突问题

现在我们迅速的想一想，记得之前学习过事件分发机制，我们的ZoomImageView在ViewPager中，如果我们不想被拦截，那么如何做呢？              
首先不想被拦截的条件是：我们的宽或高大于屏幕宽或高时，因为此时可以移动，我们不想被拦截。接下来，不想被拦截：               
getParent().requestDisallowInterceptTouchEvent(true);                     
一行代码足以  


```
switch (event.getAction())  
        {  
        case MotionEvent.ACTION_DOWN:  
            if (rectF.width() > getWidth() || rectF.height() > getHeight())  
            {  
                getParent().requestDisallowInterceptTouchEvent(true);  
            }  
            break;  
        case MotionEvent.ACTION_MOVE:  
            if (rectF.width() > getWidth() || rectF.height() > getHeight())  
            {  
                getParent().requestDisallowInterceptTouchEvent(true);  
            }  
```


### requestDisallowInterceptTouchEvent 

ViewPager来实现左右滑动切换tab，如果tab的某一项中嵌入了水平可滑动的View就会让你有些不爽，比如想滑动tab项中的可水平滑动的控件，却导致tab切换。
 
因为Android事件机制是从父View传向子View的，可以去检测你当前子View是不是在有可滑动控件等，决定事件是否拦截，但是这个麻烦，而且并不能解决所有的问题（必须检测触摸点是否在这个控件上面），其实有比较简单的方法，在你嵌套的控件中注入ViewPager实例（调用控件的getParent()方法），然后在onTouchEvent，onInterceptTouchEvent，dispatchTouchEvent里面告诉父View，也就是ViewPager不要拦截该控件上的触摸事件。


### 参考链接

[requestDisallowInterceptTouchEvent - 喜糖 - 博客园](http://www.cnblogs.com/xitang/archive/2013/06/22/3150380.html)

### postDelayed方法

```
这是一种可以创建多线程消息的函数  
使用方法：  
1，首先创建一个Handler对象  
Handler handler=new Handler();  
2，然后创建一个Runnable对象  
Runnable runnable=new Runnable(){  
   @Override  
   public void run() {  
    // TODO Auto-generated method stub  
    //要做的事情，这里再次调用此Runnable对象，以实现每两秒实现一次的定时器操作  
    handler.postDelayed(this, 2000);  
   }   
};  
3，使用PostDelayed方法，两秒后调用此Runnable对象  
handler.postDelayed(runnable, 2000);  
实际上也就实现了一个2s的一个定时器  
4，如果想要关闭此定时器，可以这样操作  
handler.removeCallbacks(runnable);  
  
当然，你也可以做一个闹钟提醒延时的函数试试，比如，先用MediaPlayer播放闹钟声音，  
如果不想起，被停止播放之后，下次就5分钟后再播放，再被停止的话，下次就4分钟后播放，  
………………  
只要更改延时的时间就可以实现了，用一个static对象的话会比较容易操作。  
  
  
全手打原创哦，百度能告诉你的我就不告诉你了。 
```


### 参考链接

[关于 android 中 postDelayed方法的讲解 - - ITeye技术网站](http://xym-love.iteye.com/blog/1643263)

### 源代码

[放大功能](http://download.csdn.net/detail/lmj623565791/7960495)

[单图版源码点击下载](http://download.csdn.net/detail/lmj623565791/7960501)

[ViewPager版源码下载](http://download.csdn.net/detail/lmj623565791/7960505)


### 本文主要参考 

[Android 手势检测实战 打造支持缩放平移的图片预览效果（上） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/39474553)

[Android 手势检测实战 打造支持缩放平移的图片预览效果（下） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/39480503)

