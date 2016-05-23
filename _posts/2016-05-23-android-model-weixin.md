---
layout: post
title: Android仿微信界面
date: 2016-5-23
categories: blog
tags: [自定义控件]
description: Android仿微信界面
---   

### 效果图

![](http://img.blog.csdn.net/20141113215629968)


### 原理介绍

![](http://img.blog.csdn.net/20140426213750328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

1、先绘制一个颜色（例如：粉红）   
2、设置Mode=DST_IN                
3、绘制我们这个可爱的小机器人                 
回答我，显示什么，是不是显示交集，交集是什么？交集是我们的小机器人的非透明区域，也就是那张脸，除了两个眼；              
好了，那怎么变色呢？             
我绘制一个颜色的时候，难道不能设置alpha么



### 自定义图标控件


### 自定义属性


```
<?xml version="1.0" encoding="utf-8"?>  
<resources>  
  
    <attr name="icon" format="reference" />  
    <attr name="color" format="color" />  
    <attr name="text" format="string" />  
    <attr name="text_size" format="dimension" />  
  
    <declare-styleable name="ChangeColorIconView">  
        <attr name="icon" />  
        <attr name="color" />  
        <attr name="text" />  
        <attr name="text_size" />  
    </declare-styleable>  
  
</resources>  
```


### 绘制图标 

绘制图标有很多步骤呀，我来列一列        
1、计算alpha（默认为0）            
2、绘制原图               
3、在绘图区域，绘制一个纯色块（设置了alpha），此步绘制在内存的bitmap上                                                           
4、设置mode，针对内存中的bitmap上的paint                  
5、绘制我们的图标，此步绘制在内存的bitmap上                  
6、绘制原文本                                        
7、绘制设置alpha和颜色后的文本                
8、将内存中的bitmap绘制出来                    
根据上面的步骤，可以看出来，我们的图标其实绘制了两次，为什么要绘制原图呢，因为我觉得比较好看。             
3-5步骤，就是我们上面分析的原理                  
6-7步，是绘制文本，可以看到，我们的文本就是通过设置alpha实现的     


```
package com.zhy.weixin6.ui;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Bitmap;
import android.graphics.Bitmap.Config;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.PorterDuff;
import android.graphics.PorterDuffXfermode;
import android.graphics.Rect;
import android.graphics.drawable.BitmapDrawable;
import android.os.Bundle;
import android.os.Looper;
import android.os.Parcelable;
import android.util.AttributeSet;
import android.util.Log;
import android.util.TypedValue;
import android.view.View;

public class ChangeColorIconWithTextView extends View
{

    private Bitmap mBitmap;
    private Canvas mCanvas;
    private Paint mPaint;
    /**
     * 颜色
     */
    private int mColor = 0xFF45C01A;
    /**
     * 透明度 0.0-1.0
     */
    private float mAlpha = 0f;
    /**
     * 图标
     */
    private Bitmap mIconBitmap;
    /**
     * 限制绘制icon的范围
     */
    private Rect mIconRect;
    /**
     * icon底部文本
     */
    private String mText = "微信";
    private int mTextSize = (int) TypedValue.applyDimension(
            TypedValue.COMPLEX_UNIT_SP, 10, getResources().getDisplayMetrics());
    private Paint mTextPaint;
    private Rect mTextBound = new Rect();

    public ChangeColorIconWithTextView(Context context)
    {
        super(context);
    }

    /**
     * 初始化自定义属性值
     * 
     * @param context
     * @param attrs
     */
    public ChangeColorIconWithTextView(Context context, AttributeSet attrs)
    {
        super(context, attrs);

        // 获取设置的图标
        TypedArray a = context.obtainStyledAttributes(attrs,
                R.styleable.ChangeColorIconView);

        int n = a.getIndexCount();
        for (int i = 0; i < n; i++)
        {

            int attr = a.getIndex(i);
            switch (attr)
            {
            case R.styleable.ChangeColorIconView_icon:
                BitmapDrawable drawable = (BitmapDrawable) a.getDrawable(attr);
                mIconBitmap = drawable.getBitmap();
                break;
            case R.styleable.ChangeColorIconView_color:
                mColor = a.getColor(attr, 0x45C01A);
                break;
            case R.styleable.ChangeColorIconView_text:
                mText = a.getString(attr);
                break;
            case R.styleable.ChangeColorIconView_text_size:
                mTextSize = (int) a.getDimension(attr, TypedValue
                        .applyDimension(TypedValue.COMPLEX_UNIT_SP, 10,
                                getResources().getDisplayMetrics()));
                break;

            }
        }

        a.recycle();

        mTextPaint = new Paint();
        mTextPaint.setTextSize(mTextSize);
        mTextPaint.setColor(0xff555555);
        // 得到text绘制范围
        mTextPaint.getTextBounds(mText, 0, mText.length(), mTextBound);

    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
    {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        // 得到绘制icon的宽
        int bitmapWidth = Math.min(getMeasuredWidth() - getPaddingLeft()
                - getPaddingRight(), getMeasuredHeight() - getPaddingTop()
                - getPaddingBottom() - mTextBound.height());

        int left = getMeasuredWidth() / 2 - bitmapWidth / 2;
        int top = (getMeasuredHeight() - mTextBound.height()) / 2 - bitmapWidth
                / 2;
        // 设置icon的绘制范围
        mIconRect = new Rect(left, top, left + bitmapWidth, top + bitmapWidth);

    }

    @Override
    protected void onDraw(Canvas canvas)
    {

        int alpha = (int) Math.ceil((255 * mAlpha));
        canvas.drawBitmap(mIconBitmap, null, mIconRect, null);
        setupTargetBitmap(alpha);
        drawSourceText(canvas, alpha);
        drawTargetText(canvas, alpha);
        canvas.drawBitmap(mBitmap, 0, 0, null);

    }
    
    private void setupTargetBitmap(int alpha)
    {
        mBitmap = Bitmap.createBitmap(getMeasuredWidth(), getMeasuredHeight(),
                Config.ARGB_8888);
        mCanvas = new Canvas(mBitmap);
        mPaint = new Paint();
        mPaint.setColor(mColor);
        mPaint.setAntiAlias(true);
        mPaint.setDither(true);
        mPaint.setAlpha(alpha);
        mCanvas.drawRect(mIconRect, mPaint);
        mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
        mPaint.setAlpha(255);
        mCanvas.drawBitmap(mIconBitmap, null, mIconRect, mPaint);
    }

    private void drawSourceText(Canvas canvas, int alpha)
    {
        mTextPaint.setTextSize(mTextSize);
        mTextPaint.setColor(0xff333333);
        mTextPaint.setAlpha(255 - alpha);
        canvas.drawText(mText, mIconRect.left + mIconRect.width() / 2
                - mTextBound.width() / 2,
                mIconRect.bottom + mTextBound.height(), mTextPaint);
    }
    
    private void drawTargetText(Canvas canvas, int alpha)
    {
        mTextPaint.setColor(mColor);
        mTextPaint.setAlpha(alpha);
        canvas.drawText(mText, mIconRect.left + mIconRect.width() / 2
                - mTextBound.width() / 2,
                mIconRect.bottom + mTextBound.height(), mTextPaint);
        
    }

    

    

    public void setIconAlpha(float alpha)
    {
        this.mAlpha = alpha;
        invalidateView();
    }

    private void invalidateView()
    {
        if (Looper.getMainLooper() == Looper.myLooper())
        {
            invalidate();
        } else
        {
            postInvalidate();
        }
    }

    public void setIconColor(int color)
    {
        mColor = color;
    }

    public void setIcon(int resId)
    {
        this.mIconBitmap = BitmapFactory.decodeResource(getResources(), resId);
        if (mIconRect != null)
            invalidateView();
    }

    public void setIcon(Bitmap iconBitmap)
    {
        this.mIconBitmap = iconBitmap;
        if (mIconRect != null)
            invalidateView();
    }

    private static final String INSTANCE_STATE = "instance_state";
    private static final String STATE_ALPHA = "state_alpha";

    @Override
    protected Parcelable onSaveInstanceState()
    {
        Bundle bundle = new Bundle();
        bundle.putParcelable(INSTANCE_STATE, super.onSaveInstanceState());
        bundle.putFloat(STATE_ALPHA, mAlpha);
        return bundle;
    }

    @Override
    protected void onRestoreInstanceState(Parcelable state)
    {
        if (state instanceof Bundle)
        {
            Bundle bundle = (Bundle) state;
            mAlpha = bundle.getFloat(STATE_ALPHA);
            super.onRestoreInstanceState(bundle.getParcelable(INSTANCE_STATE));
        } else
        {
            super.onRestoreInstanceState(state);
        }

    }

}
```


### MainActivity


```
package com.zhy.weixin6.ui;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;

import android.annotation.SuppressLint;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.view.ViewPager;
import android.support.v4.view.ViewPager.OnPageChangeListener;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.ViewConfiguration;
import android.view.Window;
import android.widget.SearchView;
import android.widget.Toast;

@SuppressLint("NewApi")
public class MainActivity extends FragmentActivity implements
        OnPageChangeListener, OnClickListener
{
    private ViewPager mViewPager;
    private List<Fragment> mTabs = new ArrayList<Fragment>();
    private FragmentPagerAdapter mAdapter;

    private String[] mTitles = new String[] { "First Fragment!",
            "Second Fragment!", "Third Fragment!", "Fourth Fragment!" };

    private List<ChangeColorIconWithTextView> mTabIndicator = new ArrayList<ChangeColorIconWithTextView>();

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        setOverflowShowingAlways();
        getActionBar().setDisplayShowHomeEnabled(false);
        mViewPager = (ViewPager) findViewById(R.id.id_viewpager);

        initDatas();

        mViewPager.setAdapter(mAdapter);
        mViewPager.setOnPageChangeListener(this);
    }

    private void initDatas()
    {

        for (String title : mTitles)
        {
            TabFragment tabFragment = new TabFragment();
            Bundle args = new Bundle();
            args.putString("title", title);
            tabFragment.setArguments(args);
            mTabs.add(tabFragment);
        }

        mAdapter = new FragmentPagerAdapter(getSupportFragmentManager())
        {

            @Override
            public int getCount()
            {
                return mTabs.size();
            }

            @Override
            public Fragment getItem(int arg0)
            {
                return mTabs.get(arg0);
            }
        };

        initTabIndicator();

    }
     SearchView searchView;
    @Override
    public boolean onCreateOptionsMenu(Menu menu)
    {
        getMenuInflater().inflate(R.menu.main, menu);
        
        MenuItem searchItem = menu.findItem(R.id.action_search);  
         searchView = (SearchView) searchItem.getActionView();  
        
        return true;
    }
    
    @Override
    public boolean onSearchRequested() {
        // TODO Auto-generated method stub
        Toast.makeText(getApplicationContext(), searchView.getQuery(), 0).show();
        return super.onSearchRequested();
    }

    private void initTabIndicator()
    {
        ChangeColorIconWithTextView one = (ChangeColorIconWithTextView) findViewById(R.id.id_indicator_one);
        ChangeColorIconWithTextView two = (ChangeColorIconWithTextView) findViewById(R.id.id_indicator_two);
        ChangeColorIconWithTextView three = (ChangeColorIconWithTextView) findViewById(R.id.id_indicator_three);
        ChangeColorIconWithTextView four = (ChangeColorIconWithTextView) findViewById(R.id.id_indicator_four);

        mTabIndicator.add(one);
        mTabIndicator.add(two);
        mTabIndicator.add(three);
        mTabIndicator.add(four);

        one.setOnClickListener(this);
        two.setOnClickListener(this);
        three.setOnClickListener(this);
        four.setOnClickListener(this);

        one.setIconAlpha(1.0f);
    }

    @Override
    public void onPageSelected(int arg0)
    {
    }
    
    
    @Override
    public void onPageScrolled(int position, float positionOffset,
            int positionOffsetPixels)
    {
        // Log.e("TAG", "position = " + position + " , positionOffset = "
        // + positionOffset);

        if (positionOffset > 0)
        {
            ChangeColorIconWithTextView left = mTabIndicator.get(position);
            ChangeColorIconWithTextView right = mTabIndicator.get(position + 1);

            left.setIconAlpha(1 - positionOffset);
            right.setIconAlpha(positionOffset);
        }

    }

    @Override
    public void onPageScrollStateChanged(int state)
    {

    }

    @Override
    public void onClick(View v)
    {

        resetOtherTabs();

        switch (v.getId())
        {
        case R.id.id_indicator_one:
            mTabIndicator.get(0).setIconAlpha(1.0f);
            mViewPager.setCurrentItem(0, false);
            break;
        case R.id.id_indicator_two:
            mTabIndicator.get(1).setIconAlpha(1.0f);
            mViewPager.setCurrentItem(1, false);
            break;
        case R.id.id_indicator_three:
            mTabIndicator.get(2).setIconAlpha(1.0f);
            mViewPager.setCurrentItem(2, false);
            break;
        case R.id.id_indicator_four:
            mTabIndicator.get(3).setIconAlpha(1.0f);
            mViewPager.setCurrentItem(3, false);
            break;

        }

    }

    /**
     * 重置其他的Tab
     */
    private void resetOtherTabs()
    {
        for (int i = 0; i < mTabIndicator.size(); i++)
        {
            mTabIndicator.get(i).setIconAlpha(0);
        }
    }

    @Override
    public boolean onMenuOpened(int featureId, Menu menu)
    {
        if (featureId == Window.FEATURE_ACTION_BAR && menu != null)
        {
            if (menu.getClass().getSimpleName().equals("MenuBuilder"))
            {
                try
                {
                    Method m = menu.getClass().getDeclaredMethod(
                            "setOptionalIconsVisible", Boolean.TYPE);
                    m.setAccessible(true);
                    m.invoke(menu, true);
                } catch (Exception e)
                {
                }
            }
        }
        return super.onMenuOpened(featureId, menu);
    }

    private void setOverflowShowingAlways()
    {
        try
        {
            // true if a permanent menu key is present, false otherwise.
            ViewConfiguration config = ViewConfiguration.get(this);
            Field menuKeyField = ViewConfiguration.class
                    .getDeclaredField("sHasPermanentMenuKey");
            menuKeyField.setAccessible(true);
            menuKeyField.setBoolean(config, false);
        } catch (Exception e)
        {
            e.printStackTrace();
        }
    }

}
```


Activity里面代码虽然没什么注释，但是很简单哈，就是初始化Fragment，得到我们的适配器，然后设置给ViewPager；
initTabIndicator我们初始化我们的自定义控件，以及加上了点击事件；
唯一一个需要指出的就是：               
我们在onPageScrolled中，动态的获取position以及positionOffset，然后拿到左右两个View，设置positionOffset ； 


两个反射的方法，是控制Actionbar的图标的，和点击menu按键显示的 


### TabFragment


```
package com.zhy.weixin6.ui;  
  
import android.graphics.Color;  
import android.os.Bundle;  
import android.support.v4.app.Fragment;  
import android.view.Gravity;  
import android.view.LayoutInflater;  
import android.view.View;  
import android.view.ViewGroup;  
import android.widget.TextView;  
  
public class TabFragment extends Fragment  
{  
    private String mTitle = "Default";  
      
  
    public TabFragment()  
    {  
    }  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState)  
    {  
        if (getArguments() != null)  
        {  
            mTitle = getArguments().getString("title");  
        }  
  
        TextView textView = new TextView(getActivity());  
        textView.setTextSize(20);  
        textView.setBackgroundColor(Color.parseColor("#ffffffff"));  
        textView.setGravity(Gravity.CENTER);  
        textView.setText(mTitle);  
        return textView;  
    }  
}  
```


### menu.xml文件  


```
<menu xmlns:android="http://schemas.android.com/apk/res/android" >

    <item
        android:id="@+id/action_search"
        android:actionViewClass="android.widget.SearchView"
        android:icon="@drawable/actionbar_search_icon"
        android:showAsAction="ifRoom|collapseActionView"
        android:title="@string/action_search"/>
    <item
        android:id="@+id/action_group_chat"
        android:icon="@drawable/ofm_group_chat_icon"
        android:title="@string/action_group_chat"/>
    <item
        android:id="@+id/action_add_friend"
        android:icon="@drawable/ofm_add_icon"
        android:title="@string/action_add_friend"/>
    <item
        android:id="@+id/action_scan"
        android:icon="@drawable/ofm_qrcode_icon"
        android:title="@string/action_scan"/>
    <item
        android:id="@+id/action_feed"
        android:icon="@drawable/ofm_feedback_icon"
        android:title="@string/action_feed"/>

</menu>
```


### 源码下载

[源码下载](http://download.csdn.net/detail/lmj623565791/8155321)


### 参考链接

[Android 高仿微信6.0主界面 带你玩转切换图标变色 - Hongyang - 博客频道 - CSDN.NET
http://blog.csdn.net/lmj623565791/article/details/41087219]()


