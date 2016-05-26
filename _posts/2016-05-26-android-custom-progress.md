---
layout: post
title: Android自定义progressBar
date: 2016-5-26
categories: blog
tags: [自定义控件]
description: Android自定义progressBar
---

### 通过继承系统ProgressBar实现


### 效果图

![](http://img.blog.csdn.net/20150201165458433)

![](http://img.blog.csdn.net/20150201165415044)


### 实现  

### HorizontalProgressBarWithNumber


### 自定义属性

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="HorizontalProgressBarWithNumber">
        <attr name="progress_unreached_color" format="color" />
        <attr name="progress_reached_color" format="color" />
        <attr name="progress_reached_bar_height" format="dimension" />
        <attr name="progress_unreached_bar_height" format="dimension" />
        <attr name="progress_text_size" format="dimension" />
        <attr name="progress_text_color" format="color" />
        <attr name="progress_text_offset" format="dimension" />
        <attr name="progress_text_visibility" format="enum">
            <enum name="visible" value="0" />
            <enum name="invisible" value="1" />
        </attr>
    </declare-styleable>
    
    <declare-styleable name="RoundProgressBarWidthNumber">
        <attr name="radius" format="dimension" />
    </declare-styleable>

</resources>
```


### HorizontalProgressBarWithNumber


```
package com.zhy.view;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.util.Log;
import android.util.TypedValue;
import android.widget.ProgressBar;

import com.zhy.library.view.R;

public class HorizontalProgressBarWithNumber extends ProgressBar
{

    private static final int DEFAULT_TEXT_SIZE = 10;
    private static final int DEFAULT_TEXT_COLOR = 0XFFFC00D1;
    private static final int DEFAULT_COLOR_UNREACHED_COLOR = 0xFFd3d6da;
    private static final int DEFAULT_HEIGHT_REACHED_PROGRESS_BAR = 2;
    private static final int DEFAULT_HEIGHT_UNREACHED_PROGRESS_BAR = 2;
    private static final int DEFAULT_SIZE_TEXT_OFFSET = 10;

    /**
     * painter of all drawing things
     */
    protected Paint mPaint = new Paint();
    /**
     * color of progress number
     */
    protected int mTextColor = DEFAULT_TEXT_COLOR;
    /**
     * size of text (sp)
     */
    protected int mTextSize = sp2px(DEFAULT_TEXT_SIZE);

    /**
     * offset of draw progress
     */
    protected int mTextOffset = dp2px(DEFAULT_SIZE_TEXT_OFFSET);

    /**
     * height of reached progress bar
     */
    protected int mReachedProgressBarHeight = dp2px(DEFAULT_HEIGHT_REACHED_PROGRESS_BAR);

    /**
     * color of reached bar
     */
    protected int mReachedBarColor = DEFAULT_TEXT_COLOR;
    /**
     * color of unreached bar
     */
    protected int mUnReachedBarColor = DEFAULT_COLOR_UNREACHED_COLOR;
    /**
     * height of unreached progress bar
     */
    protected int mUnReachedProgressBarHeight = dp2px(DEFAULT_HEIGHT_UNREACHED_PROGRESS_BAR);
    /**
     * view width except padding
     */
    protected int mRealWidth;

    protected boolean mIfDrawText = true;

    protected static final int VISIBLE = 0;

    public HorizontalProgressBarWithNumber(Context context, AttributeSet attrs)
    {
        this(context, attrs, 0);
    }

    public HorizontalProgressBarWithNumber(Context context, AttributeSet attrs,
            int defStyle)
    {
        super(context, attrs, defStyle);
        obtainStyledAttributes(attrs);
        mPaint.setTextSize(mTextSize);
        mPaint.setColor(mTextColor);
    }

    @Override
    protected synchronized void onMeasure(int widthMeasureSpec,
            int heightMeasureSpec)
    {

        int width = MeasureSpec.getSize(widthMeasureSpec);
        int height = measureHeight(heightMeasureSpec);
        setMeasuredDimension(width, height);

        mRealWidth = getMeasuredWidth() - getPaddingRight() - getPaddingLeft();
    }

    private int measureHeight(int measureSpec)
    {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY)
        {
            result = specSize;
        } else
        {
            float textHeight = (mPaint.descent() - mPaint.ascent());
            result = (int) (getPaddingTop() + getPaddingBottom() + Math.max(
                    Math.max(mReachedProgressBarHeight,
                            mUnReachedProgressBarHeight), Math.abs(textHeight)));
            if (specMode == MeasureSpec.AT_MOST)
            {
                result = Math.min(result, specSize);
            }
        }
        return result;
    }

    /**
     * get the styled attributes
     * 
     * @param attrs
     */
    private void obtainStyledAttributes(AttributeSet attrs)
    {
        // init values from custom attributes
        final TypedArray attributes = getContext().obtainStyledAttributes(
                attrs, R.styleable.HorizontalProgressBarWithNumber);

        mTextColor = attributes
                .getColor(
                        R.styleable.HorizontalProgressBarWithNumber_progress_text_color,
                        DEFAULT_TEXT_COLOR);
        mTextSize = (int) attributes.getDimension(
                R.styleable.HorizontalProgressBarWithNumber_progress_text_size,
                mTextSize);

        mReachedBarColor = attributes
                .getColor(
                        R.styleable.HorizontalProgressBarWithNumber_progress_reached_color,
                        mTextColor);
        mUnReachedBarColor = attributes
                .getColor(
                        R.styleable.HorizontalProgressBarWithNumber_progress_unreached_color,
                        DEFAULT_COLOR_UNREACHED_COLOR);
        mReachedProgressBarHeight = (int) attributes
                .getDimension(
                        R.styleable.HorizontalProgressBarWithNumber_progress_reached_bar_height,
                        mReachedProgressBarHeight);
        mUnReachedProgressBarHeight = (int) attributes
                .getDimension(
                        R.styleable.HorizontalProgressBarWithNumber_progress_unreached_bar_height,
                        mUnReachedProgressBarHeight);
        mTextOffset = (int) attributes
                .getDimension(
                        R.styleable.HorizontalProgressBarWithNumber_progress_text_offset,
                        mTextOffset);

        int textVisible = attributes
                .getInt(R.styleable.HorizontalProgressBarWithNumber_progress_text_visibility,
                        VISIBLE);
        if (textVisible != VISIBLE)
        {
            mIfDrawText = false;
        }
        
        
        attributes.recycle();
    }

    @Override
    protected synchronized void onDraw(Canvas canvas)
    {

        canvas.save();
        canvas.translate(getPaddingLeft(), getHeight() / 2);

        boolean noNeedBg = false;
        float radio = getProgress() * 1.0f / getMax();
        float progressPosX = (int) (mRealWidth * radio);
        String text = getProgress() + "%";
        // mPaint.getTextBounds(text, 0, text.length(), mTextBound);

        float textWidth = mPaint.measureText(text);
        float textHeight = (mPaint.descent() + mPaint.ascent()) / 2;

        if (progressPosX + textWidth > mRealWidth)
        {
            progressPosX = mRealWidth - textWidth;
            noNeedBg = true;
        }

        // draw reached bar
        float endX = progressPosX - mTextOffset / 2;
        if (endX > 0)
        {
            mPaint.setColor(mReachedBarColor);
            mPaint.setStrokeWidth(mReachedProgressBarHeight);
            canvas.drawLine(0, 0, endX, 0, mPaint);
        }
        // draw progress bar
        // measure text bound
        if (mIfDrawText)
        {
            mPaint.setColor(mTextColor);
            canvas.drawText(text, progressPosX, -textHeight, mPaint);
        }

        // draw unreached bar
        if (!noNeedBg)
        {
            float start = progressPosX + mTextOffset / 2 + textWidth;
            mPaint.setColor(mUnReachedBarColor);
            mPaint.setStrokeWidth(mUnReachedProgressBarHeight);
            canvas.drawLine(start, 0, mRealWidth, 0, mPaint);
        }

        canvas.restore();

    }

    /**
     * dp 2 px
     * 
     * @param dpVal
     */
    protected int dp2px(int dpVal)
    {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,
                dpVal, getResources().getDisplayMetrics());
    }

    /**
     * sp 2 px
     * 
     * @param spVal
     * @return
     */
    protected int sp2px(int spVal)
    {
        return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP,
                spVal, getResources().getDisplayMetrics());

    }

}
```


### 主要重写了onMeasure与onDraw方法 


### RoundProgressBarWidthNumber         


圆形的进度条和横向的进度条基本变量都是一致的，于是我就让RoundProgressBarWidthNumber extends HorizontalProgressBarWithNumber 了。                
然后需要改变的就是测量和onDraw了：


### 完整代码

```
package com.zhy.view;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Paint.Cap;
import android.graphics.Paint.Style;
import android.graphics.RectF;
import android.util.AttributeSet;

import com.zhy.library.view.R;

public class RoundProgressBarWidthNumber extends
        HorizontalProgressBarWithNumber
{
    /**
     * mRadius of view
     */
    private int mRadius = dp2px(30);
    private int mMaxPaintWidth;

    public RoundProgressBarWidthNumber(Context context)
    {
        this(context, null);
    }

    public RoundProgressBarWidthNumber(Context context, AttributeSet attrs)
    {
        super(context, attrs);

        mReachedProgressBarHeight = (int) (mUnReachedProgressBarHeight * 2.5f);
        TypedArray ta = context.obtainStyledAttributes(attrs,
                R.styleable.RoundProgressBarWidthNumber);
        mRadius = (int) ta.getDimension(
                R.styleable.RoundProgressBarWidthNumber_radius, mRadius);
        ta.recycle();

        mPaint.setStyle(Style.STROKE);
        mPaint.setAntiAlias(true);
        mPaint.setDither(true);
        mPaint.setStrokeCap(Cap.ROUND);

    }

    /**
     * 这里默认在布局中padding值要么不设置，要么全部设置
     */
    @Override
    protected synchronized void onMeasure(int widthMeasureSpec,
            int heightMeasureSpec)
    {

        mMaxPaintWidth = Math.max(mReachedProgressBarHeight,
                mUnReachedProgressBarHeight);
        int expect = mRadius * 2 + mMaxPaintWidth + getPaddingLeft()
                + getPaddingRight();
        int width = resolveSize(expect, widthMeasureSpec);
        int height = resolveSize(expect, heightMeasureSpec);
        int realWidth = Math.min(width, height);

        mRadius = (realWidth - getPaddingLeft() - getPaddingRight() - mMaxPaintWidth) / 2;

        setMeasuredDimension(realWidth, realWidth);

    }

    @Override
    protected synchronized void onDraw(Canvas canvas)
    {

        String text = getProgress() + "%";
        float textWidth = mPaint.measureText(text);
        float textHeight = (mPaint.descent() + mPaint.ascent()) / 2;

        canvas.save();
        canvas.translate(getPaddingLeft() + mMaxPaintWidth / 2, getPaddingTop()
                + mMaxPaintWidth / 2);
        mPaint.setStyle(Style.STROKE);
        // draw unreaded bar
        mPaint.setColor(mUnReachedBarColor);
        mPaint.setStrokeWidth(mUnReachedProgressBarHeight);
        canvas.drawCircle(mRadius, mRadius, mRadius, mPaint);
        // draw reached bar
        mPaint.setColor(mReachedBarColor);
        mPaint.setStrokeWidth(mReachedProgressBarHeight);
        float sweepAngle = getProgress() * 1.0f / getMax() * 360;
        canvas.drawArc(new RectF(0, 0, mRadius * 2, mRadius * 2), 0,
                sweepAngle, false, mPaint);
        // draw text
        mPaint.setStyle(Style.FILL);
        canvas.drawText(text, mRadius - textWidth / 2, mRadius - textHeight,
                mPaint);

        canvas.restore();

    }

}
```


首先获取它的专有属性mRadius，然后根据此属性去测量，测量完成绘制；              
绘制的过程呢？                      
先绘制一个细一点的圆，然后绘制一个粗一点的弧度，二者叠在一起就行。文本呢，绘制在中间~~~总体，没什么代码量。           
好了，两个进度条就到这了，是不是发现简单很多。总体设计上，存在些问题，如果抽取一个BaseProgressBar用于获取公共的属性；然后不同样子的进度条继承分别实现自己的测量和样子，这样结构可能会清晰些~


### 使用

### 布局文件

```
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:zhy="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="25dp" >

        <com.zhy.view.HorizontalProgressBarWithNumber
            android:id="@+id/id_progressbar01"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="50dip"
            android:padding="5dp" />

       

        <com.zhy.view.HorizontalProgressBarWithNumber
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="50dip"
            android:padding="5dp"
            android:progress="50"
            zhy:progress_text_color="#ffF53B03"
            zhy:progress_unreached_color="#ffF7C6B7" />

        <com.zhy.view.RoundProgressBarWidthNumber
            android:id="@+id/id_progress02"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="50dip"
            android:padding="5dp"
            android:progress="30" />

        <com.zhy.view.RoundProgressBarWidthNumber
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="50dip"
            android:padding="5dp"
            android:progress="50"
            zhy:progress_reached_bar_height="20dp"
            zhy:progress_text_color="#ffF53B03"
            zhy:radius="60dp" />
     
    </LinearLayout>

</ScrollView>
```



### MainActivity


```
package com.zhy.sample.progressbar;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;

import com.zhy.annotation.Log;
import com.zhy.view.HorizontalProgressBarWithNumber;

public class MainActivity extends Activity {

    private HorizontalProgressBarWithNumber mProgressBar;
    private static final int MSG_PROGRESS_UPDATE = 0x110;

    private Handler mHandler = new Handler() {
        @Log
        public void handleMessage(android.os.Message msg) {
            int progress = mProgressBar.getProgress();
            mProgressBar.setProgress(++progress);
            if (progress >= 100) {
                mHandler.removeMessages(MSG_PROGRESS_UPDATE);
                
            }
            mHandler.sendEmptyMessageDelayed(MSG_PROGRESS_UPDATE, 100);
        };
    };

    @Log
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mProgressBar = (HorizontalProgressBarWithNumber) findViewById(R.id.id_progressbar01);
        mHandler.sendEmptyMessage(MSG_PROGRESS_UPDATE);

    }

}
```


### 知识点 

- paint.ascent()和paint.descent()

![](http://g.hiphotos.baidu.com/zhidao/wh%3D600%2C800/sign=b67c2f56faf2b211e47b8d48fab04900/1c950a7b02087bf4ac951eccf3d3572c11dfcf9e.jpg)


1.基准点是baseline         
2.ascent：是baseline之上至字符最高处的距离             
3.descent：是baseline之下至字符最低处的距离             
4.leading：是上一行字符的descent到下一行的ascent之间的距离,也就是相邻行间的空白距离                           
5.top：是指的是最高字符到baseline的值,即ascent的最大值           
6.bottom：是指最低字符到baseline的值,即descent的最大值         


- View.resolveSize(int size,int measureSpec)

代码如下 

```
public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize =  MeasureSpec.getSize(measureSpec);
        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
            if (specSize < size) {
                result = specSize | MEASURED_STATE_TOO_SMALL;
            } else {
                result = size;
            }
            break;
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result | (childMeasuredState&MEASURED_STATE_MASK);
    }
```


### 源代码下载

[源代码](https://github.com/hongyangAndroid/Android-ProgressBarWidthNumber)


### 参考链接

[Android 打造形形色色的进度条 实现可以如此简单 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/43371299)



