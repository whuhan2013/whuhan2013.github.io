---
layout: post
title: Android之SurfaceView
date: 2016-5-24
categories: blog
tags: [自定义控件]
description: Android之SurfaceView
---   

SurfaceView也是继承了View，但是我们并不需要去实现它的draw方法来绘制自己，为什么呢？            
因为它和View有一个很大的区别，View在UI线程去更新自己；而SurfaceView则在一个子线程中去更新自己；这也显示出了它的优势，当制作游戏等需要不断刷新View时，因为是在子线程，避免了对UI线程的阻塞。                 
知道了优势以后，你会想那么不使用draw方法，哪来的canvas使用呢？
大家都记得更新View的时候draw方法提供了一个canvas，SurfaceView内部内嵌了一个专门用于绘制的Surface，而这个Surface中包含一个Canvas。             
有了Canvas，我们如何获取呢？                
SurfaceView里面有个getHolder方法，我们可以获取一个SurfaceHolder。通过SurfaceHolder可以监听SurfaceView的生命周期以及获取Canvas对象。  


### 通常用法 

```
public class SurfaceViewTemplate extends SurfaceView implements Callback, Runnable
{

    private SurfaceHolder mHolder;
    /**
     * 与SurfaceHolder绑定的Canvas
     */
    private Canvas mCanvas;
    /**
     * 用于绘制的线程
     */
    private Thread t;
    /**
     * 线程的控制开关
     */
    private boolean isRunning;

    public SurfaceViewTemplate(Context context)
    {
        this(context, null);
    }

    public SurfaceViewTemplate(Context context, AttributeSet attrs)
    {
        super(context, attrs);

        mHolder = getHolder();
        mHolder.addCallback(this);

        // setZOrderOnTop(true);// 设置画布 背景透明
        // mHolder.setFormat(PixelFormat.TRANSLUCENT);
        
        //设置可获得焦点
        setFocusable(true);
        setFocusableInTouchMode(true);
        //设置常亮
        this.setKeepScreenOn(true);

    }

    @Override
    public void surfaceCreated(SurfaceHolder holder)
    {

        // 开启线程
        isRunning = true;
        t = new Thread(this);
        t.start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width,
            int height)
    {
        // TODO Auto-generated method stub

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder)
    {
        // 通知关闭线程
        isRunning = false;
    }

    @Override
    public void run()
    {
        // 不断的进行draw
        while (isRunning)
        {
            draw();
        }

    }

    private void draw()
    {
        try
        {
            // 获得canvas
            mCanvas = mHolder.lockCanvas();
            if (mCanvas != null)
            {
                // drawSomething..
            }
        } catch (Exception e)
        {
        } finally
        {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```


结合上面我们的介绍，我们在构造中通过getHolder拿到SurfaceHolder对象，然后设置一个addCallback回调，去监听SurfaceView的生命周期，生命周期有三个方法，分别为create,change,destory;我们一般在create里面进行初始化的一些操作，然后开启线程；在destroy里面设置关闭线程；             
所有的绘制流程都是线程的run方法里面，可以看到我们的draw方法。
注意下，我们在draw里面进行了try catch然后很多的判空，主要是因为，当用户点击back或者按下home键以后，surfaceview会被销毁；
 mHolder.lockCanvas();返回的就是null了，所以为了避免造成空指针错误，我们各种判null，甚至还加了个try catch。   

### 实例   抽奖转盘

### 效果图

![](http://img.blog.csdn.net/20141204131218761)



### 代码如下

```
package com.zhy.view;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.Paint.Style;
import android.graphics.Path;
import android.graphics.Rect;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.util.Log;
import android.util.TypedValue;
import android.view.SurfaceHolder;
import android.view.SurfaceHolder.Callback;
import android.view.SurfaceView;

import com.zhy.demo_zhy_06_choujiangzhuanpan.R;

public class LuckyPanView extends SurfaceView implements Callback, Runnable
{

    private SurfaceHolder mHolder;
    /**
     * 与SurfaceHolder绑定的Canvas
     */
    private Canvas mCanvas;
    /**
     * 用于绘制的线程
     */
    private Thread t;
    /**
     * 线程的控制开关
     */
    private boolean isRunning;

    /**
     * 抽奖的文字
     */
    private String[] mStrs = new String[] { "单反相机", "IPAD", "恭喜发财", "IPHONE",
            "妹子一只", "恭喜发财" };
    /**
     * 每个盘块的颜色
     */
    private int[] mColors = new int[] { 0xFFFFC300, 0xFFF17E01, 0xFFFFC300,
            0xFFF17E01, 0xFFFFC300, 0xFFF17E01 };
    /**
     * 与文字对应的图片
     */
    private int[] mImgs = new int[] { R.drawable.danfan, R.drawable.ipad,
            R.drawable.f040, R.drawable.iphone, R.drawable.meizi,
            R.drawable.f040 };

    /**
     * 与文字对应图片的bitmap数组
     */
    private Bitmap[] mImgsBitmap;
    /**
     * 盘块的个数
     */
    private int mItemCount = 6;

    /**
     * 绘制盘块的范围
     */
    private RectF mRange = new RectF();
    /**
     * 圆的直径
     */
    private int mRadius;
    /**
     * 绘制盘快的画笔
     */
    private Paint mArcPaint;

    /**
     * 绘制文字的画笔
     */
    private Paint mTextPaint;

    /**
     * 滚动的速度
     */
    private double mSpeed;
    private volatile float mStartAngle = 0;
    /**
     * 是否点击了停止
     */
    private boolean isShouldEnd;

    /**
     * 控件的中心位置
     */
    private int mCenter;
    /**
     * 控件的padding，这里我们认为4个padding的值一致，以paddingleft为标准
     */
    private int mPadding;

    /**
     * 背景图的bitmap
     */
    private Bitmap mBgBitmap = BitmapFactory.decodeResource(getResources(),
            R.drawable.bg2);
    /**
     * 文字的大小
     */
    private float mTextSize = TypedValue.applyDimension(
            TypedValue.COMPLEX_UNIT_SP, 20, getResources().getDisplayMetrics());

    public LuckyPanView(Context context)
    {
        this(context, null);
    }

    public LuckyPanView(Context context, AttributeSet attrs)
    {
        super(context, attrs);

        mHolder = getHolder();
        mHolder.addCallback(this);

        // setZOrderOnTop(true);// 设置画布 背景透明
        // mHolder.setFormat(PixelFormat.TRANSLUCENT);

        setFocusable(true);
        setFocusableInTouchMode(true);
        this.setKeepScreenOn(true);

    }

    /**
     * 设置控件为正方形
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
    {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        int width = Math.min(getMeasuredWidth(), getMeasuredHeight());
        // 获取圆形的直径
        mRadius = width - getPaddingLeft() - getPaddingRight();
        // padding值
        mPadding = getPaddingLeft();
        // 中心点
        mCenter = width / 2;
        setMeasuredDimension(width, width);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder)
    {
        // 初始化绘制圆弧的画笔
        mArcPaint = new Paint();
        mArcPaint.setAntiAlias(true);
        mArcPaint.setDither(true);
        // 初始化绘制文字的画笔
        mTextPaint = new Paint();
        mTextPaint.setColor(0xFFffffff);
        mTextPaint.setTextSize(mTextSize);
        // 圆弧的绘制范围
        mRange = new RectF(getPaddingLeft(), getPaddingLeft(), mRadius
                + getPaddingLeft(), mRadius + getPaddingLeft());

        // 初始化图片
        mImgsBitmap = new Bitmap[mItemCount];
        for (int i = 0; i < mItemCount; i++)
        {
            mImgsBitmap[i] = BitmapFactory.decodeResource(getResources(),
                    mImgs[i]);
        }

        // 开启线程
        isRunning = true;
        t = new Thread(this);
        t.start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width,
            int height)
    {
        // TODO Auto-generated method stub

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder)
    {
        // 通知关闭线程
        isRunning = false;
    }

    @Override
    public void run()
    {
        // 不断的进行draw
        while (isRunning)
        {
            long start = System.currentTimeMillis();
            draw();
            long end = System.currentTimeMillis();
            try
            {
                if (end - start < 50)
                {
                    Thread.sleep(50 - (end - start));
                }
            } catch (InterruptedException e)
            {
                e.printStackTrace();
            }

        }

    }

    private void draw()
    {
        try
        {
            // 获得canvas
            mCanvas = mHolder.lockCanvas();
            if (mCanvas != null)
            {
                // 绘制背景图
                drawBg();

                /**
                 * 绘制每个块块，每个块块上的文本，每个块块上的图片
                 */
                float tmpAngle = mStartAngle;
                float sweepAngle = (float) (360 / mItemCount);
                for (int i = 0; i < mItemCount; i++)
                {
                    // 绘制快快
                    mArcPaint.setColor(mColors[i]);
//                  mArcPaint.setStyle(Style.STROKE);
                    mCanvas.drawArc(mRange, tmpAngle, sweepAngle, true,
                            mArcPaint);
                    // 绘制文本
                    drawText(tmpAngle, sweepAngle, mStrs[i]);
                    // 绘制Icon
                    drawIcon(tmpAngle, i);

                    tmpAngle += sweepAngle;
                }

                // 如果mSpeed不等于0，则相当于在滚动
                mStartAngle += mSpeed;

                // 点击停止时，设置mSpeed为递减，为0值转盘停止
                if (isShouldEnd)
                {
                    mSpeed -= 1;
                }
                if (mSpeed <= 0)
                {
                    mSpeed = 0;
                    isShouldEnd = false;
                }
                // 根据当前旋转的mStartAngle计算当前滚动到的区域
                calInExactArea(mStartAngle);
            }
        } catch (Exception e)
        {
            e.printStackTrace();
        } finally
        {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);
        }

    }

    /**
     * 根据当前旋转的mStartAngle计算当前滚动到的区域 绘制背景，不重要，完全为了美观
     */
    private void drawBg()
    {
        mCanvas.drawColor(0xFFFFFFFF);
        mCanvas.drawBitmap(mBgBitmap, null, new Rect(mPadding / 2,
                mPadding / 2, getMeasuredWidth() - mPadding / 2,
                getMeasuredWidth() - mPadding / 2), null);
    }

    /**
     * 根据当前旋转的mStartAngle计算当前滚动到的区域
     * 
     * @param startAngle
     */
    public void calInExactArea(float startAngle)
    {
        // 让指针从水平向右开始计算
        float rotate = startAngle + 90;
        rotate %= 360.0;
        for (int i = 0; i < mItemCount; i++)
        {
            // 每个的中奖范围
            float from = 360 - (i + 1) * (360 / mItemCount);
            float to = from + 360 - (i) * (360 / mItemCount);

            if ((rotate > from) && (rotate < to))
            {
                Log.d("TAG", mStrs[i]);
                return;
            }
        }
    }

    /**
     * 绘制图片
     * 
     * @param startAngle
     * @param sweepAngle
     * @param i
     */
    private void drawIcon(float startAngle, int i)
    {
        // 设置图片的宽度为直径的1/8
        int imgWidth = mRadius / 8;

        float angle = (float) ((30 + startAngle) * (Math.PI / 180));

        int x = (int) (mCenter + mRadius / 2 / 2 * Math.cos(angle));
        int y = (int) (mCenter + mRadius / 2 / 2 * Math.sin(angle));

        // 确定绘制图片的位置
        Rect rect = new Rect(x - imgWidth / 2, y - imgWidth / 2, x + imgWidth
                / 2, y + imgWidth / 2);

        mCanvas.drawBitmap(mImgsBitmap[i], null, rect, null);

    }

    /**
     * 绘制文本
     * 
     * @param rect
     * @param startAngle
     * @param sweepAngle
     * @param string
     */
    private void drawText(float startAngle, float sweepAngle, String string)
    {
        
        
        Path path = new Path();
        path.addArc(mRange, startAngle, sweepAngle);
        float textWidth = mTextPaint.measureText(string);
        // 利用水平偏移让文字居中
        float hOffset = (float) (mRadius * Math.PI / mItemCount / 2 - textWidth / 2);// 水平偏移
        float vOffset = mRadius / 2 / 6;// 垂直偏移
        mCanvas.drawTextOnPath(string, path, hOffset, vOffset, mTextPaint);
    }

    /**
     * 点击开始旋转
     * 
     * @param luckyIndex
     */
    public void luckyStart(int luckyIndex)
    {
        // 每项角度大小
        float angle = (float) (360 / mItemCount);
        // 中奖角度范围（因为指针向上，所以水平第一项旋转到指针指向，需要旋转210-270；）
        float from = 270 - (luckyIndex + 1) * angle;
        float to = from + angle;
        // 停下来时旋转的距离
        float targetFrom = 4 * 360 + from;
        /**
         * <pre>
         *  (v1 + 0) * (v1+1) / 2 = target ;
         *  v1*v1 + v1 - 2target = 0 ;
         *  v1=-1+(1*1 + 8 *1 * target)/2;
         * </pre>
         */
        float v1 = (float) (Math.sqrt(1 * 1 + 8 * 1 * targetFrom) - 1) / 2;
        float targetTo = 4 * 360 + to;
        float v2 = (float) (Math.sqrt(1 * 1 + 8 * 1 * targetTo) - 1) / 2;

        mSpeed = (float) (v1 + Math.random() * (v2 - v1));
        isShouldEnd = false;
    }

    public void luckyEnd()
    {
        mStartAngle = 0;
        isShouldEnd = true;
    }

    public boolean isStart()
    {
        return mSpeed != 0;
    }

    public boolean isShouldEnd()
    {
        return isShouldEnd;
    }

}

```



### 代码下载

[源代码](http://download.csdn.net/detail/lmj623565791/8224109)


### 参考链接

[Android SurfaceView实战 打造抽奖转盘 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/41722441)


### 实战 flabby bird


### 效果图

![](http://img.blog.csdn.net/20150121112136209)



### 分析       
仔细观察游戏，需要绘制的有：背景、地板、鸟、管道、分数；       
游戏开始时：                                  
地板给人一种想左移动的感觉；                          
管道与地板同样的速度向左移动；                         
鸟默认下落；                                                               
当用户touch屏幕时，鸟上升一段距离后，下落；                                         
运动过程中需要判断管道和鸟之间的位置关系，是否触碰，是否穿过等，需要计算分数。



### 可以分为

1. piple类
2. bird类
3. floor类
4. Game类

等来实现


### GameFlabbyBird类代码

```
package com.zhy.view;

import java.util.ArrayList;
import java.util.List;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.PixelFormat;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.SurfaceHolder;
import android.view.SurfaceHolder.Callback;
import android.view.SurfaceView;

import com.zhy.surfaceViewDemo.R;

public class GameFlabbyBird extends SurfaceView implements Callback, Runnable
{

    private SurfaceHolder mHolder;
    /**
     * 与SurfaceHolder绑定的Canvas
     */
    private Canvas mCanvas;
    /**
     * 用于绘制的线程
     */
    private Thread t;
    /**
     * 线程的控制开关
     */
    private boolean isRunning;

    private Paint mPaint;

    /**
     * 当前View的尺寸
     */
    private int mWidth;
    private int mHeight;
    private RectF mGamePanelRect = new RectF();

    /**
     * 背景
     */
    private Bitmap mBg;

    /**
     * *********鸟相关**********************
     */
    private Bird mBird;
    private Bitmap mBirdBitmap;
    /**
     * 地板
     */
    private Floor mFloor;
    private Bitmap mFloorBg;

    /**
     * *********管道相关**********************
     */
    /**
     * 管道
     */
    private Bitmap mPipeTop;
    private Bitmap mPipeBottom;
    private RectF mPipeRect;
    private int mPipeWidth;
    /**
     * 管道的宽度 60dp
     */
    private static final int PIPE_WIDTH = 60;
    private List<Pipe> mPipes = new ArrayList<Pipe>();

    /**
     * 分数
     */
    private final int[] mNums = new int[] { R.drawable.n0, R.drawable.n1,
            R.drawable.n2, R.drawable.n3, R.drawable.n4, R.drawable.n5,
            R.drawable.n6, R.drawable.n7, R.drawable.n8, R.drawable.n9 };
    private Bitmap[] mNumBitmap;

    private int mGrade = 0;
    private int mRemovedPipe = 0;

    private static final float RADIO_SINGLE_NUM_HEIGHT = 1 / 15f;
    private int mSingleGradeWidth;
    private int mSingleGradeHeight;
    private RectF mSingleNumRectF;

    private int mSpeed = Util.dp2px(getContext(), 2);

    /***********************************/

    private enum GameStatus
    {
        WAITTING, RUNNING, STOP;
    }

    /**
     * 记录游戏的状态
     */
    private GameStatus mStatus = GameStatus.WAITTING;

    /**
     * 触摸上升的距离，因为是上升，所以为负值
     */
    private static final int TOUCH_UP_SIZE = -16;
    /**
     * 将上升的距离转化为px；这里多存储一个变量，变量在run中计算
     * 
     */
    private final int mBirdUpDis = Util.dp2px(getContext(), TOUCH_UP_SIZE);

    private int mTmpBirdDis;
    /**
     * 鸟自动下落的距离
     */
    private final int mAutoDownSpeed = Util.dp2px(getContext(), 2);

    /**
     * 两个管道间距离
     */
    private final int PIPE_DIS_BETWEEN_TWO = Util.dp2px(getContext(), 300);
    /**
     * 记录移动的距离，达到 PIPE_DIS_BETWEEN_TWO 则生成一个管道
     */
    private int mTmpMoveDistance;
    /**
     * 记录需要移除的管道
     */
    private List<Pipe> mNeedRemovePipe = new ArrayList<Pipe>();

    /**
     * 处理一些逻辑上的计算
     */
    private void logic()
    {
        switch (mStatus)
        {
        case RUNNING:

            mGrade = 0;
            // 更新我们地板绘制的x坐标，地板移动
            mFloor.setX(mFloor.getX() - mSpeed);

            logicPipe();

            // 默认下落，点击时瞬间上升
            mTmpBirdDis += mAutoDownSpeed;
            mBird.setY(mBird.getY() + mTmpBirdDis);

            // 计算分数
            mGrade += mRemovedPipe;
            for (Pipe pipe : mPipes)
            {
                if (pipe.getX() + mPipeWidth < mBird.getX())
                {
                    mGrade++;
                }
            }

            checkGameOver();

            break;

        case STOP: // 鸟落下
            // 如果鸟还在空中，先让它掉下来
            if (mBird.getY() < mFloor.getY() - mBird.getWidth())
            {
                mTmpBirdDis += mAutoDownSpeed;
                
                mBird.setY(mBird.getY() + mTmpBirdDis);
            } else
            {
                mStatus = GameStatus.WAITTING;
                initPos();
            }
            break;
        default:
            break;
        }

    }

    /**
     * 重置鸟的位置等数据
     */
    private void initPos()
    {
        mPipes.clear();
        //立即增加一个
        mPipes.add(new Pipe(getContext(), getWidth(), getHeight(), mPipeTop,
                mPipeBottom));
        mNeedRemovePipe.clear();
        // 重置鸟的位置
        // mBird.setY(mHeight * 2 / 3);
        mBird.resetHeigt();
        // 重置下落速度
        mTmpBirdDis = 0;
        mTmpMoveDistance = 0 ;
        mRemovedPipe = 0;
    }

    private void checkGameOver()
    {

        // 如果触碰地板，gg
        if (mBird.getY() > mFloor.getY() - mBird.getHeight())
        {
            mStatus = GameStatus.STOP;
        }
        // 如果撞到管道
        for (Pipe wall : mPipes)
        {
            // 已经穿过的
            if (wall.getX() + mPipeWidth < mBird.getX())
            {
                continue;
            }
            if (wall.touchBird(mBird))
            {
                mStatus = GameStatus.STOP;
                break;
            }
        }
    }

    private void logicPipe()
    {
        // 管道移动
        for (Pipe pipe : mPipes)
        {
            if (pipe.getX() < -mPipeWidth)
            {
                mNeedRemovePipe.add(pipe);
                mRemovedPipe++;
                continue;
            }
            pipe.setX(pipe.getX() - mSpeed);
        }
        // 移除管道
        mPipes.removeAll(mNeedRemovePipe);
        mNeedRemovePipe.clear();

        // Log.e("TAG", "现存管道数量：" + mPipes.size());

        // 管道
        mTmpMoveDistance += mSpeed;
        // 生成一个管道
        if (mTmpMoveDistance >= PIPE_DIS_BETWEEN_TWO)
        {
            Pipe pipe = new Pipe(getContext(), getWidth(), getHeight(),
                    mPipeTop, mPipeBottom);
            mPipes.add(pipe);
            mTmpMoveDistance = 0;
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event)
    {

        int action = event.getAction();

        if (action == MotionEvent.ACTION_DOWN)
        {
            switch (mStatus)
            {
            case WAITTING:
                mStatus = GameStatus.RUNNING;
                break;
            case RUNNING:
                mTmpBirdDis = mBirdUpDis;
                break;
            }

        }

        return true;

    }

    public GameFlabbyBird(Context context)
    {
        this(context, null);
    }

    public GameFlabbyBird(Context context, AttributeSet attrs)
    {
        super(context, attrs);

        mHolder = getHolder();
        mHolder.addCallback(this);

        setZOrderOnTop(true);// 设置画布 背景透明
        mHolder.setFormat(PixelFormat.TRANSLUCENT);

        // 设置可获得焦点
        setFocusable(true);
        setFocusableInTouchMode(true);
        // 设置常亮
        this.setKeepScreenOn(true);

        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setDither(true);

        initBitmaps();

        // 初始化速度
        // mSpeed = Util.dp2px(getContext(), 2);
        mPipeWidth = Util.dp2px(getContext(), PIPE_WIDTH);

    }

    /**
     * 初始化图片
     */
    private void initBitmaps()
    {
        mBg = loadImageByResId(R.drawable.bg1);
        mBirdBitmap = loadImageByResId(R.drawable.b1);
        mFloorBg = loadImageByResId(R.drawable.floor_bg2);
        mPipeTop = loadImageByResId(R.drawable.g2);
        mPipeBottom = loadImageByResId(R.drawable.g1);

        mNumBitmap = new Bitmap[mNums.length];
        for (int i = 0; i < mNumBitmap.length; i++)
        {
            mNumBitmap[i] = loadImageByResId(mNums[i]);
        }
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder)
    {
        Log.e("TAG", "surfaceCreated");

        // 开启线程
        isRunning = true;
        t = new Thread(this);
        t.start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width,
            int height)
    {
        Log.e("TAG", "surfaceChanged");
        // TODO Auto-generated method stub

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder)
    {
        Log.e("TAG", "surfaceDestroyed");
        // 通知关闭线程
        isRunning = false;
    }

    @Override
    public void run()
    {
        while (isRunning)
        {
            long start = System.currentTimeMillis();
            logic();
            draw();
            long end = System.currentTimeMillis();

            try
            {
                if (end - start < 50)
                {
                    Thread.sleep(50 - (end - start));
                }
            } catch (InterruptedException e)
            {
                e.printStackTrace();
            }

        }

    }

    private void draw()
    {
        try
        {
            // 获得canvas
            mCanvas = mHolder.lockCanvas();
            if (mCanvas != null)
            {
                // drawSomething..

                drawBg();
                drawBird();

                drawPipes();

                drawFloor();

                drawGrades();

            }
        } catch (Exception e)
        {
        } finally
        {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);
        }
    }

    private void drawFloor()
    {
        mFloor.draw(mCanvas, mPaint);
    }

    /**
     * 绘制背景
     */
    private void drawBg()
    {
        mCanvas.drawBitmap(mBg, null, mGamePanelRect, null);
    }

    private void drawBird()
    {
        mBird.draw(mCanvas);
    }

    /**
     * 绘制管道
     */
    private void drawPipes()
    {
        for (Pipe pipe : mPipes)
        {
            // pipe.setX(pipe.getX() - mSpeed);
            pipe.draw(mCanvas, mPipeRect);
        }
    }

    /**
     * 初始化尺寸相关
     */
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh)
    {

        super.onSizeChanged(w, h, oldw, oldh);

        mWidth = w;
        mHeight = h;
        mGamePanelRect.set(0, 0, w, h);

        // 初始化mBird
        mBird = new Bird(getContext(), mWidth, mHeight, mBirdBitmap);
        // 初始化地板
        mFloor = new Floor(mWidth, mHeight, mFloorBg);
        // 初始化管道范围
        mPipeRect = new RectF(0, 0, mPipeWidth, mHeight);

        Pipe pipe = new Pipe(getContext(), w, h, mPipeTop, mPipeBottom);
        mPipes.add(pipe);

        // 初始化分数
        mSingleGradeHeight = (int) (h * RADIO_SINGLE_NUM_HEIGHT);
        mSingleGradeWidth = (int) (mSingleGradeHeight * 1.0f
                / mNumBitmap[0].getHeight() * mNumBitmap[0].getWidth());
        mSingleNumRectF = new RectF(0, 0, mSingleGradeWidth, mSingleGradeHeight);

    }

    /**
     * 绘制分数
     */
    private void drawGrades()
    {
        String grade = mGrade + "";
        mCanvas.save(Canvas.MATRIX_SAVE_FLAG);
        mCanvas.translate(mWidth / 2 - grade.length() * mSingleGradeWidth / 2,
                1f / 8 * mHeight);
        // draw single num one by one
        for (int i = 0; i < grade.length(); i++)
        {
            String numStr = grade.substring(i, i + 1);
            int num = Integer.valueOf(numStr);
            mCanvas.drawBitmap(mNumBitmap[num], null, mSingleNumRectF, null);
            mCanvas.translate(mSingleGradeWidth, 0);
        }
        mCanvas.restore();

    }

    /**
     * 根据resId加载图片
     * 
     * @param resId
     * @return
     */
    private Bitmap loadImageByResId(int resId)
    {
        return BitmapFactory.decodeResource(getResources(), resId);
    }
}
```


### MainActivity代码


```
package com.zhy.surfaceViewDemo;

import com.zhy.view.GameFlabbyBird;

import android.app.Activity;
import android.os.Bundle;
import android.view.Window;
import android.view.WindowManager;

public class MainActivity extends Activity
{
    GameFlabbyBird mGame;

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        mGame = new GameFlabbyBird(this);
        setContentView(mGame);

    }

}

```


### 代码下载


[代码下载](http://download.csdn.net/detail/lmj623565791/8391667)


### 参考链接

[Android SurfaceView实战 带你玩转flabby bird （上） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/42965779)


[Android SurfaceView实战 带你玩转flabby bird （下） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/43063331)


