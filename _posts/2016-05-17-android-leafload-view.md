---
layout: post
title: 一个绚丽的loading动效分析与实现！
date: 2016-5-17
categories: blog
tags: [自定义控件]
description: 一个绚丽的loading动效分析与实现！
---   

### 最终效果如下 

![](http://img.blog.csdn.net/20150322173842391?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



从效果上看，我们需要考虑以下几个问题：               
1.叶子的随机产生；               
2.叶子随着一条正余弦曲线移动；                 
3.叶子在移动的时候旋转，旋转方向随机，正时针或逆时针；      
4.叶子遇到进度条，似乎是融合进入；            
5.叶子不能超出最左边的弧角；              
7.叶子飘出时的角度不是一致，走的曲线的振幅也有差别，否则太有规律性，缺乏美感；                  

总的看起来，需要注意和麻烦的地方主要是以上几点，当然还有一些细节问题，比如最左边是圆弧等等；              
那接下来我们将效果进行分解，然后逐个击破：                 
整个效果来说，我们需要的图主要是飞动的小叶子和右边旋转的风扇，其他的部分都可以用色值进行绘制，当然我们为了方便，就连底部框一起切了；                
先从gif 图里把飞动的小叶子和右边旋转的风扇、底部框抠出来，小叶子图如下：               


![](http://img.blog.csdn.net/20150322181205172?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


我们需要处理的主要有两个部分：              
1. 随着进度往前绘制的进度条；                     
2. 不断飞出来的小叶片；                      

我们先处理第一部分 － 随着进度往前绘制的进度条：          
进度条的位置根据外层传入的 progress 进行计算，可以分为图中 1、2、3 三个阶段：            



![](http://img.blog.csdn.net/20150322215713127?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


1. 当progress 较小，算出的当前距离还在弧形以内时，需要绘制如图所示 1 区域的弧形，其余部分用白色填充；             
2. 当 progress 算出的距离到2时，需要绘制棕色半圆弧形，其余部分用白色矩形填充；                   
3. 当 progress 算出的距离到3 时，需要绘制棕色半圆弧形，棕色矩形，白色矩形；                                
4. 当 progress 算出的距离到头时，需要绘制棕色半圆弧形，棕色矩形；（可以合并到3中）                             

首先根据进度条的宽度和当前进度、总进度算出当前的位置：      


```
//mProgressWidth为进度条的宽度，根据当前进度算出进度条的位置  
mCurrentProgressPosition = mProgressWidth * mProgress / TOTAL_PROGRESS;  
```


然后按照上面的逻辑进行绘制，其中需要计算上图中的红色弧角角度，计算方法如下：

```
// 单边角度  
int angle = (int) Math.toDegrees(Math.acos((mArcRadius - mCurrentProgressPosition)/ (float) mArcRadius));  
```

Math.acos()  －反余弦函数；
Math.toDegrees() － 弧度转化为角度，Math.toRadians 角度转化为弧度

所以圆弧的起始点为：

```
int startAngle = 180 - angle;  
```

圆弧划过的角度为：2 * angle  

这一块的代码如下：


```
// mProgressWidth为进度条的宽度，根据当前进度算出进度条的位置  
mCurrentProgressPosition = mProgressWidth * mProgress / TOTAL_PROGRESS;  
// 即当前位置在图中所示1范围内  
if (mCurrentProgressPosition < mArcRadius) {  
    Log.i(TAG, "mProgress = " + mProgress + "---mCurrentProgressPosition = "  
            + mCurrentProgressPosition  
            + "--mArcProgressWidth" + mArcRadius);  
    // 1.绘制白色ARC，绘制orange ARC  
    // 2.绘制白色矩形  
  
    // 1.绘制白色ARC  
    canvas.drawArc(mArcRectF, 90, 180, false, mWhitePaint);  
  
    // 2.绘制白色矩形  
    mWhiteRectF.left = mArcRightLocation;  
    canvas.drawRect(mWhiteRectF, mWhitePaint);  
  
    // 3.绘制棕色 ARC  
    // 单边角度  
    int angle = (int) Math.toDegrees(Math.acos((mArcRadius - mCurrentProgressPosition)  
            / (float) mArcRadius));  
    // 起始的位置  
    int startAngle = 180 - angle;  
    // 扫过的角度  
    int sweepAngle = 2 * angle;  
    Log.i(TAG, "startAngle = " + startAngle);  
    canvas.drawArc(mArcRectF, startAngle, sweepAngle, false, mOrangePaint);  
} else {  
    Log.i(TAG, "mProgress = " + mProgress + "---transfer-----mCurrentProgressPosition = "  
            + mCurrentProgressPosition  
            + "--mArcProgressWidth" + mArcRadius);  
    // 1.绘制white RECT  
    // 2.绘制Orange ARC  
    // 3.绘制orange RECT  
     
    // 1.绘制white RECT  
    mWhiteRectF.left = mCurrentProgressPosition;  
    canvas.drawRect(mWhiteRectF, mWhitePaint);  
      
    // 2.绘制Orange ARC  
    canvas.drawArc(mArcRectF, 90, 180, false, mOrangePaint);  
    // 3.绘制orange RECT  
    mOrangeRectF.left = mArcRightLocation;  
    mOrangeRectF.right = mCurrentProgressPosition;  
    canvas.drawRect(mOrangeRectF, mOrangePaint);  
  
}  
```


### 叶子部分


首先根据效果情况基本确定出 曲线函数，标准函数方程为：y = A(wx+Q)+h，其中w影响周期，A影响振幅 ，周期T＝ 2 * Math.PI/w;
根据效果可以看出，周期大致为总进度长度，所以确定w＝(float) ((float) 2 * Math.PI /mProgressWidth)；

仔细观察效果，我们可以发现，叶子飘动的过程中振幅不是完全一致的，产生一种错落的效果，既然如此，我们给叶子定义一个Type，根据Type 确定不同的振幅；            

我们创建一个叶子对象：

```
private class Leaf {  
  
     // 在绘制部分的位置  
     float x, y;  
     // 控制叶子飘动的幅度  
     StartType type;  
     // 旋转角度  
     int rotateAngle;  
     // 旋转方向--0代表顺时针，1代表逆时针  
     int rotateDirection;  
     // 起始时间(ms)  
     long startTime;  
 }  
```

类型采用枚举进行定义，其实就是用来区分不同的振幅：

```
private enum StartType {  
    LITTLE, MIDDLE, BIG  
}  
```


创建一个LeafFactory类用于创建一个或多个叶子信息：


```
private class LeafFactory {  
    private static final int MAX_LEAFS = 6;  
    Random random = new Random();  
  
    // 生成一个叶子信息  
    public Leaf generateLeaf() {  
        Leaf leaf = new Leaf();  
        int randomType = random.nextInt(3);  
        // 随时类型－ 随机振幅  
        StartType type = StartType.MIDDLE;  
        switch (randomType) {  
            case 0:  
                break;  
            case 1:  
                type = StartType.LITTLE;  
                break;  
            case 2:  
                type = StartType.BIG;  
                break;  
            default:  
                break;  
        }  
        leaf.type = type;  
        // 随机起始的旋转角度  
        leaf.rotateAngle = random.nextInt(360);  
        // 随机旋转方向（顺时针或逆时针）  
        leaf.rotateDirection = random.nextInt(2);  
        // 为了产生交错的感觉，让开始的时间有一定的随机性  
        mAddTime += random.nextInt((int) (LEAF_FLOAT_TIME * 1.5));  
        leaf.startTime = System.currentTimeMillis() + mAddTime;  
        return leaf;  
    }  
  
    // 根据最大叶子数产生叶子信息  
    public List<Leaf> generateLeafs() {  
        return generateLeafs(MAX_LEAFS);  
    }  
  
    // 根据传入的叶子数量产生叶子信息  
    public List<Leaf> generateLeafs(int leafSize) {  
        List<Leaf> leafs = new LinkedList<Leaf>();  
        for (int i = 0; i < leafSize; i++) {  
            leafs.add(generateLeaf());  
        }  
        return leafs;  
    }  
```


定义两个常亮分别记录中等振幅和之间的振幅差：


```
// 中等振幅大小  
private static final int MIDDLE_AMPLITUDE = 13;  
// 不同类型之间的振幅差距  
private static final int AMPLITUDE_DISPARITY = 5;  
```


```
// 中等振幅大小  
private int mMiddleAmplitude = MIDDLE_AMPLITUDE;  
// 振幅差  
private int mAmplitudeDisparity = AMPLITUDE_DISPARITY;  
```


有了以上信息，我们则可以获取到叶子的Y值：


```
// 通过叶子信息获取当前叶子的Y值  
private int getLocationY(Leaf leaf) {  
    // y = A(wx+Q)+h  
    float w = (float) ((float) 2 * Math.PI / mProgressWidth);  
    float a = mMiddleAmplitude;  
    switch (leaf.type) {  
        case LITTLE:  
            // 小振幅 ＝ 中等振幅 － 振幅差  
            a = mMiddleAmplitude - mAmplitudeDisparity;  
            break;  
        case MIDDLE:  
            a = mMiddleAmplitude;  
            break;  
        case BIG:  
            // 小振幅 ＝ 中等振幅 + 振幅差  
            a = mMiddleAmplitude + mAmplitudeDisparity;  
            break;  
        default:  
            break;  
    }  
    Log.i(TAG, "---a = " + a + "---w = " + w + "--leaf.x = " + leaf.x);  
    return (int) (a * Math.sin(w * leaf.x)) + mArcRadius * 2 / 3;  
}  
```



### 绘制叶子

```
/**  
 * 绘制叶子  
 *   
 * @param canvas  
 */  
private void drawLeafs(Canvas canvas) {  
    long currentTime = System.currentTimeMillis();  
    for (int i = 0; i < mLeafInfos.size(); i++) {  
        Leaf leaf = mLeafInfos.get(i);  
        if (currentTime > leaf.startTime && leaf.startTime != 0) {  
            // 绘制叶子－－根据叶子的类型和当前时间得出叶子的（x，y）  
            getLeafLocation(leaf, currentTime);  
            // 根据时间计算旋转角度  
            canvas.save();  
            // 通过Matrix控制叶子旋转  
            Matrix matrix = new Matrix();  
            float transX = mLeftMargin + leaf.x;  
            float transY = mLeftMargin + leaf.y;  
            Log.i(TAG, "left.x = " + leaf.x + "--leaf.y=" + leaf.y);  
            matrix.postTranslate(transX, transY);  
            // 通过时间关联旋转角度，则可以直接通过修改LEAF_ROTATE_TIME调节叶子旋转快慢  
            float rotateFraction = ((currentTime - leaf.startTime) % LEAF_ROTATE_TIME)  
                    / (float) LEAF_ROTATE_TIME;  
            int angle = (int) (rotateFraction * 360);  
            // 根据叶子旋转方向确定叶子旋转角度  
            int rotate = leaf.rotateDirection == 0 ? angle + leaf.rotateAngle : -angle  
                    + leaf.rotateAngle;  
            matrix.postRotate(rotate, transX  
                    + mLeafWidth / 2, transY + mLeafHeight / 2);  
            canvas.drawBitmap(mLeafBitmap, matrix, mBitmapPaint);  
            canvas.restore();  
        } else {  
            continue;  
        }  
    }  
}  
```


### LeafLoadingView完整代码  

```

package com.example.csdnblog4;

import java.util.LinkedList;
import java.util.List;
import java.util.Random;

import android.content.Context;
import android.content.res.Resources;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Matrix;
import android.graphics.Paint;
import android.graphics.Rect;
import android.graphics.RectF;
import android.graphics.drawable.BitmapDrawable;
import android.util.AttributeSet;
import android.util.Log;
import android.view.View;

public class LeafLoadingView extends View {

    private static final String TAG = "LeafLoadingView";
    // 淡白色
    private static final int WHITE_COLOR = 0xfffde399;
    // 橙色
    private static final int ORANGE_COLOR = 0xffffa800;
    // 中等振幅大小
    private static final int MIDDLE_AMPLITUDE = 13;
    // 不同类型之间的振幅差距
    private static final int AMPLITUDE_DISPARITY = 5;

    // 总进度
    private static final int TOTAL_PROGRESS = 100;
    // 叶子飘动一个周期所花的时间
    private static final long LEAF_FLOAT_TIME = 3000;
    // 叶子旋转一周需要的时间
    private static final long LEAF_ROTATE_TIME = 2000;

    // 用于控制绘制的进度条距离左／上／下的距离
    private static final int LEFT_MARGIN = 9;
    // 用于控制绘制的进度条距离右的距离
    private static final int RIGHT_MARGIN = 25;
    private int mLeftMargin, mRightMargin;
    // 中等振幅大小
    private int mMiddleAmplitude = MIDDLE_AMPLITUDE;
    // 振幅差
    private int mAmplitudeDisparity = AMPLITUDE_DISPARITY;

    // 叶子飘动一个周期所花的时间
    private long mLeafFloatTime = LEAF_FLOAT_TIME;
    // 叶子旋转一周需要的时间
    private long mLeafRotateTime = LEAF_ROTATE_TIME;
    private Resources mResources;
    private Bitmap mLeafBitmap;
    private int mLeafWidth, mLeafHeight;

    private Bitmap mOuterBitmap;
    private Rect mOuterSrcRect, mOuterDestRect;
    private int mOuterWidth, mOuterHeight;

    private int mTotalWidth, mTotalHeight;

    private Paint mBitmapPaint, mWhitePaint, mOrangePaint;
    private RectF mWhiteRectF, mOrangeRectF, mArcRectF;
    // 当前进度
    private int mProgress;
    // 所绘制的进度条部分的宽度
    private int mProgressWidth;
    // 当前所在的绘制的进度条的位置
    private int mCurrentProgressPosition;
    // 弧形的半径
    private int mArcRadius;

    // arc的右上角的x坐标，也是矩形x坐标的起始点
    private int mArcRightLocation;
    // 用于产生叶子信息
    private LeafFactory mLeafFactory;
    // 产生出的叶子信息
    private List<Leaf> mLeafInfos;
    // 用于控制随机增加的时间不抱团
    private int mAddTime;

    public LeafLoadingView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mResources = getResources();
        mLeftMargin = UiUtils.dipToPx(context, LEFT_MARGIN);
        mRightMargin = UiUtils.dipToPx(context, RIGHT_MARGIN);

        mLeafFloatTime = LEAF_FLOAT_TIME;
        mLeafRotateTime = LEAF_ROTATE_TIME;

        initBitmap();
        initPaint();
        mLeafFactory = new LeafFactory();
        mLeafInfos = mLeafFactory.generateLeafs();

    }

    private void initPaint() {
        mBitmapPaint = new Paint();
        mBitmapPaint.setAntiAlias(true);
        mBitmapPaint.setDither(true);
        mBitmapPaint.setFilterBitmap(true);

        mWhitePaint = new Paint();
        mWhitePaint.setAntiAlias(true);
        mWhitePaint.setColor(WHITE_COLOR);

        mOrangePaint = new Paint();
        mOrangePaint.setAntiAlias(true);
        mOrangePaint.setColor(ORANGE_COLOR);
        
        
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        // 绘制进度条和叶子
        // 之所以把叶子放在进度条里绘制，主要是层级原因
        drawProgressAndLeafs(canvas);
        // drawLeafs(canvas);

        canvas.drawBitmap(mOuterBitmap, mOuterSrcRect, mOuterDestRect, mBitmapPaint);

        postInvalidate();
    }

    private void drawProgressAndLeafs(Canvas canvas) {

        if (mProgress >= TOTAL_PROGRESS) {
            mProgress = 0;
        }
        // mProgressWidth为进度条的宽度，根据当前进度算出进度条的位置
        mCurrentProgressPosition = mProgressWidth * mProgress / TOTAL_PROGRESS;
        // 即当前位置在图中所示1范围内
        if (mCurrentProgressPosition < mArcRadius) {
            Log.i(TAG, "mProgress = " + mProgress + "---mCurrentProgressPosition = "
                    + mCurrentProgressPosition
                    + "--mArcProgressWidth" + mArcRadius);
            // 1.绘制白色ARC，绘制orange ARC
            // 2.绘制白色矩形

            // 1.绘制白色ARC
            canvas.drawArc(mArcRectF, 90, 180, false, mWhitePaint);

            // 2.绘制白色矩形
            mWhiteRectF.left = mArcRightLocation;
            canvas.drawRect(mWhiteRectF, mWhitePaint);

            // 绘制叶子
            drawLeafs(canvas);

            // 3.绘制棕色 ARC
            // 单边角度
            int angle = (int) Math.toDegrees(Math.acos((mArcRadius - mCurrentProgressPosition)
                    / (float) mArcRadius));
            // 起始的位置
            int startAngle = 180 - angle;
            
            // 扫过的角度
            int sweepAngle = 2 * angle;
            
            canvas.drawArc(mArcRectF, startAngle, sweepAngle, false, mOrangePaint);
        } else {
            Log.i(TAG, "mProgress = " + mProgress + "---transfer-----mCurrentProgressPosition = "
                    + mCurrentProgressPosition
                    + "--mArcProgressWidth" + mArcRadius);
            // 1.绘制white RECT
            // 2.绘制Orange ARC
            // 3.绘制orange RECT
            // 这个层级进行绘制能让叶子感觉是融入棕色进度条中

            // 1.绘制white RECT
            mWhiteRectF.left = mCurrentProgressPosition;
            canvas.drawRect(mWhiteRectF, mWhitePaint);
            // 绘制叶子
            drawLeafs(canvas);
            // 2.绘制Orange ARC
            canvas.drawArc(mArcRectF, 90, 180, false, mOrangePaint);
            // 3.绘制orange RECT
            mOrangeRectF.left = mArcRightLocation;
            mOrangeRectF.right = mCurrentProgressPosition;
            canvas.drawRect(mOrangeRectF, mOrangePaint);

        }

    }

    /**
     * 绘制叶子
     * 
     * @param canvas
     */
    private void drawLeafs(Canvas canvas) {
        mLeafRotateTime = mLeafRotateTime <= 0 ? LEAF_ROTATE_TIME : mLeafRotateTime;
        long currentTime = System.currentTimeMillis();
        for (int i = 0; i < mLeafInfos.size(); i++) {
            Leaf leaf = mLeafInfos.get(i);
            if (currentTime > leaf.startTime && leaf.startTime != 0) {
                // 绘制叶子－－根据叶子的类型和当前时间得出叶子的（x，y）
                getLeafLocation(leaf, currentTime);
                // 根据时间计算旋转角度
                canvas.save();
                // 通过Matrix控制叶子旋转
                Matrix matrix = new Matrix();
                float transX = mLeftMargin + leaf.x;
                float transY = mLeftMargin + leaf.y;
                Log.i(TAG, "left.x = " + leaf.x + "--leaf.y=" + leaf.y);
                matrix.postTranslate(transX, transY);
                // 通过时间关联旋转角度，则可以直接通过修改LEAF_ROTATE_TIME调节叶子旋转快慢
                float rotateFraction = ((currentTime - leaf.startTime) % mLeafRotateTime)
                        / (float) mLeafRotateTime;
                int angle = (int) (rotateFraction * 360);
                // 根据叶子旋转方向确定叶子旋转角度
                int rotate = leaf.rotateDirection == 0 ? angle + leaf.rotateAngle : -angle
                        + leaf.rotateAngle;
                matrix.postRotate(rotate, transX
                        + mLeafWidth / 2, transY + mLeafHeight / 2);
                canvas.drawBitmap(mLeafBitmap, matrix, mBitmapPaint);
                canvas.restore();
            } else {
                continue;
            }
        }
    }

    private void getLeafLocation(Leaf leaf, long currentTime) {
        long intervalTime = currentTime - leaf.startTime;
        mLeafFloatTime = mLeafFloatTime <= 0 ? LEAF_FLOAT_TIME : mLeafFloatTime;
        if (intervalTime < 0) {
            return;
        } else if (intervalTime > mLeafFloatTime) {
            leaf.startTime = System.currentTimeMillis()
                    + new Random().nextInt((int) mLeafFloatTime);
        }

        float fraction = (float) intervalTime / mLeafFloatTime;
        leaf.x = (int) (mProgressWidth - mProgressWidth * fraction);
        leaf.y = getLocationY(leaf);
    }

    // 通过叶子信息获取当前叶子的Y值
    private int getLocationY(Leaf leaf) {
        // y = A(wx+Q)+h
        float w = (float) ((float) 2 * Math.PI / mProgressWidth);
        float a = mMiddleAmplitude;
        switch (leaf.type) {
            case LITTLE:
                // 小振幅 ＝ 中等振幅 － 振幅差
                a = mMiddleAmplitude - mAmplitudeDisparity;
                break;
            case MIDDLE:
                a = mMiddleAmplitude;
                break;
            case BIG:
                // 小振幅 ＝ 中等振幅 + 振幅差
                a = mMiddleAmplitude + mAmplitudeDisparity;
                break;
            default:
                break;
        }
        Log.i(TAG, "---a = " + a + "---w = " + w + "--leaf.x = " + leaf.x);
        return (int) (a * Math.sin(w * leaf.x)) + mArcRadius * 2 / 3;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        
       Log.i("test", "widthMeasureSpec=="+MeasureSpec.getSize(widthMeasureSpec)+",heightMeasureSpec=="+MeasureSpec.getSize(heightMeasureSpec));
       int w=MeasureSpec.getSize(widthMeasureSpec);
       
       int h=MeasureSpec.getSize(heightMeasureSpec);
       
       mTotalWidth = w;
       mTotalHeight = h;
      
       mProgressWidth = mTotalWidth - mLeftMargin - mRightMargin;
       mArcRadius = (mTotalHeight - 2 * mLeftMargin) / 2;

       mOuterSrcRect = new Rect(0, 0, mOuterWidth, mOuterHeight);
       mOuterDestRect = new Rect(0, 0, mTotalWidth, mTotalHeight);

       mWhiteRectF = new RectF(mLeftMargin + mCurrentProgressPosition, mLeftMargin, mTotalWidth
               - mRightMargin,
               mTotalHeight - mLeftMargin);
       mOrangeRectF = new RectF(mLeftMargin + mArcRadius, mLeftMargin,
               mCurrentProgressPosition
               , mTotalHeight - mLeftMargin);

       mArcRectF = new RectF(mLeftMargin, mLeftMargin, mLeftMargin + 2 * mArcRadius,
               mTotalHeight - mLeftMargin);
       mArcRightLocation = mLeftMargin + mArcRadius;
       
       
    }

    private void initBitmap() {
        mLeafBitmap = ((BitmapDrawable) mResources.getDrawable(R.drawable.leaf)).getBitmap();
        mLeafWidth = mLeafBitmap.getWidth();
        mLeafHeight = mLeafBitmap.getHeight();

        mOuterBitmap = ((BitmapDrawable) mResources.getDrawable(R.drawable.leaf_kuang)).getBitmap();
        mOuterWidth = mOuterBitmap.getWidth();
        mOuterHeight = mOuterBitmap.getHeight();
    }

   /* @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mTotalWidth = w;
        mTotalHeight = h;
        Log.i("test", "mTotalWidth = " + w+",mTotalHeight=="+h+"getWidth()=="+getWidth()+",getHeight"+getHeight());
        mProgressWidth = mTotalWidth - mLeftMargin - mRightMargin;
        mArcRadius = (mTotalHeight - 2 * mLeftMargin) / 2;

        mOuterSrcRect = new Rect(0, 0, mOuterWidth, mOuterHeight);
        mOuterDestRect = new Rect(0, 0, mTotalWidth, mTotalHeight);

        mWhiteRectF = new RectF(mLeftMargin + mCurrentProgressPosition, mLeftMargin, mTotalWidth
                - mRightMargin,
                mTotalHeight - mLeftMargin);
        mOrangeRectF = new RectF(mLeftMargin + mArcRadius, mLeftMargin,
                mCurrentProgressPosition
                , mTotalHeight - mLeftMargin);

        mArcRectF = new RectF(mLeftMargin, mLeftMargin, mLeftMargin + 2 * mArcRadius,
                mTotalHeight - mLeftMargin);
        mArcRightLocation = mLeftMargin + mArcRadius;
    }*/

    private enum StartType {
        LITTLE, MIDDLE, BIG
    }

    /**
     * 叶子对象，用来记录叶子主要数据
     * 
     * @author Ajian_Studio
     */
    private class Leaf {

        // 在绘制部分的位置
        float x, y;
        // 控制叶子飘动的幅度
        StartType type;
        // 旋转角度
        int rotateAngle;
        // 旋转方向--0代表顺时针，1代表逆时针
        int rotateDirection;
        // 起始时间(ms)
        long startTime;
    }

    private class LeafFactory {
        private static final int MAX_LEAFS = 8;
        Random random = new Random();

        // 生成一个叶子信息
        public Leaf generateLeaf() {
            Leaf leaf = new Leaf();
            int randomType = random.nextInt(3);
            // 随时类型－ 随机振幅
            StartType type = StartType.MIDDLE;
            switch (randomType) {
                case 0:
                    break;
                case 1:
                    type = StartType.LITTLE;
                    break;
                case 2:
                    type = StartType.BIG;
                    break;
                default:
                    break;
            }
            leaf.type = type;
            // 随机起始的旋转角度
            leaf.rotateAngle = random.nextInt(360);
            // 随机旋转方向（顺时针或逆时针）
            leaf.rotateDirection = random.nextInt(2);
            // 为了产生交错的感觉，让开始的时间有一定的随机性
            mLeafFloatTime = mLeafFloatTime <= 0 ? LEAF_FLOAT_TIME : mLeafFloatTime;
            mAddTime += random.nextInt((int) (mLeafFloatTime * 2));
            leaf.startTime = System.currentTimeMillis() + mAddTime;
            return leaf;
        }

        // 根据最大叶子数产生叶子信息
        public List<Leaf> generateLeafs() {
            return generateLeafs(MAX_LEAFS);
        }

        // 根据传入的叶子数量产生叶子信息
        public List<Leaf> generateLeafs(int leafSize) {
            List<Leaf> leafs = new LinkedList<Leaf>();
            for (int i = 0; i < leafSize; i++) {
                leafs.add(generateLeaf());
            }
            return leafs;
        }
    }

    /**
     * 设置中等振幅
     * 
     * @param amplitude
     */
    public void setMiddleAmplitude(int amplitude) {
        this.mMiddleAmplitude = amplitude;
    }

    /**
     * 设置振幅差
     * 
     * @param disparity
     */
    public void setMplitudeDisparity(int disparity) {
        this.mAmplitudeDisparity = disparity;
    }

    /**
     * 获取中等振幅
     * 
     * @param amplitude
     */
    public int getMiddleAmplitude() {
        return mMiddleAmplitude;
    }

    /**
     * 获取振幅差
     * 
     * @param disparity
     */
    public int getMplitudeDisparity() {
        return mAmplitudeDisparity;
    }

    /**
     * 设置进度
     * 
     * @param progress
     */
    public void setProgress(int progress) {
        this.mProgress = progress;
        postInvalidate();
    }

    /**
     * 设置叶子飘完一个周期所花的时间
     * 
     * @param time
     */
    public void setLeafFloatTime(long time) {
        this.mLeafFloatTime = time;
    }

    /**
     * 设置叶子旋转一周所花的时间
     * 
     * @param time
     */
    public void setLeafRotateTime(long time) {
        this.mLeafRotateTime = time;
    }

    /**
     * 获取叶子飘完一个周期所花的时间
     */
    public long getLeafFloatTime() {
        mLeafFloatTime = mLeafFloatTime == 0 ? LEAF_FLOAT_TIME : mLeafFloatTime;
        return mLeafFloatTime;
    }

    /**
     * 获取叶子旋转一周所花的时间
     */
    public long getLeafRotateTime() {
        mLeafRotateTime = mLeafRotateTime == 0 ? LEAF_ROTATE_TIME : mLeafRotateTime;
        return mLeafRotateTime;
    }
}
```



### activity

```

package com.example.csdnblog4;

import java.util.Random;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.animation.Animation;
import android.view.animation.RotateAnimation;
import android.widget.Button;
import android.widget.SeekBar;
import android.widget.SeekBar.OnSeekBarChangeListener;
import android.widget.TextView;


public class LeafLoadingActivity extends Activity implements OnSeekBarChangeListener,
        OnClickListener {

    Handler mHandler = new Handler() {
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case REFRESH_PROGRESS:
                    if (mProgress < 40) {
                        mProgress += 1;
                        // 随机800ms以内刷新一次
                        mHandler.sendEmptyMessageDelayed(REFRESH_PROGRESS,
                                new Random().nextInt(800));
                        mLeafLoadingView.setProgress(mProgress);
                    } else {
                        mProgress += 1;
                        // 随机1200ms以内刷新一次
                        mHandler.sendEmptyMessageDelayed(REFRESH_PROGRESS,
                                new Random().nextInt(1200));
                        mLeafLoadingView.setProgress(mProgress);

                    }
                    break;

                default:
                    break;
            }
        };
    };

    private static final int REFRESH_PROGRESS = 0x10;
    private LeafLoadingView mLeafLoadingView;
    private SeekBar mAmpireSeekBar;
    private SeekBar mDistanceSeekBar;
    private TextView mMplitudeText;
    private TextView mDisparityText;
    private View mFanView;
    private Button mClearButton;
    private int mProgress = 0;

    private TextView mProgressText;
    private View mAddProgress;
    private SeekBar mFloatTimeSeekBar;

    private SeekBar mRotateTimeSeekBar;
    private TextView mFloatTimeText;
    private TextView mRotateTimeText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.leaf_loading_layout);
        initViews();
        mHandler.sendEmptyMessageDelayed(REFRESH_PROGRESS, 3000);
    }

    private void initViews() {
        mFanView = findViewById(R.id.fan_pic);
        RotateAnimation rotateAnimation = AnimationUtils.initRotateAnimation(false, 1500, true,
                Animation.INFINITE);
        mFanView.startAnimation(rotateAnimation);
        mClearButton = (Button) findViewById(R.id.clear_progress);
        mClearButton.setOnClickListener(this);

        mLeafLoadingView = (LeafLoadingView) findViewById(R.id.leaf_loading);
        mMplitudeText = (TextView) findViewById(R.id.text_ampair);
        mMplitudeText.setText(getString(R.string.current_mplitude,
                mLeafLoadingView.getMiddleAmplitude()));

        mDisparityText = (TextView) findViewById(R.id.text_disparity);
        mDisparityText.setText(getString(R.string.current_Disparity,
                mLeafLoadingView.getMplitudeDisparity()));

        mAmpireSeekBar = (SeekBar) findViewById(R.id.seekBar_ampair);
        mAmpireSeekBar.setOnSeekBarChangeListener(this);
        mAmpireSeekBar.setProgress(mLeafLoadingView.getMiddleAmplitude());
        mAmpireSeekBar.setMax(50);

        mDistanceSeekBar = (SeekBar) findViewById(R.id.seekBar_distance);
        mDistanceSeekBar.setOnSeekBarChangeListener(this);
        mDistanceSeekBar.setProgress(mLeafLoadingView.getMplitudeDisparity());
        mDistanceSeekBar.setMax(20);

        mAddProgress = findViewById(R.id.add_progress);
        mAddProgress.setOnClickListener(this);
        mProgressText = (TextView) findViewById(R.id.text_progress);

        mFloatTimeText = (TextView) findViewById(R.id.text_float_time);
        mFloatTimeSeekBar = (SeekBar) findViewById(R.id.seekBar_float_time);
        mFloatTimeSeekBar.setOnSeekBarChangeListener(this);
        mFloatTimeSeekBar.setMax(5000);
        mFloatTimeSeekBar.setProgress((int) mLeafLoadingView.getLeafFloatTime());
        mFloatTimeText.setText(getResources().getString(R.string.current_float_time,
                mLeafLoadingView.getLeafFloatTime()));

        mRotateTimeText = (TextView) findViewById(R.id.text_rotate_time);
        mRotateTimeSeekBar = (SeekBar) findViewById(R.id.seekBar_rotate_time);
        mRotateTimeSeekBar.setOnSeekBarChangeListener(this);
        mRotateTimeSeekBar.setMax(5000);
        mRotateTimeSeekBar.setProgress((int) mLeafLoadingView.getLeafRotateTime());
        mRotateTimeText.setText(getResources().getString(R.string.current_float_time,
                mLeafLoadingView.getLeafRotateTime()));
    }

    @Override
    public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
        if (seekBar == mAmpireSeekBar) {
            mLeafLoadingView.setMiddleAmplitude(progress);
            mMplitudeText.setText(getString(R.string.current_mplitude,
                    progress));
        } else if (seekBar == mDistanceSeekBar) {
            mLeafLoadingView.setMplitudeDisparity(progress);
            mDisparityText.setText(getString(R.string.current_Disparity,
                    progress));
        } else if (seekBar == mFloatTimeSeekBar) {
            mLeafLoadingView.setLeafFloatTime(progress);
            mFloatTimeText.setText(getResources().getString(R.string.current_float_time,
                    progress));
        }
        else if (seekBar == mRotateTimeSeekBar) {
            mLeafLoadingView.setLeafRotateTime(progress);
            mRotateTimeText.setText(getResources().getString(R.string.current_rotate_time,
                    progress));
        }

    }

    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {

    }

    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {

    }

    @Override
    public void onClick(View v) {
        if (v == mClearButton) {
            mLeafLoadingView.setProgress(0);
            mHandler.removeCallbacksAndMessages(null);
            mProgress = 0;
        } else if (v == mAddProgress) {
            mProgress++;
            mLeafLoadingView.setProgress(mProgress);
            mProgressText.setText(String.valueOf(mProgress));
        }
    }
}
```

### layout

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#fed255"
    android:orientation="vertical" >

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="100dp"
        android:text="loading ..."
        android:textColor="#FFA800"
        android:textSize=" 30dp" />

    <RelativeLayout
        android:id="@+id/leaf_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="50dp" >

        <com.example.csdnblog4.LeafLoadingView
            android:id="@+id/leaf_loading"
            android:layout_width="302dp"
            android:layout_height="61dp"
            android:layout_centerHorizontal="true" />

        <ImageView
            android:id="@+id/fan_pic"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:layout_centerVertical="true"
            android:layout_marginRight="35dp"
            android:src="@drawable/fengshan" />
    </RelativeLayout>

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent" >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical" >

            <LinearLayout
                android:id="@+id/seek_content_one"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="15dp" >

                <TextView
                    android:id="@+id/text_ampair"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:textColor="#ffffa800"
                    android:textSize="15dp" />

                <SeekBar
                    android:id="@+id/seekBar_ampair"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginLeft="5dp"
                    android:layout_weight="1" />
            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="15dp"
                android:orientation="horizontal" >

                <TextView
                    android:id="@+id/text_disparity"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:textColor="#ffffa800"
                    android:textSize="15dp" />

                <SeekBar
                    android:id="@+id/seekBar_distance"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginLeft="5dp"
                    android:layout_weight="1" />
            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="15dp"
                android:orientation="horizontal" >

                <TextView
                    android:id="@+id/text_float_time"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:textColor="#ffffa800"
                    android:textSize="15dp" />

                <SeekBar
                    android:id="@+id/seekBar_float_time"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginLeft="5dp"
                    android:layout_weight="1" />
            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="15dp"
                android:orientation="horizontal" >

                <TextView
                    android:id="@+id/text_rotate_time"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:textColor="#ffffa800"
                    android:textSize="15dp" />

                <SeekBar
                    android:id="@+id/seekBar_rotate_time"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginLeft="5dp"
                    android:layout_weight="1" />
            </LinearLayout>

            <Button
                android:id="@+id/clear_progress"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="15dp"
                android:text="去除进度条,玩转弧线"
                android:textSize="18dp" />

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="15dp"
                android:orientation="horizontal" >

                <Button
                    android:id="@+id/add_progress"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="增加进度: "
                    android:textSize="18dp" />

                <TextView
                    android:id="@+id/text_progress"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center_vertical"
                    android:textColor="#ffffa800"
                    android:textSize="15dp" />
            </LinearLayout>
        </LinearLayout>
    </ScrollView>

</LinearLayout>
```


### 注意 

本文中用到了onSizeChanged方法，这个方法在这是为了自适应界面，每次界面变化的时候都会被调用来重新计算一些参数，一般在刚进入与横竖屏切换时调用   

### 参考链接

[Android自定义View初步 - 泡在网上的日子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0606/3001.html)


### 本文中还用到了postInvalidate

invalidate()和postInvalidate() 的区别及使用，
Android提供了Invalidate方法实现界面刷新，但是Invalidate不能直接在线程中调用，因为他是违背了单线程模型：Android UI操作并不是线程安全的，并且这些操作必须在UI线程中调用。 
invalidate()是用来刷新View的，必须是在UI线程中进行工作。比如在修改某个view的显示时，调用invalidate()才能看到重新绘制的界面。invalidate()的调用是把之前的旧的view从主UI线程队列中pop掉。 一个Android 程序默认情况下也只有一个进程，但一个进程下却可以有许多个线程。                          
在这么多线程当中，把主要是负责控制UI界面的显示、更新和控件交互的线程称为UI线程，由于onCreate()方法是由UI线程执行的，所以也可以把UI线程理解为主线程。其余的线程可以理解为工作者线程。             
invalidate()得在UI线程中被调动，在工作者线程中可以通过Handler来通知UI线程进行界面更新。                   

而postInvalidate()在工作者线程中被调用            


### 参考链接  

[Android笔记：invalidate()和postInvalidate() 的区别及使用 - Mars2639——求知de路上 - 博客频道 - CSDN.NET](http://blog.csdn.net/mars2639/article/details/6650876)


### 本文还涉及到单位的转化

### 参考链接

[Android中dip、dp、sp、pt和px的区别 - 大气象 - 博客园](http://www.cnblogs.com/greatverve/archive/2011/12/28/android-dip-dp-sp-pt-px.html)


### 最终效果如下 

![](http://img.blog.csdn.net/20150323014910130?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGlhbmppYW40NTky/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


### 参考链接

[一个绚丽的loading动效分析与实现！ - Ajian_studio - 博客频道 - CSDN.NET](http://blog.csdn.net/tianjian4592/article/details/44538605)


### 源代码下载

[源代码](http://download.csdn.net/detail/tianjian4592/8524539)



### 完成
