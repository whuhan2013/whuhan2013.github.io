---
layout: post
title: Android下拉刷新效果实现
date: 2016-6-8
categories: blog
tags: [android]
description: Android下拉刷新效果实现
---

**本文主要包括以下内容**   

1. 自定义实现pulltorefreshView
2. 使用google官方SwipeRefreshLayout

**下拉刷新大致原理** 

判断当前是否在最上面而且是向下滑的，如果是的话，则加载数据，并更新界面。


### 自定义实现pulltorefreshView

```
package com.jimstin.pulltorefreshviewdemo.view;

import com.jimstin.pulltorefreshviewdemo.R;

import android.content.Context;
import android.content.SharedPreferences;
import android.graphics.Canvas;
import android.os.AsyncTask;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewTreeObserver;
import android.view.ViewTreeObserver.OnPreDrawListener;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.RelativeLayout;
import android.widget.ScrollView;
import android.widget.TextView;
import android.view.View.OnTouchListener;
import android.view.animation.Animation;
import android.view.animation.Animation.AnimationListener;
import android.view.animation.LinearInterpolator;
import android.view.animation.RotateAnimation;

public class PullToRefreshView  extends LinearLayout implements OnTouchListener {

    private static PullToRefreshView mPullToRefreshView;
    private static Context mContext;
    private AttributeSet mAttrs;
    private View[] childs;
    private RelativeLayout mTopView, mBottomView;
    private MyScrollView mScrollView;
    private LinearLayout mContentView;
    private LayoutParams mParams;
    private static TextView mTopTips, mBottomTips;
    private static ImageView mTopTipsIcon, mBottomTipsIcon;
    private RotateAnimation  toAnmt, backAnmt, progressBarAnmt;
    
    private boolean isToTop, isToBottom;
    private boolean isGetChilds;
    private boolean isRecordY;
    private boolean isContentNotFull;
    private boolean isResetLayoutParams;
    private boolean isActviteScrollEvent;//偏移量达到刷新或加载更多的要求时，为true，否则为false
    private boolean isTurnUp;//判断手势，向上时为true，向下为false
    private static boolean isRefresing;//是否正在刷新
    private static boolean isRefreshed = true;//是否已刷新完毕
    private static boolean isLoading;//是否正在加载
    private static boolean isLoaded = true;//是否已加载完毕
    private static boolean isPulling;//是否正在拖动scrollview
    private boolean isRotate;//是否已经动画
    
    private static int topViewHeight;//topview的高度，和bottomview高度一致
    private int mHeight;//整个PullToRefreshView的高度
    private int scrollToEnd;//scrollview滚动到底部的长度
    private float lastY;//
    private float guestureLastY;//用于记录手势按下屏幕时的第一个位置
    
    private OnRefreshListener mRefreshListener;
    private OnLoadListener mLoadListener;
    
    public PullToRefreshView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mContext = context;
        this.mAttrs = attrs;
        initView();
    }

    private void initView() {
        mPullToRefreshView = this;
        toAnmt = new RotateAnimation(0, 180f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
        toAnmt.setDuration(300);
        toAnmt.setFillAfter(true);
        backAnmt = new RotateAnimation(0, -180f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
        backAnmt.setDuration(300);
        backAnmt.setFillAfter(true);
        progressBarAnmt = new RotateAnimation(0, 360f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
        progressBarAnmt.setDuration(1800);
        progressBarAnmt.setRepeatCount(-1);
        progressBarAnmt.setRepeatMode(RotateAnimation.RESTART);
        progressBarAnmt.setInterpolator(new LinearInterpolator());
        toAnmt.setAnimationListener(new AnimationListener() {
            
            @Override
            public void onAnimationStart(Animation animation) {
                // TODO Auto-generated method stub
                
            }
            
            @Override
            public void onAnimationRepeat(Animation animation) {
                // TODO Auto-generated method stub
                
            }
            
            @Override
            public void onAnimationEnd(Animation animation) {
                // TODO Auto-generated method stub
                mTopTipsIcon.clearAnimation();
                mTopTipsIcon.setBackgroundResource(R.drawable.ic_up);
                mBottomTipsIcon.clearAnimation();
                mBottomTipsIcon.setBackgroundResource(R.drawable.ic_up);
            }
            
        });
        
        backAnmt.setAnimationListener(new AnimationListener() {
            
            @Override
            public void onAnimationStart(Animation animation) {
                // TODO Auto-generated method stub
                
            }
            
            @Override
            public void onAnimationRepeat(Animation animation) {
                // TODO Auto-generated method stub
                
            }
            
            @Override
            public void onAnimationEnd(Animation animation) {
                // TODO Auto-generated method stub
                mTopTipsIcon.clearAnimation();
                mTopTipsIcon.setBackgroundResource(R.drawable.ic_down);
                mBottomTipsIcon.clearAnimation();
                mBottomTipsIcon.setBackgroundResource(R.drawable.ic_down);
            }
        });
        
        ViewTreeObserver ob = getViewTreeObserver();
        ob.addOnPreDrawListener(new OnPreDrawListener() {
            
            @Override
            public boolean onPreDraw() {
                if(topViewHeight == 0 || mHeight == 0) {
                    if(topViewHeight == 0) {
                        topViewHeight = mTopView.getHeight();
                    }
                    if(mHeight == 0) {
                        mHeight = getHeight();
                    }
                    if(mHeight > 0 && topViewHeight > 0) {
                        SharedPreferences p = mContext.getSharedPreferences("PullToRefreshView", 0);
                        p.edit().putInt("barHeight", topViewHeight).putInt("height", mHeight).commit();
                    }
                    mParams = new LayoutParams(LayoutParams.MATCH_PARENT, mHeight+2*topViewHeight);
                    setLayoutParams(mParams);
                    setY(-topViewHeight);
                }
                return true;
            }
        });
    }
    
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // TODO Auto-generated method stub
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }
    
    @Override
    protected void onFinishInflate() {
        // TODO Auto-generated method stub
        super.onFinishInflate();
        setLayout();
    }
    
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
    }

    public void setLayout() {
        if(!isGetChilds) {
            
            int count = getChildCount();
            childs = new View[count];
            for(int i=0; i<count; i++) {
                childs[i] = getChildAt(i);
            }
            for(int i=0; i<count; i++) {
                removeAllViews();
            }
            LayoutInflater inflater = LayoutInflater.from(mContext);
            mTopView = (RelativeLayout) inflater.inflate(R.layout.layout_top_pull_to_refresh_view, null);
            mBottomView = (RelativeLayout) inflater.inflate(R.layout.layout_bottom_pull_to_refresh_view, null);
            mTopTips = (TextView) mTopView.findViewById(R.id.top_tips);
            mBottomTips = (TextView) mBottomView.findViewById(R.id.bottom_tips);
            mTopTipsIcon = (ImageView) mTopView.findViewById(R.id.ic_down);
            mBottomTipsIcon = (ImageView) mBottomView.findViewById(R.id.ic_up);
            
            mScrollView = new MyScrollView(mContext, mAttrs);
            mParams = new LayoutParams(LayoutParams.MATCH_PARENT, 0);
            mParams.weight = 1;
            mScrollView.setLayoutParams(mParams);
            mContentView = new LinearLayout(mContext, mAttrs);
            mContentView.setLayoutParams(mParams);
            mContentView.setOrientation(LinearLayout.VERTICAL);
            for(int i=0; i<count; i++) {
                mContentView.addView(childs[i]);
            }
            mScrollView.addView(mContentView);
            mScrollView.setOnTouchListener(this);
            addView(mTopView);
            addView(mScrollView);
            addView(mBottomView);
            isGetChilds = true;
        }
    }
    
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if(mScrollView.getHeight() > mContentView.getHeight()) {
            isContentNotFull = true;
            mBottomView.setVisibility(View.GONE);
        } else {
            isContentNotFull = false;
        }
    }
    /**
     * 设置刷新监听器
     */
    public void setOnRefreshListener(OnRefreshListener listener) {
        mRefreshListener = listener;
    }
    /**
     * 设置加载监听器
     */
    public void setOnLoadListener(OnLoadListener listener) {
        mLoadListener = listener;
    }
    
    class MyScrollView extends ScrollView {

        public MyScrollView(Context context, AttributeSet attrs) {
            super(context, attrs);
        }

        @Override
        protected void onScrollChanged(int l, int t, int oldl, int oldt) {
            super.onScrollChanged(l, t, oldl, oldt);
            //MyLog.i("t="+t);
            if(t == 0) {
                isToTop = true;
            } else {
                if(t == getChildAt(0).getHeight()-getHeight()) {
                    scrollToEnd = t;
                    isToBottom = true;
                }
            }
        }
    }
    
    /**
     * 刷新监听器
     */
    public static abstract class OnRefreshListener {
        /**
         * 正在刷新
         */
        public void onRefresh() {
        }
        /**
         * 停止刷新，刷新完毕时请主动调用
         */
        public void stopRefresh() {
            isRefresing = false;
            mTopTips.setText(mContext.getResources().getString(R.string.refreshed));
            mTopTipsIcon.clearAnimation();
            mTopTipsIcon.setBackgroundResource(R.drawable.ic_finish);
            resetYOffset();
        }
    }
    /**
     * 加载更多监听器
     *
     */
    public static abstract class OnLoadListener {
        /**
         * 正在加载
         */
        public void onLoad() {
        }
        /**
         * 停止加载，加载完毕时请主动调用
         */
        public void stopLoad() {
            isLoading = false;
            mBottomTips.setText(mContext.getResources().getString(R.string.loaded));
            mBottomTipsIcon.clearAnimation();
            mBottomTipsIcon.setBackgroundResource(R.drawable.ic_finish);
            resetYOffset();
        }
    }
    
    public static void resetYOffset() {
        new AsyncTask<Integer, Integer, Integer>() {

            @Override
            protected Integer doInBackground(Integer... params) {
                try {
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return null;
            }
            
            protected void onPostExecute(Integer result) {
                if(!isPulling) {
                    mPullToRefreshView.setY(-topViewHeight);
                    mTopTipsIcon.setBackgroundResource(R.drawable.ic_down);
                    mBottomTipsIcon.setBackgroundResource(R.drawable.ic_up);
                    isRefreshed = true;
                    isLoaded = true;
                }
                
            };
        }.execute();
    }

    @Override
    public boolean onTouch(View v, MotionEvent event) {
        switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            guestureLastY = event.getRawY();
            isPulling = true;
            break;
        case MotionEvent.ACTION_MOVE:
            
            if(event.getRawY() - guestureLastY > 0) {
                isTurnUp = false;
            } else {
                isTurnUp = true;
            }
            if(isContentNotFull && !isResetLayoutParams) {
                mParams = new LayoutParams(LayoutParams.MATCH_PARENT, getHeight()-topViewHeight);
                setLayoutParams(mParams);
                isResetLayoutParams = true;
            }
            if(isToTop && !isTurnUp) {
                
                if(!isRecordY) {
                    lastY = event.getRawY()/3;
                    isRecordY = true;
                }
                
                mScrollView.setVerticalScrollBarEnabled(false);
                float y = 0.0f;
                if(!isRefreshed) {
                    if(!isPulling) {
                        y = event.getRawY()/3 - lastY - topViewHeight;
                    } else {
                        y = event.getRawY()/3 - lastY;
                    }
                } else {
                    y = event.getRawY()/3 - lastY - topViewHeight;
                }
                if(!isRefresing && isRefreshed) {
                    if(Math.abs(event.getRawY()/3 - lastY) >= topViewHeight) {
                        isActviteScrollEvent = true;
                        mTopTips.setText(mContext.getResources().getString(R.string.release_to_refresh));
                        if(!isRotate) {
                            mTopTipsIcon.startAnimation(toAnmt);
                            isRotate = true;
                        }
                        
                    } else {
                        
                        isActviteScrollEvent = false;
                        mTopTips.setText(mContext.getResources().getString(R.string.pull_to_refresh));
                        if(isRotate) {
                            mTopTipsIcon.startAnimation(backAnmt);
                            isRotate = false;
                        }
                    }
                }
                setY(y);
                if(event.getRawY()/3 - lastY < 0) {
                    isToTop = false;
                    isRecordY = false;
                } else {
                    mScrollView.setScrollY(0);
                }
            } else if(isToBottom || isContentNotFull) {
                
                if(!isRecordY) {
                    lastY = event.getRawY()/3;
                    isRecordY = true;
                }
                
                mScrollView.setVerticalScrollBarEnabled(false);
                
                if(!isLoading && isLoaded) {
                    if(Math.abs(event.getRawY()/3 - lastY) >= topViewHeight) {
                        isActviteScrollEvent = true;
                        mBottomTips.setText(mContext.getResources().getString(R.string.release_to_load));
                        if(!isRotate) {
                            mBottomTipsIcon.startAnimation(backAnmt);
                            isRotate = true;
                        }
                    } else {
                        isActviteScrollEvent = false;
                        mBottomTips.setText(mContext.getResources().getString(R.string.push_to_load));
                        if(isRotate) {
                            mBottomTipsIcon.startAnimation(toAnmt);
                            isRotate = false;
                        }
                    }
                }
                float y = 0.0f;
                if(!isLoaded) {
                    if(isPulling) {
                        y = event.getRawY()/3 - lastY - 2*topViewHeight;
                    } else {
                        y = event.getRawY()/3 - lastY - topViewHeight;
                    }
                } else {
                    y = event.getRawY()/3 - lastY - topViewHeight;
                }
                setY(y);
                if(event.getRawY()/3 - lastY > 0) {
                    isToBottom = false;
                    isRecordY = false;
                } else {
                    mScrollView.setScrollY(scrollToEnd);
                }
            }
            break;
        case MotionEvent.ACTION_UP:
            
            mScrollView.setVerticalScrollBarEnabled(true);
            isRotate = false;
            isRecordY = false;
            isPulling = false;
            if(isActviteScrollEvent) {
                if(isToTop && !isTurnUp) {
                    if(mRefreshListener != null) {
                        isRefresing = true;
                        isRefreshed = false;
                        setY(0);
                        mTopTips.setText(mContext.getResources().getString(R.string.refreshing));
                        mTopTipsIcon.setBackgroundResource(R.drawable.ic_refresh_progress_bar);
                        mTopTipsIcon.startAnimation(progressBarAnmt);
                        mRefreshListener.onRefresh();
                    }
                } else if(isToBottom) {
                    if(mLoadListener != null) {
                        setY(-2*topViewHeight);
                        mBottomTips.setText(mContext.getResources().getString(R.string.loading));
                        mBottomTipsIcon.setBackgroundResource(R.drawable.ic_refresh_progress_bar);
                        mBottomTipsIcon.startAnimation(progressBarAnmt);
                        isLoading = true;
                        isLoaded = false;
                        mLoadListener.onLoad();
                    }
                } else if(isContentNotFull) {
                    setY(-topViewHeight);
                }
                
                isActviteScrollEvent = false;
            } else {
                if(isRefresing) {
                    setY(0);
                } else if(isLoading) {
                    setY(-2*topViewHeight);
                } else {
                    setY(-topViewHeight);
                    
                }
            }
            if(!isRefresing) {
                isRefreshed = true;
            } 
            if(!isLoading) {
                isLoaded = true;
            }
            
            isToTop = false;
            isToBottom = false;
            break;

        default:
            break;
        
            }
            return false;
    }
    
    public void initLayoutParams() {
        SharedPreferences p = mContext.getSharedPreferences("PullToRefreshView", 0);
        topViewHeight = p.getInt("barHeight", 0);
        mHeight = p.getInt("height", 0);
        
        if(mHeight > 0) {
            mParams = new LayoutParams(LayoutParams.MATCH_PARENT, mHeight+2*topViewHeight);
            setLayoutParams(mParams);
            setY(-topViewHeight);
        }
    }
}
```



### 参考链接

[PullToRefreshView下拉刷新上来加载更多，支持任何子view！ - zhangjm_123的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/zhangjm_123/article/details/42025597)


### SwipeRefreshLayout 使用


**布局文件**

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp"
    android:background="#Ffffff"
    >
    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/swipe_refresh_layout_caselist"
        android:layout_marginTop="4dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">

    <ListView
        android:id="@+id/list"
        android:scrollbars="none"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:dividerHeight="0dp"
        android:divider="#Ffffff">

    </ListView>
    </android.support.v4.widget.SwipeRefreshLayout>
</LinearLayout>
```

**在Activity中使用**

```
 @Override
    protected void setUpView() {
        if (mIsFirstCreated)
        {
            onRefresh();
        }

        listView = $(R.id.list);
        swipeRefreshLayout=$(R.id.swipe_refresh_layout_caselist);
        //设置刷新时动画的颜色，可以设置4个
        swipeRefreshLayout.setColorSchemeResources(android.R.color.holo_blue_light, android.R.color.holo_red_light, android.R.color.holo_orange_light, android.R.color.holo_green_light);


        swipeRefreshLayout.setOnRefreshListener(this);
        getData();
//        BaseAdapter arrayAdapter = new ArrayAdapter(getActivity(),R.layout.item_list,R.id.name,
//                Arrays.asList("美食劵", "活动劵", "优惠劵", "团购劵", "外卖劵"));
//        listView.setAdapter(arrayAdapter);

    }





    @Override
    public void onRefresh() {
        //tv.setText("正在刷新");
        // TODO Auto-generated method stub
        mIsFirstCreated = false;
        getData();

    }
```


使用的类要实现SwipeRefreshLayout.OnRefreshListener接口

并且在数据获取成功后发送消息，设置swipeRefreshLayout.setRefreshing(false);

### 参考链接

[Google自己的下拉刷新组件SwipeRefreshLayout - 李金尧 - 博客园](http://www.cnblogs.com/JinyaoLi/p/3852424.html)

[你还在用开源控件的下拉刷新吗？你out了，试一试官方的下拉刷新SwipeRefreshLayout - 推酷](http://www.tuicool.com/articles/UrQzUv3)

[Android View 下拉刷新之头部效果自定义 [水] - bfbx5173的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/bfbx5173/article/details/49814551)