---
layout: post
title: ListView滑动删除效果实现
date: 2016-5-15
categories: blog
tags: [自定义控件]
description: ListView滑动删除效果实现
---   

通过继承ListView然后结合PopupWindow实现

首先是布局文件：
delete_btn.xml：这里只需要一个Button


```
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:orientation="vertical" >  
      <Button   
        android:id="@+id/id_item_btn"  
        android:layout_width="60dp"  
        android:singleLine="true"  
        android:layout_height="wrap_content"  
        android:text="删除"  
        android:background="@drawable/d_delete_btn"  
        android:textColor="#ffffff"  
        android:paddingLeft="15dp"  
        android:paddingRight="15dp"  
        android:layout_alignParentRight="true"  
        android:layout_centerVertical="true"  
        android:layout_marginRight="15dp"  
        />  
</LinearLayout>  
```


主布局文件：activity_main.xml，ListView的每个Item的样式直接使用了系统的android.R.layout.simple_list_item_1

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  
  
    <com.example.listviewitemslidedeletebtnshow.QQListView  
        android:id="@+id/id_listview"  
        android:layout_width="fill_parent"  
        android:layout_height="wrap_content" >  
    </com.example.listviewitemslidedeletebtnshow.QQListView>  
  
</RelativeLayout>  
```


接下来看看QQListView的实现：

```
package com.example.listviewitemslidedeletebtnshow;

import android.content.Context;
import android.util.AttributeSet;
import android.util.Log;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewConfiguration;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.ListView;
import android.widget.PopupWindow;

public class QQListView extends ListView
{

    private static final String TAG = "QQlistView";

    // private static final int VELOCITY_SANP = 200;
    // private VelocityTracker mVelocityTracker;
    /**
     * 用户滑动的最小距离
     */
    private int touchSlop;

    /**
     * 是否响应滑动
     */
    private boolean isSliding;

    /**
     * 手指按下时的x坐标
     */
    private int xDown;
    /**
     * 手指按下时的y坐标
     */
    private int yDown;
    /**
     * 手指移动时的x坐标
     */
    private int xMove;
    /**
     * 手指移动时的y坐标
     */
    private int yMove;

    private LayoutInflater mInflater;

    private PopupWindow mPopupWindow;
    private int mPopupWindowHeight;
    private int mPopupWindowWidth;

    private Button mDelBtn;
    /**
     * 为删除按钮提供一个回调接口
     */
    private DelButtonClickListener mListener;

    /**
     * 当前手指触摸的View
     */
    private View mCurrentView;

    /**
     * 当前手指触摸的位置
     */
    private int mCurrentViewPos;

    /**
     * 必要的一些初始化
     * 
     * @param context
     * @param attrs
     */
    public QQListView(Context context, AttributeSet attrs)
    {
        super(context, attrs);

        mInflater = LayoutInflater.from(context);
        touchSlop = ViewConfiguration.get(context).getScaledTouchSlop();

        View view = mInflater.inflate(R.layout.delete_btn, null);
        mDelBtn = (Button) view.findViewById(R.id.id_item_btn);
        mPopupWindow = new PopupWindow(view, LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT);
        /**
         * 先调用下measure,否则拿不到宽和高
         */
        mPopupWindow.getContentView().measure(0, 0);
        mPopupWindowHeight = mPopupWindow.getContentView().getMeasuredHeight();
        mPopupWindowWidth = mPopupWindow.getContentView().getMeasuredWidth();
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev)
    {
        int action = ev.getAction();
        int x = (int) ev.getX();
        int y = (int) ev.getY();
        switch (action)
        {

        case MotionEvent.ACTION_DOWN:
            xDown = x;
            yDown = y;
            /**
             * 如果当前popupWindow显示，则直接隐藏，然后屏蔽ListView的touch事件的下传
             */
            if (mPopupWindow.isShowing())
            {
                dismissPopWindow();
                return false;
            }
            // 获得当前手指按下时的item的位置
            mCurrentViewPos = pointToPosition(xDown, yDown);
            // 获得当前手指按下时的item
            View view = getChildAt(mCurrentViewPos - getFirstVisiblePosition());
            mCurrentView = view;
            break;
        case MotionEvent.ACTION_MOVE:
            xMove = x;
            yMove = y;
            int dx = xMove - xDown;
            int dy = yMove - yDown;
            /**
             * 判断是否是从右到左的滑动
             */
            if (xMove < xDown && Math.abs(dx) > touchSlop && Math.abs(dy) < touchSlop)
            {
                // Log.e(TAG, "touchslop = " + touchSlop + " , dx = " + dx +
                // " , dy = " + dy);
                isSliding = true;
            }
            break;
        }
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev)
    {
        int action = ev.getAction();
        /**
         * 如果是从右到左的滑动才相应
         */
        if (isSliding)
        {
            switch (action)
            {
            case MotionEvent.ACTION_MOVE:

                int[] location = new int[2];
                // 获得当前item的位置x与y
                mCurrentView.getLocationOnScreen(location);
                
                // 设置popupWindow的动画
                mPopupWindow.setAnimationStyle(R.style.popwindow_delete_btn_anim_style);
                mPopupWindow.update();
                mPopupWindow.showAtLocation(mCurrentView, Gravity.LEFT | Gravity.TOP,
                        location[0] + mCurrentView.getWidth(), location[1] + mCurrentView.getHeight() / 2
                                - mPopupWindowHeight / 2);
                Log.i("test", "location="+location[0]+","+location[1]);
                // 设置删除按钮的回调
                mDelBtn.setOnClickListener(new OnClickListener()
                {
                    @Override
                    public void onClick(View v)
                    {
                        if (mListener != null)
                        {
                            mListener.clickHappend(mCurrentViewPos);
                            mPopupWindow.dismiss();
                        }
                    }
                });
                // Log.e(TAG, "mPopupWindow.getHeight()=" + mPopupWindowHeight);

                break;
            case MotionEvent.ACTION_UP:
                isSliding = false;

            }
            // 相应滑动期间屏幕itemClick事件，避免发生冲突
            return true;
        }

        return super.onTouchEvent(ev);
    }

    /**
     * 隐藏popupWindow
     */
    private void dismissPopWindow()
    {
        if (mPopupWindow != null && mPopupWindow.isShowing())
        {
            mPopupWindow.dismiss();
        }
    }

    public void setDelButtonClickListener(DelButtonClickListener listener)
    {
        mListener = listener;
    }

    interface DelButtonClickListener
    {
        public void clickHappend(int position);
    }

}
```


代码注释写得很详细，简单说一下，在dispatchTouchEvent中设置当前是否响应用户滑动，然后在onTouchEvent中判断是否响应，如果响应则popupWindow以动画的形式展示出来。当然屏幕上如果存在PopupWindow则屏幕ListView的滚动与Item的点击，以及从右到左滑动时屏幕Item的click事件。

点击时记录下当前的位置，再判断是否右滑，如果右滑则出现popupWindow,根据按下时记录下的位置判断出popuwindow出现的位置。

MainActivity.java

```
package com.example.listviewitemslidedeletebtnshow;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.ArrayAdapter;
import android.widget.Toast;

import com.example.listviewitemslidedeletebtnshow.QQListView.DelButtonClickListener;

public class MainActivity extends Activity
{
    private QQListView mListView;
    private ArrayAdapter<String> mAdapter;
    private List<String> mDatas;

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mListView = (QQListView) findViewById(R.id.id_listview);
        // 不要直接Arrays.asList
        mDatas = new ArrayList<String>(Arrays.asList("HelloWorld", "Welcome", "Java", "Android", "Servlet", "Struts",
                "Hibernate", "Spring", "HTML5", "Javascript", "Lucene"));
        mAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, mDatas);
        mListView.setAdapter(mAdapter);

        mListView.setDelButtonClickListener(new DelButtonClickListener()
        {
            @Override
            public void clickHappend(final int position)
            {
                Toast.makeText(MainActivity.this, position + " : " + mAdapter.getItem(position), 1).show();
                mAdapter.remove(mAdapter.getItem(position));
            }
        });

        mListView.setOnItemClickListener(new OnItemClickListener()
        {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id)
            {
                Toast.makeText(MainActivity.this, position + " : " + mAdapter.getItem(position), 1).show();
            }
        });
    }
}
```


### 注意，这里使用了一种设计方法

在QQListView中定义了接口

```
public void setDelButtonClickListener(DelButtonClickListener listener)
    {
        mListener = listener;
    }

    interface DelButtonClickListener
    {
        public void clickHappend(int position);
    }
```

### 在MainActivity中实例化，然后listView中会调用

```
mListView.setDelButtonClickListener(new DelButtonClickListener()
        {
            @Override
            public void clickHappend(final int position)
            {
                Toast.makeText(MainActivity.this, position + " : " + mAdapter.getItem(position), 1).show();
                mAdapter.remove(mAdapter.getItem(position));
            }
        });
```

实例化的clickHappend方法。

### 参考资料

[ListView滑动删除 ，仿腾讯QQ - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/22961279)

[Android PopupWindow的使用和分析 - 圣骑士wind - 博客园
](http://www.cnblogs.com/mengdd/p/3569127.html)

[PopUpWindow使用详解(一)——基本使用 - 启舰 - 博客频道 - CSDN.NET](http://blog.csdn.net/harvic880925/article/details/49272285)

### 注意

```
mPopupWindow.showAtLocation(mCurrentView, Gravity.LEFT | Gravity.TOP,
                        location[0]+1850 , location[1] + mCurrentView.getHeight() / 2
                                - mPopupWindowHeight / 2);
```

似乎，不论向右偏移多大，popupwindow最多也是在最右边

### 完成


