---
layout: post
title: Android实现圆形圆角图片
date: 2016-5-23
categories: blog
tags: [自定义控件]
description: Android实现圆形圆角图片
---   

### 本文主要使用两种方法实现图形圆角图片  

1. 自定View加上使用Xfermode实现
2. Shader实现   

### 自定View加上使用Xfermode实现


```
/** 
     * 根据原图和变长绘制圆形图片 
     *  
     * @param source 
     * @param min 
     * @return 
     */  
    private Bitmap createCircleImage(Bitmap source, int min)  
    {  
        final Paint paint = new Paint();  
        paint.setAntiAlias(true);  
        Bitmap target = Bitmap.createBitmap(min, min, Config.ARGB_8888);  
        /** 
         * 产生一个同样大小的画布 
         */  
        Canvas canvas = new Canvas(target);  
        /** 
         * 首先绘制圆形 
         */  
        canvas.drawCircle(min / 2, min / 2, min / 2, paint);  
        /** 
         * 使用SRC_IN 
         */  
        paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));  
        /** 
         * 绘制图片 
         */  
        canvas.drawBitmap(source, 0, 0, paint);  
        return target;  
    }  
```

其实主要靠：

```
paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
```         
这行代码，为什么呢，我给大家解释下，SRC_IN这种模式，两个绘制的效果叠加后取交集展现后图，怎么说呢，咱们第一个绘制的是个圆形，第二个绘制的是个Bitmap，于是交集为圆形


![](http://img.blog.csdn.net/20140426213750328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


### 参考链接

[Android 完美实现图片圆角和圆形（对实现进行分析） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/24555655)



### 使用BitmapShader实现

### 效果图

![](http://img.blog.csdn.net/20141216230037920?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 浅谈BitmapShader

BitmapShader是Shader的子类，可以通过Paint.setShader（Shader shader）进行设置、                  
这里我们只关注BitmapShader，构造方法：          

```
mBitmapShader = new BitmapShader(bitmap, TileMode.CLAMP, TileMode.CLAMP);
```

参数1：bitmap              
参数2，参数3：TileMode；            

TileMode的取值有三种：           

- CLAMP 拉伸    
- REPEAT 重复
- MIRROR 镜像

如果大家给电脑屏幕设置屏保的时候，如果图片太小，可以选择重复、拉伸、镜像；          
重复：就是横向、纵向不断重复这个bitmap              
镜像：横向不断翻转重复，纵向不断翻转重复；     
拉伸：这个和电脑屏保的模式应该有些不同，这个拉伸的是图片最后的那一个像素；横向的最后一个横行像素，不断的重复，纵项的那一列像素，不断的重复；

现在大概明白了，BitmapShader通过设置给mPaint，然后用这个mPaint绘图时，就会根据你设置的TileMode，对绘制区域进行着色。
这里需要注意一点：就是BitmapShader是从你的画布的左上角开始绘制的，不在view的右下角绘制个正方形，它不会在你正方形的左上角开始。
好了，到此，我相信大家对BitmapShader有了一定的了解了；当然了，如果你希望对Shader充分的了解

### 参考

[自定义控件其实很简单1/3 - AigeStudio - 博客频道 - CSDN.NET](http://blog.csdn.net/aigestudio/article/details/41799811)


对于我们的圆角，以及圆形，我们设置的模式都是CLAMP ，但是你会不会会有一个疑问：            
view的宽或者高大于我们的bitmap宽或者高岂不是会拉伸？

嗯，我们会为BitmapShader设置一个matrix，去适当的放大或者缩小图片，不会让“ view的宽或者高大于我们的bitmap宽或者高 ”此条件成立的。

到此我们的原理基本介绍完毕了，拿到drawable转化为bitmap，然后直接初始化BitmapShader，画笔设置Shader，最后在onDraw里面进行画圆就行了。  


### BitmapShader实战

### 自定义属性

```
<?xml version="1.0" encoding="utf-8"?>  
<resources>  
  
    <attr name="borderRadius" format="dimension" />  
    <attr name="type">  
        <enum name="circle" value="0" />  
        <enum name="round" value="1" />  
    </attr>  
    
  
    <declare-styleable name="RoundImageView">  
        <attr name="borderRadius" />  
        <attr name="type" />  
    </declare-styleable>  
  
</resources>  
```


### 获取自定义属性


```
public class RoundImageView extends ImageView  
{  
  
    /** 
     * 图片的类型，圆形or圆角 
     */  
    private int type;  
    private static final int TYPE_CIRCLE = 0;  
    private static final int TYPE_ROUND = 1;  
  
    /** 
     * 圆角大小的默认值 
     */  
    private static final int BODER_RADIUS_DEFAULT = 10;  
    /** 
     * 圆角的大小 
     */  
    private int mBorderRadius;  
  
    /** 
     * 绘图的Paint 
     */  
    private Paint mBitmapPaint;  
    /** 
     * 圆角的半径 
     */  
    private int mRadius;  
    /** 
     * 3x3 矩阵，主要用于缩小放大 
     */  
    private Matrix mMatrix;  
    /** 
     * 渲染图像，使用图像为绘制图形着色 
     */  
    private BitmapShader mBitmapShader;  
    /** 
     * view的宽度 
     */  
    private int mWidth;  
    private RectF mRoundRect;  
  
    public RoundImageView(Context context, AttributeSet attrs)  
    {  
        super(context, attrs);  
        mMatrix = new Matrix();  
        mBitmapPaint = new Paint();  
        mBitmapPaint.setAntiAlias(true);  
  
        TypedArray a = context.obtainStyledAttributes(attrs,  
                R.styleable.RoundImageView);  
  
        mBorderRadius = a.getDimensionPixelSize(  
                R.styleable.RoundImageView_borderRadius, (int) TypedValue  
                        .applyDimension(TypedValue.COMPLEX_UNIT_DIP,  
                                BODER_RADIUS_DEFAULT, getResources()  
                                        .getDisplayMetrics()));// 默认为10dp  
        type = a.getInt(R.styleable.RoundImageView_type, TYPE_CIRCLE);// 默认为Circle  
  
        a.recycle();  
    }  

```


### onMeasure

```
@Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)  
    {  
        Log.e("TAG", "onMeasure");  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);  
  
        /** 
         * 如果类型是圆形，则强制改变view的宽高一致，以小值为准 
         */  
        if (type == TYPE_CIRCLE)  
        {  
            mWidth = Math.min(getMeasuredWidth(), getMeasuredHeight());  
            mRadius = mWidth / 2;  
            setMeasuredDimension(mWidth, mWidth);  
        }  
  
    }  
```


### 设置BitmapShader

```
/** 
     * 初始化BitmapShader 
     */  
    private void setUpShader()  
    {  
        Drawable drawable = getDrawable();  
        if (drawable == null)  
        {  
            return;  
        }  
  
        Bitmap bmp = drawableToBitamp(drawable);  
        // 将bmp作为着色器，就是在指定区域内绘制bmp  
        mBitmapShader = new BitmapShader(bmp, TileMode.CLAMP, TileMode.CLAMP);  
        float scale = 1.0f;  
        if (type == TYPE_CIRCLE)  
        {  
            // 拿到bitmap宽或高的小值  
            int bSize = Math.min(bmp.getWidth(), bmp.getHeight());  
            scale = mWidth * 1.0f / bSize;  
  
        } else if (type == TYPE_ROUND)  
        {  
            // 如果图片的宽或者高与view的宽高不匹配，计算出需要缩放的比例；缩放后的图片的宽高，一定要大于我们view的宽高；所以我们这里取大值；  
            scale = Math.max(getWidth() * 1.0f / bmp.getWidth(), getHeight()  
                    * 1.0f / bmp.getHeight());  
        }  
        // shader的变换矩阵，我们这里主要用于放大或者缩小  
        mMatrix.setScale(scale, scale);  
        // 设置变换矩阵  
        mBitmapShader.setLocalMatrix(mMatrix);  
        // 设置shader  
        mBitmapPaint.setShader(mBitmapShader);  
    }  
```


在setUpShader中，首先对drawable转化为我们的bitmap;
然后初始化

```
mBitmapShader = new BitmapShader(bmp, TileMode.CLAMP, TileMode.CLAMP);
```


接下来，根据类型以及bitmap和view的宽高，计算scale；
关于scale的计算：

圆形时：取bitmap的宽或者高的小值作为基准，如果采用大值，缩放后肯定不能填满我们的圆形区域。然后，view的mWidth/bSize ; 得到的就是scale。

圆角时：因为设计到宽/高比例，我们分别

```
getWidth() * 1.0f / bmp.getWidth() 和 getHeight() * 1.0f / bmp.getHeight() ；
```
最终取大值，因为我们要让最终缩放完成的图片一定要大于我们的view的区域，有点类似centerCrop；

比如：view的宽高为10*20；图片的宽高为5*100 ； 最终我们应该按照宽的比例放大，而不是按照高的比例缩小；因为我们需要让缩放后的图片，自定大于我们的view宽高，并保证原图比例。

有了scale，就可以设置给我们的matrix；           
然后使用mBitmapShader.setLocalMatrix(mMatrix);            
最后将bitmapShader设置给paint。                  
关于drawable转bitmap的代码： 


```
/** 
     * drawable转bitmap 
     *  
     * @param drawable 
     * @return 
     */  
    private Bitmap drawableToBitamp(Drawable drawable)  
    {  
        if (drawable instanceof BitmapDrawable)  
        {  
            BitmapDrawable bd = (BitmapDrawable) drawable;  
            return bd.getBitmap();  
        }  
        int w = drawable.getIntrinsicWidth();  
        int h = drawable.getIntrinsicHeight();  
        Bitmap bitmap = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);  
        Canvas canvas = new Canvas(bitmap);  
        drawable.setBounds(0, 0, w, h);  
        drawable.draw(canvas);  
        return bitmap;  
    }  
```


最后我们会在onDraw里面调用setUpShader（），然后进行绘制。


### 绘制


```
@Override  
    protected void onDraw(Canvas canvas)  
    {  
        if (getDrawable() == null)  
        {  
            return;  
        }  
        setUpShader();  
  
        if (type == TYPE_ROUND)  
        {  
            canvas.drawRoundRect(mRoundRect, mBorderRadius, mBorderRadius,  
                    mBitmapPaint);  
        } else  
        {  
            canvas.drawCircle(mRadius, mRadius, mRadius, mBitmapPaint);  
            // drawSomeThing(canvas);  
        }  
    }  
      
    @Override  
    protected void onSizeChanged(int w, int h, int oldw, int oldh)  
    {  
        super.onSizeChanged(w, h, oldw, oldh);  
        // 圆角图片的范围  
        if (type == TYPE_ROUND)  
            mRoundRect = new RectF(0, 0, getWidth(), getHeight());  
    }  
```


### 状态的存储与恢复


当然了，如果内存不足，而恰好我们的Activity置于后台，不幸被重启，或者用户旋转屏幕造成Activity重启，我们的View应该也能尽可能的去保存自己的属性。         
状态保存什么用处呢？比如，现在一个的圆角大小是10dp，用户点击后变成50dp；当用户旋转以后，或者长时间置于后台以后，返回我们的Activity应该还是50dp；                    
我们简单的存储一下，当前的type以及mBorderRadius  


```
private static final String STATE_INSTANCE = "state_instance";  
    private static final String STATE_TYPE = "state_type";  
    private static final String STATE_BORDER_RADIUS = "state_border_radius";  
  
    @Override  
    protected Parcelable onSaveInstanceState()  
    {  
        Bundle bundle = new Bundle();  
        bundle.putParcelable(STATE_INSTANCE, super.onSaveInstanceState());  
        bundle.putInt(STATE_TYPE, type);  
        bundle.putInt(STATE_BORDER_RADIUS, mBorderRadius);  
        return bundle;  
    }  
  
    @Override  
    protected void onRestoreInstanceState(Parcelable state)  
    {  
        if (state instanceof Bundle)  
        {  
            Bundle bundle = (Bundle) state;  
            super.onRestoreInstanceState(((Bundle) state)  
                    .getParcelable(STATE_INSTANCE));  
            this.type = bundle.getInt(STATE_TYPE);  
            this.mBorderRadius = bundle.getInt(STATE_BORDER_RADIUS);  
        } else  
        {  
            super.onRestoreInstanceState(state);  
        }  
  
    }  
```


### 参考链接

[Android BitmapShader 实战 实现圆形、圆角图片 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/41967509)


### 源代码

[源代码](http://download.csdn.net/detail/lmj623565791/8271979)


### 最终效果如下

![](http://img.blog.csdn.net/20141218011259585)