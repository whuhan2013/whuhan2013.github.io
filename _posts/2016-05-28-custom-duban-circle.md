---
layout: post
title: 豆瓣加载动画实现
date: 2016-5-28
categories: blog
tags: [自定义控件]
description: 豆瓣加载动画实现
---

### 最终效果如下

![](http://www.idtkm.com/wp-content/uploads/2016/05/%E7%AC%91%E8%84%B8.gif)



### ValueAnimator类API   简介

- ofFloat(float… values)    构建ValueAnimator，设置动画的浮点值，需要设置2个以上的值

- setDuration(long duration)    设置动画时长，默认的持续时间为300毫秒。

- setInterpolator(TimeInterpolator value)   设置动画的线性非线性运动，默认AccelerateDecelerateInterpolator

- addUpdateListener(ValueAnimator.AnimatorUpdateListener listener)  监听动画属性每一帧的变化


分解步骤，计算一下总共需要的角度:           
1、一个笑脸，x轴下方的圆弧旋转135°，覆盖2个点，此过程中圆弧增加45°               
2、画布旋转135°，此过程中圆弧增加45°              
3、画布旋转360°，此过程中圆弧减少360/5度                
4、画布旋转90°，此过程中圆弧减少90/5度                          
5、画布旋转135°，释放覆盖的2个点                       



### 实现


```
package com.zj.test;

import android.animation.TimeInterpolator;
import android.animation.ValueAnimator;
import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.view.View;
import android.view.animation.DecelerateInterpolator;

/**
 * Created by jjx on 2016/5/28.
 */
public class customView extends View{



//    public customView(Context context, AttributeSet attrs, int defStyleAttr) {
//        super(context, attrs, defStyleAttr);
//
//        initAnimator(animatorDuration);
//        mPaint=new Paint();
//        mPaint.setStyle(Paint.Style.STROKE);//设置画笔样式为描边，如果已经设置，可以忽略
//        mPaint.setColor(Color.GREEN);
//        mPaint.setStrokeWidth(10);
//    }

    float Width;
    float Height;

    public customView(Context context, AttributeSet attrs) {
        super(context, attrs);

        initAnimator(animatorDuration);
        mPaint=new Paint();
        mPaint.setStyle(Paint.Style.STROKE);//设置画笔样式为描边，如果已经设置，可以忽略
        mPaint.setColor(Color.GREEN);
        mPaint.setStrokeWidth(10);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        Width=MeasureSpec.getSize(widthMeasureSpec);
        mViewWidth=Width;

        Height=MeasureSpec.getSize(heightMeasureSpec);
    }

    Paint mPaint;
    float mViewWidth;


    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.translate(Width/2,Height/2);
        doubanAnimator(canvas, mPaint);
    }

    private ValueAnimator animator;
    private float animatedValue;
    private long animatorDuration = 5000;
    private TimeInterpolator timeInterpolator = new DecelerateInterpolator();

    private void initAnimator(long duration){
        if (animator !=null &&animator.isRunning()){
            animator.cancel();
            animator.start();
        }else {
            animator=ValueAnimator.ofFloat(0,855).setDuration(duration);
            animator.setInterpolator(timeInterpolator);
            animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    animatedValue = (float) animation.getAnimatedValue();
                    invalidate();
                }
            });
            animator.start();
        }
    }


    private void doubanAnimator(Canvas canvas, Paint mPaint){
        mPaint.setStyle(Paint.Style.STROKE);//描边
        mPaint.setStrokeCap(Paint.Cap.ROUND);//圆角笔触
        mPaint.setColor(Color.rgb(97, 195, 109));
        mPaint.setStrokeWidth(15);
        float point = Math.min(mViewWidth,mViewWidth)*0.06f/2;
        float r = point*(float) Math.sqrt(2);
        RectF rectF = new RectF(-r,-r,r,r);
        canvas.save();

        // rotate
        if (animatedValue>=135){
            canvas.rotate(animatedValue-135);
        }

        // draw mouth
        float startAngle=0, sweepAngle=0;
        if (animatedValue<135){
            startAngle = animatedValue +5;
            sweepAngle = 170+animatedValue/3;
        }else if (animatedValue<270){
            startAngle = 135+5;
            sweepAngle = 170+animatedValue/3;
        }else if (animatedValue<630){
            startAngle = 135+5;
            sweepAngle = 260-(animatedValue-270)/5;
        }else if (animatedValue<720){
            startAngle = 135-(animatedValue-630)/2+5;
            sweepAngle = 260-(animatedValue-270)/5;
        }else{
            startAngle = 135-(animatedValue-630)/2-(animatedValue-720)/6+5;
            sweepAngle = 170;
        }
        canvas.drawArc(rectF,startAngle,sweepAngle,false,mPaint);

        // draw eye
        canvas.drawPoints(new float[]{
                -point,-point
                ,point,-point
        },mPaint);

        canvas.restore();
    }

}
```


### 布局文件 

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"

    tools:context="com.zj.test.MainActivity">

    <com.zj.test.customView
        android:layout_width="match_parent"
        android:layout_height="match_parent"

        ></com.zj.test.customView>
</RelativeLayout>
```


### 参考链接

[自定义View——Canvas与ValueAnimator – Idtk](http://www.idtkm.com/customview/customview2/)


