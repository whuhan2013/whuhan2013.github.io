---
layout: post
title: Android之canvas详解
date: 2016-5-14
categories: blog
tags: [自定义控件]
description: Android之canvas详解
---   

 
### 首先说一下canvas类：

```
Class Overview
The Canvas class holds the "draw" calls. To draw something, you need 4 basic components: A Bitmap to hold the pixels, a Canvas to host the draw calls (writing into the bitmap), a drawing primitive (e.g. Rect, Path, text, Bitmap), and a paint (to describe the colors and styles for the drawing). 
```

这个类相当于一个画布，你可以在里面画很多东西；

我们可以把这个Canvas理解成系统提供给我们的一块内存区域(但实际上它只是一套画图的API，真正的内存是下面的Bitmap)，而且它还提供了一整套对这个内存区域进行操作的方法，所有的这些操作都是画图API。也就是说在这种方式下我们已经能一笔一划或者使用Graphic来画我们所需要的东西了，要画什么要显示什么都由我们自己控制。

这种方式根据环境还分为两种：一种就是使用普通View的canvas画图，还有一种就是使用专门的SurfaceView的canvas来画图。两种的主要是区别就是可以在SurfaceView中定义一个专门的线程来完成画图工作，应用程序不需要等待View的刷图，提高性能。前面一种适合处理量比较小，帧率比较小的动画，比如说象棋游戏之类的；而后一种主要用在游戏，高品质动画方面的画图。

下面是Canvas类常用的方法

- drawRect(RectF rect, Paint paint) //绘制区域，参数一为RectF一个区域 

- drawPath(Path path, Paint paint) //绘制一个路径，参数一为Path路径对象

- drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint)  //贴图，参数一就是我们常规的Bitmap对象，参数二是源区域(这里是bitmap)，参数三是目标区域(应该在canvas的位置和大小)，参数四是Paint画刷对象，因为用到了缩放和拉伸的可能，当原始Rect不等于目标Rect时性能将会有大幅损失。

- drawLine(float startX, float startY, float stopX, float stopY, Paintpaint) //画线，参数一起始点的x轴位置，参数二起始点的y轴位置，参数三终点的x轴水平位置，参数四y轴垂直位置，最后一个参数为Paint 画刷对象。

- drawPoint(float x, float y, Paint paint) //画点，参数一水平x轴，参数二垂直y轴，第三个参数为Paint对象。

- drawText(String text, float x, floaty, Paint paint)  //渲染文本，Canvas类除了上面的还可以描绘文字，参数一是String类型的文本，参数二x轴，参数三y轴，参数四是Paint对象。

- drawOval(RectF oval, Paint paint)//画椭圆，参数一是扫描区域，参数二为paint对象；

- drawCircle(float cx, float cy, float radius,Paint paint)// 绘制圆，参数一是中心点的x轴，参数二是中心点的y轴，参数三是半径，参数四是paint对象；

- drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint)//画弧，参数一是RectF对象，一个矩形区域椭圆形的界限用于定义在形状、大小、电弧，参数二是起始角(度)在电弧的开始，
参数三扫描角(度)开始顺时针测量的，参数四是如果这是真的话,包括椭圆中心的电弧,并关闭它,如果它是假这将是一个弧线,参数五是Paint对象；


### 还要理解一个paint类：

```
Class Overview
The Paint class holds the style and color information about how to draw geometries, text and bitmaps. 
paint类拥有风格和颜色信息如何绘制几何学,文本和位图。
Paint 代表了Canvas上的画笔、画刷、颜料等等；
Paint类常用方法:
setARGB(int a, int r, int g, int b) // 设置 Paint对象颜色，参数一为alpha透明值
setAlpha(int a) // 设置alpha不透明度，范围为0~255
setAntiAlias(boolean aa) // 是否抗锯齿
setColor(int color)  // 设置颜色，这里Android内部定义的有Color类包含了一些常见颜色定义
setTextScaleX(float scaleX)  // 设置文本缩放倍数，1.0f为原始
setTextSize(float textSize)  // 设置字体大小
setUnderlineText(booleanunderlineText)  // 设置下划线
```

### 案例
看一下效果图：

![](http://img.blog.csdn.net/20130731234453546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcmhsamlheW91/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### CustomActivity.java

```
public class CustomActivity extends Activity {  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        init();  
    }  
  
    private void init() {  
        LinearLayout layout=(LinearLayout) findViewById(R.id.root);  
        final DrawView view=new DrawView(this);  
        view.setMinimumHeight(500);  
        view.setMinimumWidth(300);  
        //通知view组件重绘    
        view.invalidate();  
        layout.addView(view);  
          
    }  
}  
```

重要的类自定义View组件要重写View组件的onDraw(Canvase)方法，接下来是在该 Canvas上绘制大量的几何图形，点、直线、弧、圆、椭圆、文字、矩形、多边形、曲线、圆角矩形，等各种形状！
DrawView.java

```
public class DrawView extends View {  
  
    public DrawView(Context context) {  
        super(context);  
    }  
  
    @Override  
    protected void onDraw(Canvas canvas) {  
        super.onDraw(canvas);  
        /* 
         * 方法 说明 drawRect 绘制矩形 drawCircle 绘制圆形 drawOval 绘制椭圆 drawPath 绘制任意多边形 
         * drawLine 绘制直线 drawPoin 绘制点 
         */  
        // 创建画笔  
        Paint p = new Paint();  
        p.setColor(Color.RED);// 设置红色  
  
        canvas.drawText("画圆：", 10, 20, p);// 画文本  
        canvas.drawCircle(60, 20, 10, p);// 小圆  
        p.setAntiAlias(true);// 设置画笔的锯齿效果。 true是去除，大家一看效果就明白了  
        canvas.drawCircle(120, 20, 20, p);// 大圆  
  
        canvas.drawText("画线及弧线：", 10, 60, p);  
        p.setColor(Color.GREEN);// 设置绿色  
        canvas.drawLine(60, 40, 100, 40, p);// 画线  
        canvas.drawLine(110, 40, 190, 80, p);// 斜线  
        //画笑脸弧线  
        p.setStyle(Paint.Style.STROKE);//设置空心  
        RectF oval1=new RectF(150,20,180,40);  
        canvas.drawArc(oval1, 180, 180, false, p);//小弧形  
        oval1.set(190, 20, 220, 40);  
        canvas.drawArc(oval1, 180, 180, false, p);//小弧形  
        oval1.set(160, 30, 210, 60);  
        canvas.drawArc(oval1, 0, 180, false, p);//小弧形  
  
        canvas.drawText("画矩形：", 10, 80, p);  
        p.setColor(Color.GRAY);// 设置灰色  
        p.setStyle(Paint.Style.FILL);//设置填满  
        canvas.drawRect(60, 60, 80, 80, p);// 正方形  
        canvas.drawRect(60, 90, 160, 100, p);// 长方形  
  
        canvas.drawText("画扇形和椭圆:", 10, 120, p);  
        /* 设置渐变色 这个正方形的颜色是改变的 */  
        Shader mShader = new LinearGradient(0, 0, 100, 100,  
                new int[] { Color.RED, Color.GREEN, Color.BLUE, Color.YELLOW,  
                        Color.LTGRAY }, null, Shader.TileMode.REPEAT); // 一个材质,打造出一个线性梯度沿著一条线。  
        p.setShader(mShader);  
        // p.setColor(Color.BLUE);  
        RectF oval2 = new RectF(60, 100, 200, 240);// 设置个新的长方形，扫描测量  
        canvas.drawArc(oval2, 200, 130, true, p);  
        // 画弧，第一个参数是RectF：该类是第二个参数是角度的开始，第三个参数是多少度，第四个参数是真的时候画扇形，是假的时候画弧线  
        //画椭圆，把oval改一下  
        oval2.set(210,100,250,130);  
        canvas.drawOval(oval2, p);  
  
        canvas.drawText("画三角形：", 10, 200, p);  
        // 绘制这个三角形,你可以绘制任意多边形  
        Path path = new Path();  
        path.moveTo(80, 200);// 此点为多边形的起点  
        path.lineTo(120, 250);  
        path.lineTo(80, 250);  
        path.close(); // 使这些点构成封闭的多边形  
        canvas.drawPath(path, p);  
  
        // 你可以绘制很多任意多边形，比如下面画六连形  
        p.reset();//重置  
        p.setColor(Color.LTGRAY);  
        p.setStyle(Paint.Style.STROKE);//设置空心  
        Path path1=new Path();  
        path1.moveTo(180, 200);  
        path1.lineTo(200, 200);  
        path1.lineTo(210, 210);  
        path1.lineTo(200, 220);  
        path1.lineTo(180, 220);  
        path1.lineTo(170, 210);  
        path1.close();//封闭  
        canvas.drawPath(path1, p);  
        /* 
         * Path类封装复合(多轮廓几何图形的路径 
         * 由直线段*、二次曲线,和三次方曲线，也可画以油画。drawPath(路径、油漆),要么已填充的或抚摸 
         * (基于油漆的风格),或者可以用于剪断或画画的文本在路径。 
         */  
          
        //画圆角矩形  
        p.setStyle(Paint.Style.FILL);//充满  
        p.setColor(Color.LTGRAY);  
        p.setAntiAlias(true);// 设置画笔的锯齿效果  
        canvas.drawText("画圆角矩形:", 10, 260, p);  
        RectF oval3 = new RectF(80, 260, 200, 300);// 设置个新的长方形  
        canvas.drawRoundRect(oval3, 20, 15, p);//第二个参数是x半径，第三个参数是y半径  
          
        //画贝塞尔曲线  
        canvas.drawText("画贝塞尔曲线:", 10, 310, p);  
        p.reset();  
        p.setStyle(Paint.Style.STROKE);  
        p.setColor(Color.GREEN);  
        Path path2=new Path();  
        path2.moveTo(100, 320);//设置Path的起点  
        path2.quadTo(150, 310, 170, 400); //设置贝塞尔曲线的控制点坐标和终点坐标  
        canvas.drawPath(path2, p);//画出贝塞尔曲线  
          
        //画点  
        p.setStyle(Paint.Style.FILL);  
        canvas.drawText("画点：", 10, 390, p);  
        canvas.drawPoint(60, 390, p);//画一个点  
        canvas.drawPoints(new float[]{60,400,65,400,70,400}, p);//画多个点  
          
        //画图片，就是贴图  
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.ic_launcher);  
        canvas.drawBitmap(bitmap, 250,360, p);  
    }  
}  
```

### 参考链接

[Android利用canvas画各种图形(点、直线、弧、圆、椭圆、文字、矩形、多边形、曲线、圆角矩形) - 任海丽(3G/移动开发) - 博客频道 - CSDN.NET](http://blog.csdn.net/rhljiayou/article/details/7212620)


### 道Canvas可以绘制的对象有：

弧线(arcs)、填充颜色(argb和color)、 Bitmap、圆(circle和oval)、点(point)、线(line)、矩形(Rect)、图片(Picture)、圆角矩形 (RoundRect)、文本(text)、顶点(Vertices)、路径(path)。通过组合这些对象我们可以画出一些简单有趣的界面出来，但是光有这些功能还是不够的，如果我要画一个仪表盘(数字围绕显示在一个圆圈中)呢？ 幸好Android还提供了一些对Canvas位置转换的方法：rorate、scale、translate、skew(扭曲)等，而且它允许你通过获得它的转换矩阵对象(getMatrix方法，不知道什么是转换矩阵？看这里) 直接操作它。这些操作就像是虽然你的笔还是原来的地方画，但是画纸旋转或者移动了，所以你画的东西的方位就产生变化。为了方便一些转换操作，Canvas 还提供了保存和回滚属性的方法(save和restore)，比如你可以先保存目前画纸的位置(save)，然后旋转90度，向下移动100像素后画一些图形，画完后调用restore方法返回到刚才保存的位置。下面我们就演示下canvas的一些简单用法：


```
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    canvas.drawCircle(100, 100, 90, paint);   
}
```

### 效果


![](http://www.jcodecraeer.com/uploads/allimg/130305/19311J4E-1.jpg)

```
@Override   
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    //绘制弧线区域   
                                                                                                                                  
    RectF rect = new RectF(0, 0, 100, 100);   
                                                                                                                                  
    canvas.drawArc(rect, //弧线所使用的矩形区域大小   
            0,  //开始角度   
            90, //扫过的角度   
            false, //是否使用中心   
            paint);   
                                                                                                                                  
}
```

### 效果

![](http://www.jcodecraeer.com/uploads/allimg/130305/19311LN8-2.jpg)


```
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    //绘制弧线区域   
                                                                                                                                  
    RectF rect = new RectF(0, 0, 100, 100);   
                                                                                                                                  
    canvas.drawArc(rect, //弧线所使用的矩形区域大小   
            0,  //开始角度   
            90, //扫过的角度   
            true, //是否使用中心   
            paint);   
                                                                                                                                  
}
```
![](http://www.jcodecraeer.com/uploads/allimg/130305/19311IW6-3.jpg)

两图对比我们可以发现，当 drawArcs(rect,startAngel,sweepAngel,useCenter,paint)中的useCenter为false时，弧线区域是用弧线开始角度和结束角度直接连接起来的，当useCenter为true时，是弧线开始角度和结束角度都与中心点连接，形成一个扇形。


```
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    canvas.drawColor(Color.BLUE);   
                                                                                                                                  
}
```
![](http://www.jcodecraeer.com/uploads/allimg/130305/19311I022-4.jpg)

canvas.drawColor是直接将View显示区域用某个颜色填充满。


```
@Override   
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    //画一条线   
    canvas.drawLine(10, 10, 100, 100, paint);   
                                                                                                                                  
}
```

![](http://www.jcodecraeer.com/uploads/allimg/130305/19311H643-5.jpg)


Canvas.drawOval：

```
@Override   
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    //定义一个矩形区域   
    RectF oval = new RectF(0,0,200,300);   
    //矩形区域内切椭圆   
    canvas.drawOval(oval, paint);   
                                                                                                                                  
}
```

![](http://www.jcodecraeer.com/uploads/allimg/130305/1935452914-0.jpg)

canvas.drawPosText：

```
@Override   
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    //按照既定点 绘制文本内容   
    canvas.drawPosText("Android777", new float[]{   
            10,10, //第一个字母在坐标10,10   
            20,20, //第二个字母在坐标20,20   
            30,30, //....   
            40,40,   
            50,50,   
            60,60,   
            70,70,   
            80,80,   
            90,90,   
            100,100   
    }, paint);   
                                                                                                                                  
}
```

![](http://www.jcodecraeer.com/uploads/allimg/130305/1935454O2-1.jpg)

canvas.drawRect：

```
@Override   
    protected void onDraw(Canvas canvas) {   
                                                                                                                                  
        RectF rect = new RectF(50, 50, 200, 200);   
                                                                                                                                  
        canvas.drawRect(rect, paint);   
                                                                                                                                  
    }   
                                                                                                                                  
}
```

### rect的参数是表示左上角与右下角的坐标  

![](http://www.jcodecraeer.com/uploads/allimg/130305/1935455357-2.jpg)


canvas.drawRoundRect：

```
@Override   
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    RectF rect = new RectF(50, 50, 200, 200);   
                                                                                                                                  
    canvas.drawRoundRect(rect,   
                        30, //x轴的半径   
                        30, //y轴的半径   
                        paint);   
                                                                                                                                  
}
```

![](http://www.jcodecraeer.com/uploads/allimg/130305/1935453P9-3.jpg)

canvas.drawPath：

```
@Override   
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    Path path = new Path(); //定义一条路径   
    path.moveTo(10, 10); //移动到 坐标10,10   
    path.lineTo(50, 60);   
    path.lineTo(200,80);   
    path.lineTo(10, 10);   
                                                                                                                                  
    canvas.drawPath(path, paint);   
                                                                                                                                  
}
```

![](http://www.jcodecraeer.com/uploads/allimg/130305/1935455911-4.jpg)

canvas.drawTextOnPath：

```
@Override   
        protected void onDraw(Canvas canvas) {   
                                                                                                                                  
            Path path = new Path(); //定义一条路径   
            path.moveTo(10, 10); //移动到 坐标10,10   
            path.lineTo(50, 60);   
            path.lineTo(200,80);   
            path.lineTo(10, 10);   
                                                                                                                                  
//          canvas.drawPath(path, paint);   
            canvas.drawTextOnPath("Android777开发者博客", path, 10, 10, paint);   
                                                                                                                                  
        }
```

![](http://www.jcodecraeer.com/uploads/allimg/130305/193545NE-5.jpg)


位置转换方法，canvas.rorate和canvas.translate：


```
@Override   
protected void onDraw(Canvas canvas) {   
                                                                                                                                  
    paint.setAntiAlias(true);   
    paint.setStyle(Style.STROKE);   
    canvas.translate(canvas.getWidth()/2, 200); //将位置移动画纸的坐标点:150,150   
    canvas.drawCircle(0, 0, 100, paint); //画圆圈   
                                                                                                                                  
    //使用path绘制路径文字   
    canvas.save();   
    canvas.translate(-75, -75);   
    Path path = new Path();   
    path.addArc(new RectF(0,0,150,150), -180, 180);   
    Paint citePaint = new Paint(paint);   
    citePaint.setTextSize(14);   
    citePaint.setStrokeWidth(1);   
    canvas.drawTextOnPath("http://www.android777.com", path, 28, 0, citePaint);   
    canvas.restore();   
                                                                                                                                  
    Paint tmpPaint = new Paint(paint); //小刻度画笔对象   
    tmpPaint.setStrokeWidth(1);   
                                                                                                                                  
    float  y=100;   
    int count = 60; //总刻度数   
                                                                                                                                  
    for(int i=0 ; i <count ; i++){   
        if(i%5 == 0){   
            canvas.drawLine(0f, y, 0, y+12f, paint);   
            canvas.drawText(String.valueOf(i/5+1), -4f, y+25f, tmpPaint);   
                                                                                                                                  
        }else{   
            canvas.drawLine(0f, y, 0f, y +5f, tmpPaint);   
        }   
        canvas.rotate(360/count,0f,0f); //旋转画纸   
    }   
                                                                                                                                  
    //绘制指针   
    tmpPaint.setColor(Color.GRAY);   
    tmpPaint.setStrokeWidth(4);   
    canvas.drawCircle(0, 0, 7, tmpPaint);   
    tmpPaint.setStyle(Style.FILL);   
    tmpPaint.setColor(Color.YELLOW);   
    canvas.drawCircle(0, 0, 5, tmpPaint);   
    canvas.drawLine(0, 10, 0, -65, paint);   
                                                                                                                                  
}
```

![](http://www.jcodecraeer.com/uploads/allimg/130305/19354561M-6.jpg)


上面几个例子基本已经将常用的canvas.draw*方法测试过了，我们结合一些事件，做一些有用户交互的应用：


```
package com.android777.demo.uicontroller.graphics;   
                                                                                                                                  
import java.util.ArrayList;   
                                                                                                                                  
import android.app.Activity;   
import android.content.Context;   
import android.graphics.Canvas;   
import android.graphics.Color;   
import android.graphics.Paint;   
import android.graphics.PointF;   
import android.os.Bundle;   
import android.view.MotionEvent;   
import android.view.View;   
                                                                                                                                  
public class CanvasDemoActivity extends Activity {   
                                                                                                                                  
    @Override   
    protected void onCreate(Bundle savedInstanceState) {   
        super.onCreate(savedInstanceState);   
                                                                                                                                  
        setContentView(new CustomView1(this));   
                                                                                                                                  
    }   
                                                                                                                                  
    /**   
     * 使用内部类 自定义一个简单的View   
     * @author Administrator   
     *   
     */
    class CustomView1 extends View{   
                                                                                                                                  
        Paint paint;   
        private ArrayList<PointF> graphics = new ArrayList<PointF>();   
        PointF point;   
                                                                                                                                  
        public CustomView1(Context context) {   
            super(context);   
            paint = new Paint(); //设置一个笔刷大小是3的黄色的画笔   
            paint.setColor(Color.YELLOW);   
            paint.setStrokeJoin(Paint.Join.ROUND);   
            paint.setStrokeCap(Paint.Cap.ROUND);   
            paint.setStrokeWidth(3);   
                                                                                                                                  
        }   
                                                                                                                                  
        @Override   
        public boolean onTouchEvent(MotionEvent event) {   
                                                                                                                                  
            graphics.add(new PointF(event.getX(),event.getY()));   
                                                                                                                                  
            invalidate(); //重新绘制区域   
                                                                                                                                  
            return true;   
        }   
                                                                                                                                  
        //在这里我们将测试canvas提供的绘制图形方法   
        @Override   
        protected void onDraw(Canvas canvas) {   
            for (PointF point : graphics) {   
                canvas.drawPoint(point.x, point.y, paint);   
            }   
//          super.onDraw(canvas);   
                                                                                                                                  
        }   
    }   
                                                                                                                                  
}
```

当用户点击时将出现一个小点，拖动时将画出一条用细点组成的虚线：


![](http://www.jcodecraeer.com/uploads/allimg/130305/1935452617-7.jpg)

### 参考链接

[Android Canvas绘图详解（图文） - 泡在网上的日子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/1212/703.html)

[Android利用canvas画各种图形(点、直线、弧、圆、椭圆、文字、矩形、多边形、曲线、圆角矩形) - 任海丽(3G/移动开发) - 博客频道 - CSDN.NET](http://blog.csdn.net/rhljiayou/article/details/7212620)


### Canvas之translate、scale、rotate、skew详解

### 详情请参见

[Canvas之translate、scale、rotate、skew方法讲解！ - Ajian_studio - 博客频道 - CSDN.NET](http://blog.csdn.net/tianjian4592/article/details/45234419)


### 补充 

- canvas.saveLayer 

Canvas 在一般的情况下可以看作是一张画布，所有的绘图操作如drawBitmap, drawCircle都发生在这张画布上，这张画板还定义了一些属性比如Matrix，颜色等等。但是如果需要实现一些相对复杂的绘图操作，比如多层动画，地图（地图可以有多个地图层叠加而成，比如：政区层，道路层，兴趣点层）。Canvas提供了图层（Layer）支持，缺省情况可以看作是只有一个图层Layer。如果需要按层次来绘图，Android的Canvas可以使用SaveLayerXXX, Restore 来创建一些中间层，对于这些Layer是按照“栈结构“来管理的：  

![](http://img.my.csdn.net/uploads/201212/19/1355906035_7646.png)

 创建一个新的Layer到“栈”中，可以使用saveLayer, savaLayerAlpha, 从“栈”中推出一个Layer，可以使用restore,restoreToCount。但Layer入栈时，后续的DrawXXX操作都发生在这个Layer上，而Layer退栈时，就会把本层绘制的图像“绘制”到上层或是Canvas上，在复制Layer到Canvas上时，可以指定Layer的透明度(Layer），这是在创建Layer时指定的：public int saveLayerAlpha(RectF bounds, int alpha, int saveFlags)本例Layers 介绍了图层的基本用法：Canvas可以看做是由两个图层（Layer）构成的，为了更好的说明问题，我们将代码稍微修改一下，缺省图层绘制一个红色的圆，在新的图层画一个蓝色的圆，新图层的透明度为0×88。      

 ```
 public class Layers extends Activity {  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(new SampleView(this));  
    }  
  
    private static class SampleView extends View {  
        private static final int LAYER_FLAGS = Canvas.MATRIX_SAVE_FLAG | Canvas.CLIP_SAVE_FLAG  
                | Canvas.HAS_ALPHA_LAYER_SAVE_FLAG | Canvas.FULL_COLOR_LAYER_SAVE_FLAG  
                | Canvas.CLIP_TO_LAYER_SAVE_FLAG;  
  
        private Paint mPaint;  
  
        public SampleView(Context context) {  
            super(context);  
            setFocusable(true);  
  
            mPaint = new Paint();  
            mPaint.setAntiAlias(true);  
        }  
  
        @Override  
        protected void onDraw(Canvas canvas) {  
            canvas.drawColor(Color.WHITE);    
            canvas.translate(10, 10);    
            mPaint.setColor(Color.RED);    
            canvas.drawCircle(75, 75, 75, mPaint);    
            canvas.saveLayerAlpha(0, 0, 200, 200, 0x88, LAYER_FLAGS);    
            mPaint.setColor(Color.BLUE);    
            canvas.drawCircle(125, 125, 75, mPaint);    
            canvas.restore();   
         }  
    }  
}  
```

### 参考链接

参考链接 [Android中的canvas介绍 - linghu_java的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/linghu_java/article/details/8939952)


- android中Canvas使用drawBitmap绘制图片

1、基本的绘制图片方法
     
```
   //Bitmap：图片对象，left:偏移左边的位置，top： 偏移顶部的位置
    drawBitmap(Bitmap bitmap, float left, float top, Paint paint)
```

 
2、对图片剪接和限定显示区域
   
```   
drawBitmap(Bitmap bitmap, Rect src, RectF dst, Paint paint)；
Rect src: 是对图片进行裁截，若是空null则显示整个图片
RectF dst：是图片在Canvas画布中显示的区域，
           大于src则把src的裁截区放大，
           小于src则把src的裁截区缩小。
```

### 参考链接  

[android中Canvas使用drawBitmap绘制图片 - longyi_java的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/longyi_java/article/details/6930480)


- canvas.setXfermode属性

1. AvoidXfermode  指定了一个颜色和容差，强制Paint避免在它上面绘图(或者只在它上面绘图)。
2. PixelXorXfermode  当覆盖已有的颜色时，应用一个简单的像素XOR操作。
3. PorterDuffXfermode  这是一个非常强大的转换模式，使用它，可以使用图像合成的16条Porter-Duff规则的任意一条来控制Paint如何与已有的Canvas图像进行交互。

要应用转换模式，可以使用setXferMode方法，如下所示：

```
AvoidXfermode avoid = new AvoidXfermode(Color.BLUE, 10, AvoidXfermode.Mode. AVOID);    
borderPen.setXfermode(avoid);
```

Porter-Duff 效果图：

![](http://hi.csdn.net/attachment/201111/22/0_13219433774KaR.gif)

### 16条Porter-Duff规则 

1.PorterDuff.Mode.CLEAR
   所绘制不会提交到画布上。     

2.PorterDuff.Mode.SRC
   显示上层绘制图片

3.PorterDuff.Mode.DST
  显示下层绘制图片

4.PorterDuff.Mode.SRC_OVER
  正常绘制显示，上下层绘制叠盖。

5.PorterDuff.Mode.DST_OVER
  上下层都显示。下层居上显示。

6.PorterDuff.Mode.SRC_IN
   取两层绘制交集。显示上层。
7.PorterDuff.Mode.DST_IN
  取两层绘制交集。显示下层。

8.PorterDuff.Mode.SRC_OUT
 取上层绘制非交集部分。

9.PorterDuff.Mode.DST_OUT
 取下层绘制非交集部分。

10.PorterDuff.Mode.SRC_ATOP
 取下层非交集部分与上层交集部分

11.PorterDuff.Mode.DST_ATOP
  取上层非交集部分与下层交集部分

12.PorterDuff.Mode.XOR
  
13.PorterDuff.Mode.DARKEN

14.PorterDuff.Mode.LIGHTEN

15.PorterDuff.Mode.MULTIPLY

16.PorterDuff.Mode.SCREEN


### 参考链接:

[setXfermode属性 - xSTARx - ITeye技术网站](http://407827531.iteye.com/blog/1470519)



### 完成