---
layout: post
title: Android手绘效果实现
date: 2016-5-31
categories: blog
tags: [图像处理]
description: Android手绘效果实现
---

### 效果图

![](http://img.blog.csdn.net/20160530101623711)


### 原理

大概介绍一下实现原理。首先你得有一张图（废话~）,接下来就是把这张图的轮廓提取出来，轮廓提取算法有很多，本人不是搞图像处理的，对图像处理感兴趣的童鞋可以查看相关资料。如果你有好的轮廓提取算法，也可以把源码中的算法替换掉，我们采用的轮廓提取算法是Sobel边缘检测。网上的实现有很多，我懒得去实现一遍，github有开源的实现，直接下载了:GraphicLib.本文不具体介绍轮廓提取算法。得到轮廓图以后，接下来要做的是，按照线条的走势，一个一个像素点绘制出来。注意，一定要按照线条的走势显示对应的像素点，如果用两个嵌套的for循环，动画会像网速不好时浏览器显示图片一样。难点就在于此，如何把像素点按照线条方向绘制。接下来我们一起研究。

### 代码实现

如何让绘制的点不会跳远太远，使之连贯起来？首先，对于一个刚绘制完成的点，接下来要绘制的点肯定要选择离它最近的点，这样肯定是最佳的下一个绘制点。因此，只要我们找到最近的点就可以，寻找最近的点，可以通过以圆的方式不断改变半径的大小进行探测。但是用圆的话需要各种三角函数运算，影响效率。我们可以换一种方法：根据当前的点可以轻松得到每一层以该点为中心的正方形，一层层遍历，直到找到需要的点就是我们要的点。遍历的方法很简单，就是比较对应的正方形的上下左右四条边上面的像素点。如下图所示：


![](http://img.blog.csdn.net/20160530101716884)


**接下来看看如何用代码去实现寻找最近的点：**


```
//获取离指定点最近的一个未绘制过的点
    private Point getNearestPoint(Point p) {
        if (p == null) return null;
        //以点p为中心，向外扩大搜索范围，每次搜索的是与p点相距add的正方形
        for (int add = 1; add < mSrcBmWidth && add < mSrcBmHeight; add++) {
            //
            int beginX = (p.x - add) >= 0 ? (p.x - add) : 0;
            int endX = (p.x + add) < mSrcBmWidth ? (p.x + add) : mSrcBmWidth - 1;
            int beginY = (p.y - add) >= 0 ? (p.y - add) : 0;
            int endY = (p.y + add) < mSrcBmHeight ? (p.y + add) : mSrcBmHeight - 1;
            //搜索正方形的上下边
            for (int x = beginX; x <= endX; x++) {
                if (mArray[x][beginY]) {
                   //标记当前点已经访问过
                    mArray[x][beginY] = false;
                    return new Point(x, beginY);
                }
                if (mArray[x][endY]) {
                    //标记当前点已经访问过
                    mArray[x][endY] = false;
                    return new Point(x, endY);
                }
            }
            //搜索正方形的左右边
            for (int y = beginY + 1; y <= endY - 1; y++) {
                if (mArray[beginX][y]) {
                    //标记当前点已经访问过
                    mArray[beginX][beginY] = false;
                    return new Point(beginX, beginY);
                }
                if (mArray[endX][y]) {
                   //标记当前点已经访问过
                    mArray[endX][y] = false;
                    return new Point(endX, y);
                }
            }
        }

        return null;
    }
```


任何一个点，只要还存在没有绘制过的点，就一定能找得到与它最近的点，如果找不到，说明所有的点已经绘制完毕。为了防止查找到重复的点，需要把访问过的点做上记号（即设为false）。我们需要把整张图中每一个像素点位置作好记号，标记哪些点是需要绘制，哪些点是不需要绘制或者是已经绘制过。用一个boolean[][]型数组保存。还需要记录最后一次访问的点，以便继续下一次的绘制。根据最后一次访问的点继续寻找最近点，反复迭代，把所有的点绘制完成后，整张图就出来了。程序开始时，将最后一次访问的点初始化为左上角的点。


```
   private Point mLastPoint = new Point(0, 0);
//获取下一个需要绘制的点
    private Point getNextPoint() {
        mLastPoint = getNearestPoint(mLastPoint);
        return mLastPoint;
    }
```


接下来是将点绘制到Bitmap上，在将Bitmap绘制到SurfaceView的Canvas上。这里这么做的目的是，SurfaceView内部使用了双缓存，直接绘制到SurfaceView的Canvas可能会闪屏。


```
/**
     * //绘制
     * return :false 表示绘制完成，true表示还需要继续绘制
     */
    private boolean draw() {

        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setColor(Color.BLACK);
        //获取count个点后，一次性绘制到bitmap在把bitmap绘制到SurfaceView
        int count = 100;
        Point p = null;
        while (count-- > 0) {
            p = getNextPoint();
            if (p == null) {//如果p为空，说明所有的点已经绘制完成
                return false;
            }
            mTmpCanvas.drawPoint(p.x, p.y + offsetY, mPaint);
        }
        //将bitmap绘制到SurfaceView中
        Canvas canvas = mSurfaceHolder.lockCanvas();
        canvas.drawBitmap(mTmpBm, 0, 0, mPaint);
        if (p != null)
            canvas.drawBitmap(mPaintBm, p.x, p.y - mPaintBm.getHeight() + offsetY, mPaint);
        mSurfaceHolder.unlockCanvasAndPost(canvas);
        return true;
    }
```


**基本上绘制算完成了，附上完整的代码：**


```
package com.hc.myoutline;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Point;
import android.util.AttributeSet;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

/**
 * Package com.hc.myoutline
 * Created by HuaChao on 2016/5/27.
 */
public class DrawOutlineView extends SurfaceView implements SurfaceHolder.Callback {

    private SurfaceHolder mSurfaceHolder;
    private Bitmap mTmpBm;
    private Canvas mTmpCanvas;
    private int mWidth;
    private int mHeight;
    private Paint mPaint;
    private int mSrcBmWidth;
    private int mSrcBmHeight;
    private boolean[][] mArray;
    private int offsetY = 100;

    private Bitmap mPaintBm;
    private Point mLastPoint = new Point(0, 0);

    public DrawOutlineView(Context context) {
        super(context);
        init();
    }

    public DrawOutlineView(Context context, AttributeSet attrs) {
        super(context, attrs);

        init();
    }

    private void init() {
        mSurfaceHolder = getHolder();
        mSurfaceHolder.addCallback(this);
        mPaint = new Paint();
        mPaint.setColor(Color.BLACK);
    }

    //设置画笔图片
    public void setPaintBm(Bitmap paintBm) {
        mPaintBm = paintBm;
    }

    //获取离指定点最近的一个未绘制过的点
    private Point getNearestPoint(Point p) {
        if (p == null) return null;
        //以点p为中心，向外扩大搜索范围，每次搜索的是与p点相距add的正方形
        for (int add = 1; add < mSrcBmWidth && add < mSrcBmHeight; add++) {
            //
            int beginX = (p.x - add) >= 0 ? (p.x - add) : 0;
            int endX = (p.x + add) < mSrcBmWidth ? (p.x + add) : mSrcBmWidth - 1;
            int beginY = (p.y - add) >= 0 ? (p.y - add) : 0;
            int endY = (p.y + add) < mSrcBmHeight ? (p.y + add) : mSrcBmHeight - 1;
            //搜索正方形的上下边
            for (int x = beginX; x <= endX; x++) {
                if (mArray[x][beginY]) {
                    mArray[x][beginY] = false;
                    return new Point(x, beginY);
                }
                if (mArray[x][endY]) {
                    mArray[x][endY] = false;
                    return new Point(x, endY);
                }
            }
            //搜索正方形的左右边
            for (int y = beginY + 1; y <= endY - 1; y++) {
                if (mArray[beginX][y]) {
                    mArray[beginX][beginY] = false;
                    return new Point(beginX, beginY);
                }
                if (mArray[endX][y]) {

                    mArray[endX][y] = false;
                    return new Point(endX, y);
                }
            }
        }

        return null;
    }

    //获取下一个需要绘制的点
    private Point getNextPoint() {
        mLastPoint = getNearestPoint(mLastPoint);
        return mLastPoint;
    }


    /**
     * //绘制
     * return :false 表示绘制完成，true表示还需要继续绘制
     */
    private boolean draw() {

        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setColor(Color.BLACK);
        //获取count个点后，一次性绘制到bitmap在把bitmap绘制到SurfaceView
        int count = 100;
        Point p = null;
        while (count-- > 0) {
            p = getNextPoint();
            if (p == null) {//如果p为空，说明所有的点已经绘制完成
                return false;
            }
            mTmpCanvas.drawPoint(p.x, p.y + offsetY, mPaint);
        }
        //将bitmap绘制到SurfaceView中
        Canvas canvas = mSurfaceHolder.lockCanvas();
        canvas.drawBitmap(mTmpBm, 0, 0, mPaint);
        if (p != null)
            canvas.drawBitmap(mPaintBm, p.x, p.y - mPaintBm.getHeight() + offsetY, mPaint);
        mSurfaceHolder.unlockCanvasAndPost(canvas);
        return true;
    }
    //重画
    public void reDraw(boolean[][] array) {
        if (isDrawing) return;
        mTmpBm = Bitmap.createBitmap(mWidth, mHeight, Bitmap.Config.ARGB_8888);
        mTmpCanvas = new Canvas(mTmpBm);
        mPaint.setColor(Color.WHITE);
        mPaint.setStyle(Paint.Style.FILL);
        mTmpCanvas.drawRect(0, 0, mWidth, mHeight, mPaint);
        mLastPoint = new Point(0, 0);
        beginDraw(array);
    }

    private boolean isDrawing = false;

    public void beginDraw(boolean[][] array) {
        if (isDrawing) return;
        this.mArray = array;
        mSrcBmWidth = array.length;
        mSrcBmHeight = array[0].length;

        new Thread() {
            @Override
            public void run() {
                while (true) {
                    isDrawing = true;
                    boolean rs = draw();
                    if (!rs) break;
                    try {
                        sleep(20);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                isDrawing = false;
            }
        }.start();
    }


    @Override
    public void surfaceCreated(SurfaceHolder holder) {

    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        this.mWidth = width;
        this.mHeight = height;
        mTmpBm = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
        mTmpCanvas = new Canvas(mTmpBm);
        mPaint.setColor(Color.WHITE);
        mPaint.setStyle(Paint.Style.FILL);
        mTmpCanvas.drawRect(0, 0, mWidth, mHeight, mPaint);
        Canvas canvas = holder.lockCanvas();
        canvas.drawBitmap(mTmpBm, 0, 0, mPaint);
        holder.unlockCanvasAndPost(canvas);

        mPaint.setStyle(Paint.Style.STROKE);
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {

    }
}
```



**接下来是在MainActivity里面传入参数过来，附上MainActivity的代码：**


```
package com.hc.myoutline;

import android.graphics.Bitmap;
import android.graphics.Color;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.MotionEvent;

public class MainActivity extends AppCompatActivity {
    private DrawOutlineView drawOutlineView;
    private Bitmap sobelBm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);
        //将Bitmap压缩处理，防止OOM
        Bitmap bm = CommenUtils.getRatioBitmap(this, R.drawable.test, 100, 100);
        //返回的是处理过的Bitmap
        sobelBm = SobelUtils.Sobel(bm);
        drawOutlineView = (DrawOutlineView) findViewById(R.id.outline);

        Bitmap paintBm = CommenUtils.getRatioBitmap(this, R.drawable.paint, 10, 20);
        drawOutlineView.setPaintBm(paintBm);

    }
    //根据Bitmap信息，获取每个位置的像素点是否需要绘制
    //使用boolean数组而不是int[][]主要是考虑到内存的消耗
    private boolean[][] getArray(Bitmap bitmap) {
        boolean[][] b = new boolean[bitmap.getWidth()][bitmap.getHeight()];

        for (int i = 0; i < bitmap.getWidth(); i++) {
            for (int j = 0; j < bitmap.getHeight(); j++) {
                if (bitmap.getPixel(i, j) != Color.WHITE)
                    b[i][j] = true;
                else
                    b[i][j] = false;
            }
        }
        return b;
    }

    boolean first = true;

   //点击时开始绘制
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (first) {
            first = false;
            drawOutlineView.beginDraw(getArray(sobelBm));
        } else
            drawOutlineView.reDraw(getArray(sobelBm));
        return true;
    }
}
```


关于轮廓提取的具体实现代码这里不粘出，可以去GitHub查看：[GraphicLib](https://github.com/zl-leaf/GraphicLib) 或者是下载我的源代码查看。所有内容已经结束，赶紧下载源码运行一下你的照片去秀一下你的逼格吧！

最后的最后：请注意，源码中，轮廓的提取是运行在主线程中，如果图片比较复杂，可能会导致ANR，建议另开线程处理。别问为什么我不去改，一个字：懒！另外，接下来一篇文章中，我将介绍提高运算速度相关内容，提升轮廓提取速度，敬请期待~

源码地址：[Android自动手绘，圆你儿时画家梦](http://download.csdn.net/detail/huachao1001/9533380)


### 参考链接

[Android自动手绘，圆你儿时画家梦！ - huachao1001的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/huachao1001/article/details/51518322)






