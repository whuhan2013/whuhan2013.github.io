---
layout: post
title: 实现翻转卡片的动画效果
date: 2016-6-4
categories: blog
tags: [自定义控件]
description: 实现翻转卡片的动画效果
---

### 转载   


在Android设计中, 经常会使用卡片元素, 正面显示图片或主要信息, 背面显示详细内容, 如网易有道词典的单词翻转和海底捞的食谱展示. 实现卡片视图非常容易, 那么如何实现翻转动画呢?

![](http://upload-images.jianshu.io/upload_images/749674-9b0255288c82a519.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 首页    

首页由正面和背面两张卡片组成, 同时, 设置点击事件flipCard.

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    android:id="@+id/main_fl_container"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:onClick="flipCard"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="me.chunyu.spike.wcl_flip_anim_demo.MainActivity">

    <include
        layout="@layout/cell_card_back"/>

    <include
        layout="@layout/cell_card_front"/>

</FrameLayout>
```



**逻辑, 初始化动画和镜头距离.**      


```
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        setAnimators(); // 设置动画
        setCameraDistance(); // 设置镜头距离
    }
```


### 动画 

初始化右出(RightOut)和左入(LeftIn)动画, 使用动画集合AnimatorSet.
当右出动画开始时, 点击事件无效, 当左入动画结束时, 点击事件恢复.

```
    // 设置动画
    private void setAnimators() {
        mRightOutSet = (AnimatorSet) AnimatorInflater.loadAnimator(this, R.animator.anim_out);
        mLeftInSet = (AnimatorSet) AnimatorInflater.loadAnimator(this, R.animator.anim_in);

        // 设置点击事件
        mRightOutSet.addListener(new AnimatorListenerAdapter() {
            @Override public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
                mFlContainer.setClickable(false);
            }
        });
        mLeftInSet.addListener(new AnimatorListenerAdapter() {
            @Override public void onAnimationEnd(Animator animation) {
                super.onAnimationEnd(animation);
                mFlContainer.setClickable(true);
            }
        });
    }
```

**右出动画**

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <!--旋转-->
    <objectAnimator
        android:duration="@integer/anim_length"
        android:propertyName="rotationY"
        android:valueFrom="0"
        android:valueTo="180"/>

    <!--消失-->
    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:startOffset="@integer/anim_half_length"
        android:valueFrom="1.0"
        android:valueTo="0.0"/>
</set>
```

旋转180°, 当旋转一半时, 卡片消失.

**左入动画**

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <!--消失-->
    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:valueFrom="1.0"
        android:valueTo="0.0"/>

    <!--旋转-->
    <objectAnimator
        android:duration="@integer/anim_length"
        android:propertyName="rotationY"
        android:valueFrom="-180"
        android:valueTo="0"/>

    <!--出现-->
    <objectAnimator
        android:duration="0"
        android:propertyName="alpha"
        android:startOffset="@integer/anim_half_length"
        android:valueFrom="0.0"
        android:valueTo="1.0"/>
</set>
```

在开始时是隐藏, 逆向旋转, 当旋转一半时, 显示卡片.


###  镜头视角        

改变视角, 涉及到旋转卡片的Y轴, 即rotationY, 需要修改视角距离.
如果不修改, 则会超出屏幕高度, 影响视觉体验.


```
    // 改变视角距离, 贴近屏幕
    private void setCameraDistance() {
        int distance = 16000;
        float scale = getResources().getDisplayMetrics().density * distance;
        mFlCardFront.setCameraDistance(scale);
        mFlCardBack.setCameraDistance(scale);
    }
```

###  旋转控制

设置右出和左入动画的目标控件, 两个动画同步进行, 并区分正反面朝上.

```
    // 翻转卡片
    public void flipCard(View view) {
        // 正面朝上
        if (!mIsShowBack) {
            mRightOutSet.setTarget(mFlCardFront);
            mLeftInSet.setTarget(mFlCardBack);
            mRightOutSet.start();
            mLeftInSet.start();
            mIsShowBack = true;
        } else { // 背面朝上
            mRightOutSet.setTarget(mFlCardBack);
            mLeftInSet.setTarget(mFlCardFront);
            mRightOutSet.start();
            mLeftInSet.start();
            mIsShowBack = false;
        }
    }
```

### 动画效果

![](http://upload-images.jianshu.io/upload_images/749674-89be3fa5b7608647.gif?imageMogr2/auto-orient/strip)

### 原文链接 

[实现翻转卡片的动画效果 - 简书](http://www.jianshu.com/p/7db8425e84fc)
