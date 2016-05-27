---
layout: post
title: Material Design入门（三）
date: 2016-4-8
categories: blog
tags: [Material Design]
description: Material Design入门（三）
---

### 本文主要包括  

1. CollapsingToolbarLayout实现滚动动画效果  
2.  ViewPager+tabLayout实现左右类Tab效果   




### 控件介绍
这次需要用到得新控件比较多，主要有以下几个：

- CoordinatorLayout
组织它的子views之间协作的一个Layout，它可以给子View切换提供动画效果。
- AppBarLayout
可以让包含在其中的控件响应被标记了ScrollingViewBehavior的View的滚动事件
- CollapsingToolbarLayout
可以控制包含在CollapsingToolbarLayout其中的控件，在响应collapse时是移除屏幕和固定在最上面
- TabLayout
结合ViewPager，实现多个TAB的切换的功能
- NestedScrollView
与ScrollView基本相同，不过包含在NestedScrollView中的控件移动时才能时AppBarLayout缩放 


CollapsingToolbarLayout作用是提供了一个可以折叠的Toolbar，它继承至FrameLayout，给它设置layout_scrollFlags，它可以控制包含在CollapsingToolbarLayout中的控件(如：ImageView、Toolbar)在响应layout_behavior事件时作出相应的scrollFlags滚动事件(移除屏幕或固定在屏幕顶端)。


### 这些控件详细介绍参见  

[通过来模仿稀土掘金个人页面的布局来学习使用CoordinatorLayout](http://mp.weixin.qq.com/s?__biz=MjM5NDkxMTgyNw==&mid=2653057481&idx=1&sn=b7fe4335fb4618a87758d51420535074&scene=0#wechat_redirect)

使用CollapsingToolbarLayout：


#### layout布局   

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="256dp"
        android:fitsSystemWindows="true"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:id="@+id/ivImage"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:fitsSystemWindows="true"
                android:scaleType="centerCrop"
                android:src="@drawable/book1"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.7" />

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />


        </android.support.design.widget.CollapsingToolbarLayout>

    </android.support.design.widget.AppBarLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <android.support.design.widget.TabLayout
            android:id="@+id/sliding_tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabGravity="fill"
            app:tabMode="fixed" />

        <android.support.v4.view.ViewPager
            android:id="@+id/viewpager"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </LinearLayout>
</android.support.design.widget.CoordinatorLayout>
```    


### activity代码  

```
 Toolbar mToolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(mToolbar);
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        mToolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                onBackPressed();
            }
        });
        //使用CollapsingToolbarLayout必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上则不会显示
        CollapsingToolbarLayout mCollapsingToolbarLayout = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
        mCollapsingToolbarLayout.setTitle("MyCollapsingToolbar");
        //通过CollapsingToolbarLayout修改字体颜色
        mCollapsingToolbarLayout.setExpandedTitleColor(Color.WHITE);//设置还没收缩时状态下字体颜色
        mCollapsingToolbarLayout.setCollapsedTitleTextColor(Color.GREEN);//设置收缩后Toolbar上字体的颜色
```


### 代码解释  

我们在CollapsingToolbarLayout中设置了一个ImageView和一个Toolbar。并把这个CollapsingToolbarLayout放到AppBarLayout中作为一个整体。

1. 在CollapsingToolbarLayout中：  

我们设置了layout_scrollFlags:关于它的值我这里再说一下：

- scroll - 想滚动就必须设置这个。

- enterAlways - 实现quick return效果, 当向下移动时，立即显示View（比如Toolbar)。

- exitUntilCollapsed - 向上滚动时收缩View，但可以固定Toolbar一直在上面。

- enterAlwaysCollapsed - 当你的View已经设置minHeight属性又使用此标志时，你的View只能以最小高度进入，只有当滚动视图到达顶部时才扩大到完整高度。

其中还设置了一些属性，简要说明一下：

- contentScrim - 设置当完全CollapsingToolbarLayout折叠(收缩)后的背景颜色。

- expandedTitleMarginStart - 设置扩张时候(还没有收缩时)title向左填充的距离。

2. 在ImageView控件中：

我们设置了：

- layout_collapseMode (折叠模式) - 有两个值:

- pin -  设置为这个模式时，当CollapsingToolbarLayout完全收缩后，Toolbar还可以保留在屏幕上。

- parallax - 设置为这个模式时，在内容滚动时，CollapsingToolbarLayout中的View（比如ImageView)也可以同时滚动，实现视差滚动效果，通常和layout_collapseParallaxMultiplier(设置视差因子)搭配使用。

- layout_collapseParallaxMultiplier(视差因子) - 设置视差滚动因子，值为：0~1。 

3. 在Toolbar控件中：

我们设置了layout_collapseMode(折叠模式)：为pin



###综上分析：
当设置了layout_behavior的控件响应起了CollapsingToolbarLayout中的layout_scrollFlags事件时，ImageView会有视差效果的向上滚动移除屏幕，当开始折叠时CollapsingToolbarLayout的背景色(也就是Toolbar的背景色)就会变为我们设置好的背景色，Toolbar也一直会固定在最顶端。


【注】:使用CollapsingToolbarLayout时必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上不会显示。即：

mCollapsingToolbarLayout.setTitle(" ");

该变title的字体颜色：

扩张时候的title颜色：mCollapsingToolbarLayout.setExpandedTitleColor();

收缩后在Toolbar上显示时的title的颜色：  mCollapsingToolbarLayout.setCollapsedTitleTextColor();

这个颜色的过度变化其实CollapsingToolbarLayout已经帮我们做好，它会自动的过度.

### 参考链接：
[Android5.0+(CollapsingToolbarLayout) - OPEN 开发经验库](http://www.open-open.com/lib/view/open1438265746378.html)


### 问题  

我在做这里的时候遇到一个问题，那就是CollapsingToolbarLayout里的Title的问题，一般默认是显示的，即使你不写，它也有会一个默认值一直显示在那里，等折叠收缩完的时候，停留在标题工具栏上。怎么消除这个默认值呢？怎么知道收缩完成了，再把这个值设置出来呢？这里我对AppBarLayout设置了一个监听，它有一个监听方法：addOnOffsetChangedListener监听折叠收缩的位移。如下：

```
app_bar_layout.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
            @Override
            public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
                if (verticalOffset <= -head_layout.getHeight() / 2) {
                    mCollapsingToolbarLayout.setTitle("涩郎");
                } else {
                    mCollapsingToolbarLayout.setTitle(" ");
                }
            }
        });
```



#### ViewPager+tabLayout实现Tab效果  


### 布局文件见上

### activity代码  

```
package com.zj.materialdesign5;

import android.content.res.AssetManager;
import android.graphics.Color;
import android.os.Bundle;
import android.support.design.widget.CollapsingToolbarLayout;
import android.support.design.widget.TabLayout;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.View;

import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class MainActivity extends AppCompatActivity {

    ViewPager mViewPager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Toolbar mToolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(mToolbar);
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        mToolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                onBackPressed();
            }
        });
        //使用CollapsingToolbarLayout必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上则不会显示
        CollapsingToolbarLayout mCollapsingToolbarLayout = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
        mCollapsingToolbarLayout.setTitle("MyCollapsingToolbar");
        //通过CollapsingToolbarLayout修改字体颜色
        mCollapsingToolbarLayout.setExpandedTitleColor(Color.WHITE);//设置还没收缩时状态下字体颜色
        mCollapsingToolbarLayout.setCollapsedTitleTextColor(Color.GREEN);//设置收缩后Toolbar上字体的颜色


        //设置ViewPager
        mViewPager = (ViewPager) findViewById(R.id.viewpager);
        setupViewPager(mViewPager);

//给TabLayout增加Tab, 并关联ViewPager
        TabLayout tabLayout = (TabLayout) findViewById(R.id.sliding_tabs);
        tabLayout.addTab(tabLayout.newTab().setText("内容简介"));
        tabLayout.addTab(tabLayout.newTab().setText("作者简介"));
        tabLayout.addTab(tabLayout.newTab().setText("目录"));
        tabLayout.setupWithViewPager(mViewPager);


    }

    private void setupViewPager(ViewPager mViewPager) {
        MyPagerAdapter adapter = new MyPagerAdapter(getSupportFragmentManager());
        adapter.addFragment(DetailFragment.newInstance(getAsset("book_content.txt")), "内容简介");
        adapter.addFragment(DetailFragment.newInstance(getAsset("book_author.txt")), "作者简介");
        adapter.addFragment(DetailFragment.newInstance(getAsset("book_menu.txt")), "目录");
        mViewPager.setAdapter(adapter);
    }

    private String getAsset(String fileName) {
        AssetManager am = getResources().getAssets();
        InputStream is = null;
        try {
            is = am.open(fileName, AssetManager.ACCESS_BUFFER);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return new Scanner(is).useDelimiter("\\Z").next();
    }


    static class MyPagerAdapter extends FragmentPagerAdapter {
        private final List<Fragment> mFragments = new ArrayList<>();
        private final List<String> mFragmentTitles = new ArrayList<>();

        public MyPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        public void addFragment(Fragment fragment, String title) {
            mFragments.add(fragment);
            mFragmentTitles.add(title);
        }

        @Override
        public Fragment getItem(int position) {
            return mFragments.get(position);
        }

        @Override
        public int getCount() {
            return mFragments.size();
        }

        @Override
        public CharSequence getPageTitle(int position) {
            return mFragmentTitles.get(position);
        }
    }






}
```  

### DetailFragment布局文件

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:id="@+id/tvInfo"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:padding="10dp"
            android:textColor="@color/primary_text"
            android:textSize="16sp" />
    </LinearLayout>

</android.support.v4.widget.NestedScrollView>
```

### DetailFragment代码

```
package com.zj.materialdesign5;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

/**
 * Created by Chenyc on 2015/6/29.
 */
public class DetailFragment extends Fragment {

    public static DetailFragment newInstance(String info) {
        Bundle args = new Bundle();
        DetailFragment fragment = new DetailFragment();
        args.putString("info", info);
        fragment.setArguments(args);
        return fragment;
    }


    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_detail, null);
        TextView tvInfo = (TextView) view.findViewById(R.id.tvInfo);
        tvInfo.setText(getArguments().getString("info"));
//        tvInfo.setOnClickListener(new View.OnClickListener() {
//            @Override
//            public void onClick(View v) {
//                Snackbar.make(v,"hello",Snackbar.LENGTH_SHORT).show();
//            }
//        });
        return view;
    }
}
```   


### 代码解释 

- 为什么不用new DetailFragment而要用newInstance，简单说是因为activity发生变化时仍旧保留数据，详情参见链接   
[【译】使用newInstance()来实例化fragment - 陈哈哈 - 博客园](http://www.cnblogs.com/kissazi2/p/4127336.html)
[Android解惑 - 为什么要用Fragment.setArguments(Bundle bundle)来传递参数 - 推酷](http://www.tuicool.com/articles/j22E3u)

- getAsset函数，是从asserts文件夹中读取资源，在该文件夹下，有book_content.txt，book_author.txt，book_menu.txt三个文件，通过这个函数读取出来

### 完成，效果如下
![这里写图片描述](http://img.blog.csdn.net/20160408161440287)
















