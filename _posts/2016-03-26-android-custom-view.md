---
layout: post
title: Android之自定义控件深入
date: 2016-3-26
categories: blog
tags: [android]
description: 安卓。
---

### 本文主要讲述两个知识点:popwindow的使用和通过继承View实现一个自定义控件，实现点击，手动按钮的效果.


### popwindow的使用  

```
//定义 popupWindow
        popWin = new PopupWindow(MainActivity.this);
        popWin.setWidth(input.getWidth()); //设置宽度
        popWin.setHeight(200);  //设置popWin 高度
        
        popWin.setContentView(listView); //为popWindow填充内容
        popWin.setOutsideTouchable(true); // 点击popWin 以处的区域，自动关闭 popWin
        
        popWin.showAsDropDown(input, 0, 0);//设置 弹出窗口，显示的位置
```   

### 自定义控件实现开关拖动按钮

### 第一步：实现自定义控件要继承view    

```
public class MyToggleButton extends View implements OnClickListener{
```   

### 第二步：写构造函数并初始化     

```
/**
   * 在代码里面创建对象的时候，使用此构造方法
   */
  public MyToggleButton(Context context) {
    super(context);
    // TODO Auto-generated constructor stub
  }

  /**
   * 在布局文件中声名的view，创建时由系统自动调用。
   * @param context 上下文对象
   * @param attrs   属性集
   */
  public MyToggleButton(Context context, AttributeSet attrs) {
    super(context, attrs);
    
    initView();
  }
  
  /**
   * 初始化
   */
  private void initView() {
    
    //初始化图片
    backgroundBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.switch_background);
    slideBtn = BitmapFactory.decodeResource(getResources(), R.drawable.slide_button);
    
    
    //初始化 画笔
    paint = new Paint();
    paint.setAntiAlias(true); // 打开抗矩齿
    
    
    //添加onclick事件监听
    setOnClickListener(this);
  }
```   

### 第三步：重写方法    

```
/*
   * view 对象显示的屏幕上，有几个重要步骤：
   * 1、构造方法 创建 对象。
   * 2、测量view的大小。 onMeasure(int,int);
   * 3、确定view的位置 ，view自身有一些建议权，决定权在 父view手中。  onLayout();
   * 4、绘制 view 的内容 。 onDraw(Canvas)
   */
  
  @Override
  /**
   * 测量尺寸时的回调方法 
   */
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
//    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    
    /**
     * 设置当前view的大小
     * width  :view的宽度
     * height :view的高度   （单位：像素）
     */
    setMeasuredDimension(backgroundBitmap.getWidth(),backgroundBitmap.getHeight());
  }

  //确定位置的时候调用此方法
  //自定义view的时候，作用不大
//  @Override
//  protected void onLayout(boolean changed, int left, int top, int right,
//      int bottom) {
//    super.onLayout(changed, left, top, right, bottom);
//  }
  
  /**
   * 当前开关的状态
   *  true 为开
   */
  private boolean currState = false;
  
  @Override
  /**
   * 绘制当前view的内容
   */
  protected void onDraw(Canvas canvas) {
//    super.onDraw(canvas);
    
    // 绘制 背景
    /*
     * backgroundBitmap 要绘制的图片
     * left 图片的左边届
     * top  图片的上边届
     * paint 绘制图片要使用的画笔
     */
    canvas.drawBitmap(backgroundBitmap, 0, 0, paint);
    
    //绘制 可滑动的按钮
    canvas.drawBitmap(slideBtn, slideBtn_left, 0, paint);
  }
```   

### 第四步：监听点击与拖动事件   

```
/**
   * 判断是否发生拖动，
   * 如果拖动了，就不再响应 onclick 事件
   * 
   */
  private boolean isDrag = false;
  @Override
  /**
   * onclick 事件在View.onTouchEvent 中被解析。
   * 系统对onclick 事件的解析，过于简陋，只要有down 事件  up 事件，系统即认为 发生了click 事件
   * 
   */
  public void onClick(View v) {
    /*
     * 如果没有拖动，才执行改变状态的动作
     */
    if(!isDrag){
      currState = !currState;
      flushState();
    }
  }


  
  /**
   * down 事件时的x值
   */
  private int firstX;
  /**
   * touch 事件的上一个x值
   */
  private int lastX;
  
  @Override
  public boolean onTouchEvent(MotionEvent event) {
    super.onTouchEvent(event);
    
    switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:
      firstX = lastX =(int) event.getX();
      isDrag = false;
      
      break;
    case MotionEvent.ACTION_MOVE:
      
      //判断是否发生拖动
      if(Math.abs(event.getX()-firstX)>5){
        isDrag = true;
      }
      
      //计算 手指在屏幕上移动的距离
      int dis = (int) (event.getX() - lastX);
      
      //将本次的位置 设置给lastX
      lastX = (int) event.getX();
      
      //根据手指移动的距离，改变slideBtn_left 的值
      slideBtn_left = slideBtn_left+dis;
      break;
    case MotionEvent.ACTION_UP:
      
      //在发生拖动的情况下，根据最后的位置，判断当前开关的状态
      if (isDrag) {

        int maxLeft = backgroundBitmap.getWidth() - slideBtn.getWidth(); // slideBtn
                                          // 左边届最大值
        /*
         * 根据 slideBtn_left 判断，当前应是什么状态
         */
        if (slideBtn_left > maxLeft / 2) { // 此时应为 打开的状态
          currState = true;
        } else {
          currState = false;
        }

        flushState();
      }
      break;
    }
    
    flushView();
    
    return true; 
  }
```   

### 第五步：刷新当前状态    

```
/**
   * 刷新当前状态
   */
  private void flushState() {
    if(currState){
      slideBtn_left = backgroundBitmap.getWidth()-slideBtn.getWidth();
    }else{
      slideBtn_left = 0;
    }
    
    flushView(); 
  }
  
  /**
   * 刷新当前视图
   */
  private void flushView() {
    /*
     * 对 slideBtn_left  的值进行判断 ，确保其在合理的位置 即       0<=slideBtn_left <=  maxLeft
     * 
     */
    
    int maxLeft = backgroundBitmap.getWidth()-slideBtn.getWidth();  //  slideBtn 左边届最大值
    
    //确保 slideBtn_left >= 0
    slideBtn_left = (slideBtn_left>0)?slideBtn_left:0;
    
    //确保 slideBtn_left <=maxLeft
    slideBtn_left = (slideBtn_left<maxLeft)?slideBtn_left:maxLeft;
    
    /*
     * 刷新当前视图  导致 执行onDraw执行
     */
    invalidate();
  }
```

### 自定义控件完整代码  

```
package com.itheima.togglebtn28;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;
import android.view.View.OnClickListener;

public class MyToggleButton extends View implements OnClickListener{

  /**
   * 做为背景的图片
   */
  private Bitmap backgroundBitmap;
  /**
   * 可以滑动的图片
   */
  private Bitmap slideBtn;
  private Paint paint;
  /**
   * 滑动按钮的左边届
   */
  private float slideBtn_left;

  /**
   * 在代码里面创建对象的时候，使用此构造方法
   */
  public MyToggleButton(Context context) {
    super(context);
    // TODO Auto-generated constructor stub
  }

  /**
   * 在布局文件中声名的view，创建时由系统自动调用。
   * @param context 上下文对象
   * @param attrs   属性集
   */
  public MyToggleButton(Context context, AttributeSet attrs) {
    super(context, attrs);
    
    initView();
  }
  
  /**
   * 初始化
   */
  private void initView() {
    
    //初始化图片
    backgroundBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.switch_background);
    slideBtn = BitmapFactory.decodeResource(getResources(), R.drawable.slide_button);
    
    
    //初始化 画笔
    paint = new Paint();
    paint.setAntiAlias(true); // 打开抗矩齿
    
    
    //添加onclick事件监听
    setOnClickListener(this);
  }

  /*
   * view 对象显示的屏幕上，有几个重要步骤：
   * 1、构造方法 创建 对象。
   * 2、测量view的大小。 onMeasure(int,int);
   * 3、确定view的位置 ，view自身有一些建议权，决定权在 父view手中。  onLayout();
   * 4、绘制 view 的内容 。 onDraw(Canvas)
   */
  
  @Override
  /**
   * 测量尺寸时的回调方法 
   */
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
//    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    
    /**
     * 设置当前view的大小
     * width  :view的宽度
     * height :view的高度   （单位：像素）
     */
    setMeasuredDimension(backgroundBitmap.getWidth(),backgroundBitmap.getHeight());
  }

  //确定位置的时候调用此方法
  //自定义view的时候，作用不大
//  @Override
//  protected void onLayout(boolean changed, int left, int top, int right,
//      int bottom) {
//    super.onLayout(changed, left, top, right, bottom);
//  }
  
  /**
   * 当前开关的状态
   *  true 为开
   */
  private boolean currState = false;
  
  @Override
  /**
   * 绘制当前view的内容
   */
  protected void onDraw(Canvas canvas) {
//    super.onDraw(canvas);
    
    // 绘制 背景
    /*
     * backgroundBitmap 要绘制的图片
     * left 图片的左边届
     * top  图片的上边届
     * paint 绘制图片要使用的画笔
     */
    canvas.drawBitmap(backgroundBitmap, 0, 0, paint);
    
    //绘制 可滑动的按钮
    canvas.drawBitmap(slideBtn, slideBtn_left, 0, paint);
  }

  /**
   * 判断是否发生拖动，
   * 如果拖动了，就不再响应 onclick 事件
   * 
   */
  private boolean isDrag = false;
  @Override
  /**
   * onclick 事件在View.onTouchEvent 中被解析。
   * 系统对onclick 事件的解析，过于简陋，只要有down 事件  up 事件，系统即认为 发生了click 事件
   * 
   */
  public void onClick(View v) {
    /*
     * 如果没有拖动，才执行改变状态的动作
     */
    if(!isDrag){
      currState = !currState;
      flushState();
    }
  }


  
  /**
   * down 事件时的x值
   */
  private int firstX;
  /**
   * touch 事件的上一个x值
   */
  private int lastX;
  
  @Override
  public boolean onTouchEvent(MotionEvent event) {
    super.onTouchEvent(event);
    
    switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:
      firstX = lastX =(int) event.getX();
      isDrag = false;
      
      break;
    case MotionEvent.ACTION_MOVE:
      
      //判断是否发生拖动
      if(Math.abs(event.getX()-firstX)>5){
        isDrag = true;
      }
      
      //计算 手指在屏幕上移动的距离
      int dis = (int) (event.getX() - lastX);
      
      //将本次的位置 设置给lastX
      lastX = (int) event.getX();
      
      //根据手指移动的距离，改变slideBtn_left 的值
      slideBtn_left = slideBtn_left+dis;
      break;
    case MotionEvent.ACTION_UP:
      
      //在发生拖动的情况下，根据最后的位置，判断当前开关的状态
      if (isDrag) {

        int maxLeft = backgroundBitmap.getWidth() - slideBtn.getWidth(); // slideBtn
                                          // 左边届最大值
        /*
         * 根据 slideBtn_left 判断，当前应是什么状态
         */
        if (slideBtn_left > maxLeft / 2) { // 此时应为 打开的状态
          currState = true;
        } else {
          currState = false;
        }

        flushState();
      }
      break;
    }
    
    flushView();
    
    return true; 
  }

  /**
   * 刷新当前状态
   */
  private void flushState() {
    if(currState){
      slideBtn_left = backgroundBitmap.getWidth()-slideBtn.getWidth();
    }else{
      slideBtn_left = 0;
    }
    
    flushView(); 
  }
  
  /**
   * 刷新当前视力
   */
  private void flushView() {
    /*
     * 对 slideBtn_left  的值进行判断 ，确保其在合理的位置 即       0<=slideBtn_left <=  maxLeft
     * 
     */
    
    int maxLeft = backgroundBitmap.getWidth()-slideBtn.getWidth();  //  slideBtn 左边届最大值
    
    //确保 slideBtn_left >= 0
    slideBtn_left = (slideBtn_left>0)?slideBtn_left:0;
    
    //确保 slideBtn_left <=maxLeft
    slideBtn_left = (slideBtn_left<maxLeft)?slideBtn_left:maxLeft;
    
    /*
     * 刷新当前视图  导致 执行onDraw执行
     */
    invalidate();
  }

}
```

### 第六步：在layout中添加全类名使用   

```
<com.zj.switchbutton.MyTrouggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />
```   

### 运行效果      

![这里写图片描述](http://img.blog.csdn.net/20160326160439705)












