---
layout: post
title: Android实现高仿QQ附近的人搜索展示
date: 2016-5-12
categories: blog
tags: [自定义控件]
description: Android实现高仿QQ附近的人搜索展示
---   

### 本文主要实现了高仿QQ附近的人搜索展示，用到了自定义控件的方法

### 最终效果如下

![](http://img.blog.csdn.net/20160505001804763)


1.下面展示列表我们可以使用ViewPager来实现（当然如果你不觉得麻烦，你也可以用HorizontalScrollView来试试）

2.上面的扫描图，肯定是个ViewGroup(因为里面的小圆点是可以点击的，如果是View的话，对于这些小圆点的位置的判断，以及对小圆点缩放动画的处理都会超级麻烦，难以实现)，所以我们肯定需要自定义ViewGroup

3.确定好了是自定义ViewGroup后，对于里面需要放什么对象呢？没错，就是N个小圆点+一个扫描的大圈圈。


### 下面的viewPager实现

### 自定义CustomViewPager

```
package com.zj.custom;

import android.content.Context;
import android.support.v4.view.ViewPager;
import android.util.AttributeSet;
import android.view.MotionEvent;

/**
 * com.zj.custom.CustomViewPager
 * Created by jjx on 2016/5/12.
 */
public class CustomViewPager extends ViewPager{
    private long downTime;
    private float LastX;
    private float mSpeed;


    public CustomViewPager(Context context) {
        super(context);
    }

    public CustomViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        float x = ev.getX();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downTime = System.currentTimeMillis();
                LastX = x;
                break;
            case MotionEvent.ACTION_MOVE:
                x = ev.getX();
                break;
            case MotionEvent.ACTION_UP:
                //计算得到手指从按下到离开的滑动速度
                mSpeed = (x - LastX) * 1000 / (System.currentTimeMillis() - downTime);
                break;
        }
        return super.dispatchTouchEvent(ev);
    }

    public float getSpeed() {
        return mSpeed;
    }

    public void setSpeed(float mSpeed) {
        this.mSpeed = mSpeed;
    }
}
```

### MainActivity实现  

```
package com.zj.modelqq;

import android.os.Bundle;
import android.os.Handler;
import android.support.v4.view.PagerAdapter;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.util.SparseArray;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;
import android.view.animation.AccelerateInterpolator;
import android.widget.ImageView;
import android.widget.RelativeLayout;
import android.widget.TextView;
import android.widget.Toast;

import com.zj.bean.Info;
import com.zj.custom.CustomViewPager;
import com.zj.custom.RadarViewGroup;
import com.zj.util.FixedSpeedScroller;
import com.zj.util.LogUtil;
import com.zj.util.ZoomOutPageTransformer;

import java.lang.reflect.Field;

public class MainActivity extends AppCompatActivity implements ViewPager.OnPageChangeListener, RadarViewGroup.IRadarClickListener {

    private CustomViewPager viewPager;
    private RelativeLayout ryContainer;
    private RadarViewGroup radarViewGroup;
    private int[] mImgs = {R.drawable.len, R.drawable.leo, R.drawable.lep,
            R.drawable.leq, R.drawable.ler, R.drawable.les, R.drawable.mln, R.drawable.mmz, R.drawable.mna,
            R.drawable.mnj, R.drawable.leo, R.drawable.leq, R.drawable.les, R.drawable.lep};
    private String[] mNames = {"ImmortalZ", "唐马儒", "王尼玛", "张全蛋", "蛋花", "王大锤", "叫兽", "哆啦A梦"};
    private int mPosition;
    private FixedSpeedScroller scroller;
    private SparseArray<Info> mDatas = new SparseArray<>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
        initData();

        /**
         * 将Viewpager所在容器的事件分发交给ViewPager
         */
        ryContainer.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                return viewPager.dispatchTouchEvent(event);
            }
        });
        ViewpagerAdapter mAdapter = new ViewpagerAdapter();
        viewPager.setAdapter(mAdapter);
        //设置缓存数为展示的数目
        viewPager.setOffscreenPageLimit(mImgs.length);
        viewPager.setPageMargin(getResources().getDimensionPixelOffset(R.dimen.viewpager_margin));
        //设置切换动画
        viewPager.setPageTransformer(true, new ZoomOutPageTransformer());
        viewPager.addOnPageChangeListener(this);
        setViewPagerSpeed(250);

        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                radarViewGroup.setDatas(mDatas);
            }
        }, 1500);
        radarViewGroup.setiRadarClickListener(this);
    }

    private void initData() {
        for (int i = 0; i < mImgs.length; i++) {
            Info info = new Info();
            info.setPortraitId(mImgs[i]);
            info.setAge(((int) Math.random() * 25 + 16) + "岁");
            info.setName(mNames[(int) (Math.random() * mNames.length)]);
            info.setSex(i % 3 == 0 ? false : true);
            info.setDistance(Math.round((Math.random() * 10) * 100) / 100);
            mDatas.put(i, info);
        }
    }

    private void initView() {
        viewPager = (CustomViewPager) findViewById(R.id.vp);
        radarViewGroup = (RadarViewGroup) findViewById(R.id.radar);
        ryContainer = (RelativeLayout) findViewById(R.id.ry_container);
    }

    /**
     * 设置ViewPager切换速度
     * 使用了反射机制
     * @param duration
     */
    private void setViewPagerSpeed(int duration) {
        try {
            Field field = ViewPager.class.getDeclaredField("mScroller");
            field.setAccessible(true);
            scroller = new FixedSpeedScroller(MainActivity.this, new AccelerateInterpolator());
            field.set(viewPager, scroller);
            scroller.setmDuration(duration);
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        mPosition = position;
    }

    @Override
    public void onPageSelected(int position) {
        //radarViewGroup.setCurrentShowItem(position);
        LogUtil.m("当前位置 " + mPosition);
        LogUtil.m("速度 " + viewPager.getSpeed());
        //当手指左滑速度大于2000时viewpager右滑（注意是item+2）
        if (viewPager.getSpeed() < -1800) {

            viewPager.setCurrentItem(mPosition + 2);
            LogUtil.m("位置 " + mPosition);
            viewPager.setSpeed(0);
        } else if (viewPager.getSpeed() > 1800 && mPosition > 0) {
            //当手指右滑速度大于2000时viewpager左滑（注意item-1即可）
            viewPager.setCurrentItem(mPosition - 1);
            LogUtil.m("位置 " + mPosition);
            viewPager.setSpeed(0);
        }
    }

    @Override
    public void onPageScrollStateChanged(int state) {

    }

    @Override
    public void onRadarItemClick(int position) {
        viewPager.setCurrentItem(position);
    }




    class ViewpagerAdapter extends PagerAdapter {
        @Override
        public Object instantiateItem(ViewGroup container, final int position) {
            final Info info = mDatas.get(position);
            //设置一大堆演示用的数据，麻里麻烦~~
            View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.viewpager_layout, null);
            ImageView ivPortrait = (ImageView) view.findViewById(R.id.iv);
            ImageView ivSex = (ImageView) view.findViewById(R.id.iv_sex);
            TextView tvName = (TextView) view.findViewById(R.id.tv_name);
            TextView tvDistance = (TextView) view.findViewById(R.id.tv_distance);
            tvName.setText(info.getName());
            tvDistance.setText(info.getDistance() + "km");
            ivPortrait.setImageResource(info.getPortraitId());
            if (info.getSex()) {
                ivSex.setImageResource(R.drawable.girl);
            } else {
                ivSex.setImageResource(R.drawable.boy);
            }
            ivPortrait.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Toast.makeText(MainActivity.this, "这是 " + info.getName() + " >.<", Toast.LENGTH_SHORT).show();
                }
            });
            container.addView(view);
            return view;
        }

        @Override
        public int getCount() {
            return mImgs.length;
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view == object;
        }

        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            View view = (View) object;
            container.removeView(view);
        }

    }
}
```

### 界面配置文件  

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/lkd"
    android:paddingLeft="5dp"
    android:paddingRight="5dp"
    tools:context="com.zj.modelqq.MainActivity">


    <com.zj.custom.RadarViewGroup
        android:id="@+id/radar"
        android:layout_width="280dp"
        android:layout_height="280dp"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="50dp">

        <com.zj.custom.RadarView
            android:id="@+id/id_scan_circle"
            android:layout_width="280dp"
            android:layout_height="280dp"/>
    </com.zj.custom.RadarViewGroup>

    <RelativeLayout
        android:id="@+id/ry_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="25dp"
        android:clipChildren="false">

        <com.zj.custom.CustomViewPager
            android:id="@+id/vp"
            android:layout_width="130dp"
            android:layout_height="160dp"
            android:layout_centerInParent="true"
            android:layout_marginLeft="120dp"
            android:layout_marginRight="120dp"
            />
    </RelativeLayout>
</RelativeLayout>
```

注意如果我们想要让ViewPager一次显示多个，需要设置其所在 父容器 Android:clipChildren=”false” 
意思就是不限制子View在其范围内。 
细心的你可能会发现MainAcitivity中有 
viewPager.setPageTransformer(true, new ZoomOutPageTransformer()); 
这个，没错，这个就是用来控制我们的切换动画（我在谷歌官方提供的这个基础上进行了修改，也是很好理解）

```
package com.zj.util;

import android.support.v4.view.ViewPager;
import android.view.View;

/**
 * Created by jjx on 2016/5/12.
 */
public class ZoomOutPageTransformer implements ViewPager.PageTransformer{

    private static final float MIN_SCALE = 0.70f;
    private static final float MIN_ALPHA = 0.5f;
    @Override
    public void transformPage(View view, float position) {
        int pageWidth = view.getWidth();
        int pageHeight = view.getHeight();
        if (position < -1) { // [-Infinity,-1)
            // This page is way off-screen to the left.
            view.setAlpha(MIN_ALPHA);
            view.setScaleX(MIN_SCALE);
            view.setScaleY(MIN_SCALE);
        } else if (position <= 1) { // [-1,1]
            // Modify the default slide transition to shrink the page as well
            float scaleFactor = Math.max(MIN_SCALE, 1 - Math.abs(position));
            float vertMargin = pageHeight * (1 - scaleFactor) / 2;
            float horzMargin = pageWidth * (1 - scaleFactor) / 2;
            if (position < 0) {
                view.setTranslationX(horzMargin - vertMargin / 2);
                view.setScaleX(1 + 0.3f * position);
                view.setScaleY(1 + 0.3f * position);
            } else {
                view.setTranslationX(-horzMargin + vertMargin / 2);

                view.setScaleX(1 - 0.3f * position);
                view.setScaleY(1 - 0.3f * position);
            }

            // Scale the page down (between MIN_SCALE and 1)

            // Fade the page relative to its size.
            view.setAlpha(MIN_ALPHA + (scaleFactor - MIN_SCALE) / (1 - MIN_SCALE) * (1 - MIN_ALPHA));

        } else { // (1,+Infinity]
            // This page is way off-screen to the right.
            view.setScaleX(MIN_SCALE);
            view.setScaleY(MIN_SCALE);
            view.setAlpha(MIN_ALPHA);
        }
    }
}
```

### viewpager效果如下

![](http://img.blog.csdn.net/20160505092328156)


### 实现雷达扫描图

代码中也注释得很清楚了，当然因为要扫描，我们需要不停的转动，所以这里我们用到了矩阵变换Matrix，扫描消息的停顿和传递我们用到了Runnable 
，如果要是觉得在向主线程一直投递变换的消息对主线程不好，你可以考虑下用SurfaceView来实现

### RadarView

```
package com.zj.custom;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Matrix;
import android.graphics.Paint;
import android.graphics.Rect;
import android.graphics.Shader;
import android.graphics.SweepGradient;
import android.util.AttributeSet;
import android.view.View;

import com.zj.modelqq.R;

/**
 * Created by jjx on 2016/5/12.
 */
public class RadarView extends View{

    private Paint mPaintLine;//画圆线需要用到的paint
    private Paint mPaintCircle;//画圆需要用到的paint
    private Paint mPaintScan;//画扫描需要用到的paint

    private int mWidth, mHeight;//整个图形的长度和宽度

    private Matrix matrix = new Matrix();//旋转需要的矩阵
    private int scanAngle;//扫描旋转的角度
    private Shader scanShader;//扫描渲染shader
    private Bitmap centerBitmap;//最中间icon

    //每个圆圈所占的比例
    private static float[] circleProportion = {1 / 13f, 2 / 13f, 3 / 13f, 4 / 13f, 5 / 13f, 6 / 13f};
    private int scanSpeed = 5;

    private int currentScanningCount;//当前扫描的次数
    private int currentScanningItem;//当前扫描显示的item
    private int maxScanItemCount;//最大扫描次数
    private boolean startScan = false;//只有设置了数据后才会开始扫描
    private IScanningListener iScanningListener;//扫描时监听回调接口

    public void setScanningListener(IScanningListener iScanningListener) {
        this.iScanningListener = iScanningListener;
    }

    private Runnable run = new Runnable() {
        @Override
        public void run() {
            scanAngle = (scanAngle + scanSpeed) % 360;
            matrix.postRotate(scanSpeed, mWidth / 2, mHeight / 2);
            invalidate();
            postDelayed(run, 130);
            //开始扫描显示标志为true 且 只扫描一圈
            if (startScan && currentScanningCount <= (360 / scanSpeed)) {
                if (iScanningListener != null && currentScanningCount % scanSpeed == 0
                        && currentScanningItem < maxScanItemCount) {

                    iScanningListener.onScanning(currentScanningItem, scanAngle);
                    currentScanningItem++;
                } else if (iScanningListener != null && currentScanningItem == maxScanItemCount) {
                    iScanningListener.onScanSuccess();
                }
                currentScanningCount++;
            }
        }
    };

    public RadarView(Context context) {
        this(context, null);
    }

    public RadarView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RadarView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
        post(run);
    }


    private void init() {
        mPaintLine = new Paint();
        mPaintLine.setColor(getResources().getColor(R.color.line_color_blue));
        mPaintLine.setAntiAlias(true);
        mPaintLine.setStrokeWidth(1);
        mPaintLine.setStyle(Paint.Style.STROKE);

        mPaintCircle = new Paint();
        mPaintCircle.setColor(Color.WHITE);
        mPaintCircle.setAntiAlias(true);

        mPaintScan = new Paint();
        mPaintScan.setStyle(Paint.Style.FILL_AND_STROKE);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(measureSize(widthMeasureSpec), measureSize(widthMeasureSpec));
        mWidth = getMeasuredWidth();
        mHeight = getMeasuredHeight();
        mWidth = mHeight = Math.min(mWidth, mHeight);
        centerBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.circle_photo);
        //设置扫描渲染的shader
        scanShader = new SweepGradient(mWidth / 2, mHeight / 2,
                new int[]{Color.TRANSPARENT, Color.parseColor("#84B5CA")}, null);
    }

    private int measureSize(int measureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else {
            result = 300;
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        drawCircle(canvas);
        drawScan(canvas);
        drawCenterIcon(canvas);
    }

    /**
     * 绘制圆线圈
     *
     * @param canvas
     */
    private void drawCircle(Canvas canvas) {
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[1], mPaintLine);     // 绘制小圆
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[2], mPaintLine);   // 绘制中圆
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[3], mPaintLine); // 绘制中大圆
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[4], mPaintLine);  // 绘制大圆
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[5], mPaintLine);  // 绘制大大圆
    }

    /**
     * 绘制扫描
     *
     * @param canvas
     */
    private void drawScan(Canvas canvas) {
        canvas.save();
        mPaintScan.setShader(scanShader);
        canvas.concat(matrix);
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[4], mPaintScan);
        canvas.restore();
    }

    /**
     * 绘制最中间的图标
     *
     * @param canvas
     */
    private void drawCenterIcon(Canvas canvas) {
        canvas.drawBitmap(centerBitmap, null,
                new Rect((int) (mWidth / 2 - mWidth * circleProportion[0]), (int) (mHeight / 2 - mWidth * circleProportion[0]),
                        (int) (mWidth / 2 + mWidth * circleProportion[0]), (int) (mHeight / 2 + mWidth * circleProportion[0])), mPaintCircle);
    }


    public interface IScanningListener {
        //正在扫描（此时还没有扫描完毕）时回调
        void onScanning(int position, float scanAngle);

        //扫描成功时回调
        void onScanSuccess();
    }

    public void setMaxScanItemCount(int maxScanItemCount) {
        this.maxScanItemCount = maxScanItemCount;
    }

    /**
     * 开始扫描
     */
    public void startScan() {
        this.startScan = true;
    }
}
```

### 小圆点CircleView

```
package com.zj.custom;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.Rect;
import android.util.AttributeSet;
import android.view.View;

import com.zj.modelqq.R;
import com.zj.util.DisplayUtils;

/**
 * Created by jjx on 2016/5/12.
 */
public class CircleView extends View{
    private Paint mPaint;
    private Bitmap mBitmap;
    private float radius = DisplayUtils.dp2px(getContext(), 9);//半径
    private float disX;//位置X
    private float disY;//位置Y
    private float angle;//旋转的角度
    private float proportion;//根据远近距离的不同计算得到的应该占的半径比例

    public float getProportion() {
        return proportion;
    }

    public void setProportion(float proportion) {
        this.proportion = proportion;
    }

    public float getAngle() {
        return angle;
    }

    public void setAngle(float angle) {
        this.angle = angle;
    }

    public float getDisX() {
        return disX;
    }

    public void setDisX(float disX) {
        this.disX = disX;
    }

    public float getDisY() {
        return disY;
    }

    public void setDisY(float disY) {
        this.disY = disY;
    }

    public CircleView(Context context) {
        this(context, null);
    }

    public CircleView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        mPaint = new Paint();
        mPaint.setColor(getResources().getColor(R.color.bg_color_pink));
        mPaint.setAntiAlias(true);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(measureSize(widthMeasureSpec), measureSize(heightMeasureSpec));
    }

    private int measureSize(int measureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else {
            result = DisplayUtils.dp2px(getContext(),18);
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;
    }


    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawCircle(radius, radius, radius, mPaint);
        if (mBitmap != null) {
            canvas.drawBitmap(mBitmap, null, new Rect(0, 0, 2 * (int) radius, 2 * (int) radius), mPaint);
        }
    }

    public void setPaintColor(int resId) {
        mPaint.setColor(resId);
        invalidate();
    }

    public void setPortraitIcon(int resId) {
        mBitmap = BitmapFactory.decodeResource(getResources(), resId);
        invalidate();
    }
    public void clearPortaitIcon(){
        mBitmap = null;
        invalidate();
    }
}
```

有了小圆点，我们最后只需要把扫描图和小圆点放在一起就好了 
因为我们是想变扫描变出现小圆点，所以我们需要在RadarView中定义一个接口IScanningListener，告诉RadarViewGroup我正在扫描，你快让小圆点出现吧 
所以在RadarViewGroup的onScanning中需要调用requestLayout();


```
package com.zj.custom;

import android.animation.ObjectAnimator;
import android.content.Context;
import android.util.AttributeSet;
import android.util.SparseArray;
import android.view.View;
import android.view.ViewGroup;

import com.zj.bean.Info;
import com.zj.modelqq.R;
import com.zj.util.LogUtil;

/**
 * Created by jjx on 2016/5/12.
 */
public class RadarViewGroup extends ViewGroup implements RadarView.IScanningListener {

    private int mWidth, mHeight;//viewgroup的宽高
    private SparseArray<Float> scanAngleList = new SparseArray<>();//记录展示的item所在的扫描位置角度
    private SparseArray<Info> mDatas;//数据源
    private int dataLength;//数据源长度
    private int minItemPosition;//最小距离的item所在数据源中的位置
    private CircleView currentShowChild;//当前展示的item
    private CircleView minShowChild;//最小距离的item
    private IRadarClickListener iRadarClickListener;//雷达图中点击监听CircleView小圆点回调接口

    public void setiRadarClickListener(IRadarClickListener iRadarClickListener) {
        this.iRadarClickListener = iRadarClickListener;
    }

    public RadarViewGroup(Context context) {
        this(context, null);
    }

    public RadarViewGroup(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public RadarViewGroup(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(measureSize(widthMeasureSpec), measureSize(heightMeasureSpec));
        mWidth = getMeasuredWidth();
        mHeight = getMeasuredHeight();
        mWidth = mHeight = Math.min(mWidth, mHeight);
        //测量每个children
        measureChildren(widthMeasureSpec, heightMeasureSpec);
        for (int i = 0; i < getChildCount(); i++) {
            View child = getChildAt(i);
            if (child.getId() == R.id.id_scan_circle) {
                //为雷达扫描图设置需要的属性
                ((RadarView) child).setScanningListener(this);
                //考虑到数据没有添加前扫描图在扫描，但是不会开始为CircleView布局
                if (mDatas != null && mDatas.size() > 0) {
                    ((RadarView) child).setMaxScanItemCount(mDatas.size());
                    ((RadarView) child).startScan();
                }
                continue;
            }
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childCount = getChildCount();
        //首先放置雷达扫描图
        View view = findViewById(R.id.id_scan_circle);
        if (view != null) {
            view.layout(0, 0, view.getMeasuredWidth(), view.getMeasuredHeight());
        }
        //放置雷达图中需要展示的item圆点
        for (int i = 0; i < childCount; i++) {
            final int j = i;
            final View child = getChildAt(i);
            if (child.getId() == R.id.id_scan_circle) {
                //如果不是Circleview跳过
                continue;
            }
            //设置CircleView小圆点的坐标信息
            //坐标 = 旋转角度 * 半径 * 根据远近距离的不同计算得到的应该占的半径比例
            ((CircleView) child).setDisX((float) Math.cos(Math.toRadians(scanAngleList.get(i - 1) - 5))
                    * ((CircleView) child).getProportion() * mWidth / 2);
            ((CircleView) child).setDisY((float) Math.sin(Math.toRadians(scanAngleList.get(i - 1) - 5))
                    * ((CircleView) child).getProportion() * mWidth / 2);
            //如果扫描角度记录SparseArray中的对应的item的值为0，
            // 说明还没有扫描到该item，跳过对该item的layout
            //（scanAngleList设置数据时全部设置的value=0，
            // 当onScanning时，value设置的值始终不会0，具体可以看onScanning中的实现）
            if (scanAngleList.get(i - 1) == 0) {
                continue;
            }
            //放置Circle小圆点
            child.layout((int) ((CircleView) child).getDisX() + mWidth / 2, (int) ((CircleView) child).getDisY() + mHeight / 2,
                    (int) ((CircleView) child).getDisX() + child.getMeasuredWidth() + mWidth / 2,
                    (int) ((CircleView) child).getDisY() + child.getMeasuredHeight() + mHeight / 2);
            //设置点击事件
            child.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    resetAnim(currentShowChild);
                    currentShowChild = (CircleView) child;
                    //因为雷达图是childAt(0),所以这里需要作-1才是正确的Circle
                    startAnim(currentShowChild, j - 1);
                    if (iRadarClickListener != null) {
                        iRadarClickListener.onRadarItemClick(j - 1);

                    }
                }
            });
        }


    }

    private int measureSize(int measureSpec) {
        int result = 0;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        if (specMode == MeasureSpec.EXACTLY) {
            result = specSize;
        } else {
            result = 300;
            if (specMode == MeasureSpec.AT_MOST) {
                result = Math.min(result, specSize);
            }
        }
        return result;

    }

    /**
     * 设置数据
     *
     * @param mDatas
     */
    public void setDatas(SparseArray<Info> mDatas) {
        this.mDatas = mDatas;
        dataLength = mDatas.size();
        float min = Float.MAX_VALUE;
        float max = Float.MIN_VALUE;
        //找到距离的最大值，最小值对应的minItemPosition
        for (int j = 0; j < dataLength; j++) {
            Info item = mDatas.get(j);
            if (item.getDistance() < min) {
                min = item.getDistance();
                minItemPosition = j;
            }
            if (item.getDistance() > max) {
                max = item.getDistance();
            }
            scanAngleList.put(j, 0f);
        }
        //根据数据源信息动态添加CircleView
        for (int i = 0; i < dataLength; i++) {
            CircleView circleView = new CircleView(getContext());
            if (mDatas.get(i).getSex()) {
                circleView.setPaintColor(getResources().getColor(R.color.bg_color_pink));
            } else {
                circleView.setPaintColor(getResources().getColor(R.color.bg_color_blue));
            }
            //根据远近距离的不同计算得到的应该占的半径比例 0.312-0.832
            circleView.setProportion((mDatas.get(i).getDistance() / max + 0.6f) * 0.52f);
            if (minItemPosition == i) {
                minShowChild = circleView;
            }
            addView(circleView);
        }
    }

    /**
     * 雷达图没有扫描完毕时回调
     *
     * @param position
     * @param scanAngle
     */
    @Override
    public void onScanning(int position, float scanAngle) {
        if (scanAngle == 0) {
            scanAngleList.put(position, 1f);
        } else {
            scanAngleList.put(position, scanAngle);
        }
        requestLayout();
    }

    /**
     * 雷达图扫描完毕时回调
     */
    @Override
    public void onScanSuccess() {
        LogUtil.m("完成回调");
        resetAnim(currentShowChild);
        currentShowChild = minShowChild;
        startAnim(currentShowChild, minItemPosition);
    }

    /**
     * 恢复CircleView小圆点原大小
     *
     * @param object
     */
    private void resetAnim(CircleView object) {
        if (object != null) {
            object.clearPortaitIcon();
            ObjectAnimator.ofFloat(object, "scaleX", 1f).setDuration(300).start();
            ObjectAnimator.ofFloat(object, "scaleY", 1f).setDuration(300).start();
        }

    }

    /**
     * 放大CircleView小圆点大小
     *
     * @param object
     * @param position
     */
    private void startAnim(CircleView object, int position) {
        if (object != null) {
            object.setPortraitIcon(mDatas.get(position).getPortraitId());
            ObjectAnimator.ofFloat(object, "scaleX", 2f).setDuration(300).start();
            ObjectAnimator.ofFloat(object, "scaleY", 2f).setDuration(300).start();
        }
    }

    /**
     * 雷达图中点击监听CircleView小圆点回调接口
     */
    public interface IRadarClickListener {
        void onRadarItemClick(int position);
    }

    /**
     * 根据position，放大指定的CircleView小圆点
     *
     * @param position
     */
    public void setCurrentShowItem(int position) {
        CircleView child = (CircleView) getChildAt(position + 1);
        resetAnim(currentShowChild);
        currentShowChild = child;
        startAnim(currentShowChild, position);
    }
}
```


每次点击雷达图中的小圆点都会告诉ViewPager切换到指定的页面，所以RadarViewGroup中需要定义一个IRadarClickListener，让ViewPager所在的MainAcitivity去实现该接口 
完成的效果就是这样了

![](http://img.blog.csdn.net/20160505001804763)

### 参考链接

[Android超高仿QQ附近的人搜索展示 - Mr_immortalZ的博客 - 博客频道 - CSDN.NET](http://blog.csdn.net/mr_immortalz/article/details/51319354)


### 源码下载 

[ImmortalZ/RadarScan: 这是Android一个雷达扫描显示的扫描图，超高仿QQ附近的人搜索展示](https://github.com/ImmortalZ/RadarScan)

### 完成