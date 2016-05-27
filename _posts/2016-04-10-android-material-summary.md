---
layout: post
title: Material Design综合实例
date: 2016-4-10
categories: blog
tags: [Material Design]
description: Material Design综合实例
---

### 背景知识  

1. drawlayout的使用     
2.  recycleView的使用   
3.  CardView的使用  
4.  一些开源动画库的使用  

### ImageView的scaleType属性与adjustViewBounds属性  ，参考链接：
[ImageView的android:adjustViewBounds属性 - - ITeye技术网站](http://892848153.iteye.com/blog/2071774)   
[Android ImageView的scaleType属性与adjustViewBounds属性 - Android移动开发技术文章_手机开发 - 红黑联盟](http://www.2cto.com/kf/201411/348601.html)  

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160410153734528)
![这里写图片描述](http://img.blog.csdn.net/20160410153749330)
![这里写图片描述](http://img.blog.csdn.net/20160410153806997)


### 程序开始动画实现  

### 第一步：使用开源库
[gzu-liyujiang/ViewAnimator: A fluent Android animation library。安卓动画库](https://github.com/gzu-liyujiang/ViewAnimator)  

在depencies中加入

```
compile 'com.github.florent37:viewanimator:1.0.2@aar'
compile 'com.nineoldandroids:library:2.4.0'
```   

### java代码 

```
package com.zj.materialfood;

import android.content.Intent;
import android.graphics.Color;
import android.os.Bundle;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.view.animation.AccelerateDecelerateInterpolator;
import android.widget.ImageView;
import android.widget.TextView;

import com.github.florent37.viewanimator.AnimationListener;
import com.github.florent37.viewanimator.ViewAnimator;

import java.util.Locale;

/**
 * Created by jjx on 2016/4/9.
 */
public class StartActivity extends AppCompatActivity{


    ImageView image;
    ImageView mountain;
    TextView text;
    TextView percent;
    android.os.Handler handler=new android.os.Handler()
    {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_start);

        image = (ImageView) findViewById(R.id.image1);
        mountain = (ImageView) findViewById(R.id.mountain);
        text = (TextView) findViewById(R.id.text1);
        percent = (TextView) findViewById(R.id.percent);


        animateParallel();


        /*handler.postDelayed(new Runnable() {

            @Override
            public void run() {
                // TODO Auto-generated method stub
                //进入主页面
                Intent intent = new Intent(StartActivity.this, MainActivity.class);
                startActivity(intent);

            }
        }, 6000);*/


    }

    private void animateParallel() {
        ViewAnimator.animate(mountain, image)
                .dp().translationY(-1000, 0)
                .alpha(0, 1)

                .andAnimate(percent)
                .scale(0, 1)

                .andAnimate(text)
                .dp().translationY(1000, 0)
                .textColor(Color.BLACK, Color.WHITE)
                .backgroundColor(Color.WHITE, Color.BLACK)

                .waitForHeight()
                .interpolator(new AccelerateDecelerateInterpolator())
                .duration(1000)

                .thenAnimate(percent)
                .custom(new AnimationListener.Update<TextView>() {
                    @Override
                    public void update(TextView view, float value) {
                        value = value * 100;
                        view.setText(String.format(Locale.US, "%.0f%%", value));
                    }
                }, 0, 1)

                .andAnimate(image)
                .rotation(0, 360)
                .onStop(new AnimationListener.Stop() {
                    @Override
                    public void onStop() {
                        Intent intent = new Intent(StartActivity.this, MainActivity.class);
                        startActivity(intent);
                    }
                })

                .duration(1000)



                .start();






    }


}
```   


### 主界面效果实现  

### 使用开源库
[gabrielemariotti/RecyclerViewItemAnimators: An Android library which provides simple Item animations to RecyclerView items](https://github.com/gabrielemariotti/RecyclerViewItemAnimators)

### 添加依赖  

```
dependencies {
    compile 'com.github.gabrielemariotti.recyclerview:recyclerview-animators:0.3.0-SNAPSHOT@aar'
}
```

### 添加仓库  

```
repositories {
    maven { url "https://oss.sonatype.org/content/repositories/snapshots/" }
}
```   


### 实现  

```
package com.zj.materialfood;

import android.content.Intent;
import android.os.Bundle;
import android.os.Message;
import android.support.design.widget.NavigationView;
import android.support.v4.widget.DrawerLayout;
import android.support.v7.app.ActionBarDrawerToggle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.Toolbar;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;

import java.util.List;

import it.gmariotti.recyclerview.adapter.SlideInBottomAnimatorAdapter;
import it.gmariotti.recyclerview.adapter.SlideInRightAnimatorAdapter;

public class MainActivity extends AppCompatActivity {
    RecyclerView recyclerView;


    private List<String> mDatas;
    private HomeAdapter mAdapter;
    int ids[]=new int[]{R.drawable.food1,R.drawable.food2,R.drawable.food3,R.drawable.food4,R.drawable.food5,R.drawable.food6,R.drawable.food7,R.drawable.food8};
    Toolbar toolbar;

    SlideInBottomAnimatorAdapter slideInBottomAnimatorAdapter;
    SlideInRightAnimatorAdapter slideInRightAnimatorAdapter;

    DrawerLayout mDrawerLayout;
    ActionBarDrawerToggle mDrawerToggle;
    NavigationView mNavigationView;

    android.os.Handler handler=new android.os.Handler()
    {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        toolbar= (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        setTitle("天天美食");


        //设置DrawerLayout
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerToggle = new ActionBarDrawerToggle(this, mDrawerLayout, toolbar,
                R.string.drawer_open, R.string.drawer_close)
        {
            //关闭侧边栏时响应
            @Override
            public void onDrawerClosed(View drawerView) {
                super.onDrawerClosed(drawerView);
            }
            //打开侧边栏时响应
            @Override
            public void onDrawerOpened(View drawerView) {
                super.onDrawerOpened(drawerView);

            }
        };
        mDrawerToggle.syncState();
        mDrawerLayout.setDrawerListener(mDrawerToggle);

        //设置NavigationView点击事件
        mNavigationView = (NavigationView) findViewById(R.id.navigation_view);
        setupDrawerContent(mNavigationView);
        //设置NavigationView点击事件


        recyclerView= (RecyclerView) findViewById(R.id.id_recyclerview);
        recyclerView.setLayoutManager(new GridLayoutManager(this, 4));
        mAdapter=new HomeAdapter();
        slideInBottomAnimatorAdapter=new SlideInBottomAnimatorAdapter(mAdapter,recyclerView);

        //recyclerView.setAdapter(slideInRightAnimatorAdapter);
        handler.postDelayed(new Runnable() {

            @Override
            public void run() {
                // TODO Auto-generated method stub
                //进入主页面
                recyclerView.setAdapter(slideInBottomAnimatorAdapter);
            }
        }, 1000);

        //recyclerView.setAdapter(mAdapter);



    }

    private void setupDrawerContent(NavigationView mNavigationView) {



    }

    class HomeAdapter extends RecyclerView.Adapter<HomeAdapter.MyViewHolder>
    {




        @Override
        public HomeAdapter.MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

            MyViewHolder holder=new MyViewHolder(LayoutInflater.from(MainActivity.this).inflate(R.layout.item_main,parent,false));
            return holder;
        }

        @Override
        public void onBindViewHolder(HomeAdapter.MyViewHolder holder, int position) {



            holder.iv.setBackgroundResource(ids[position]);

        }

        @Override
        public int getItemCount() {
            return 8;
        }

        class MyViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener
        {

            ImageView iv;

            public MyViewHolder(View view)
            {
                super(view);
                iv = (ImageView) view.findViewById(R.id.iv_food);
                view.setOnClickListener(this);
            }

            @Override
            public void onClick(View view) {

                Intent intent=new Intent(MainActivity.this,FoodCardActivity.class);
                startActivity(intent);
            }
        }
    }
}
```   

### 食物列表界面

### 注意，笔者在这里碰到了很大的坑   
老是报空指针异常，后来发现问题出在设置TextView时，setTextView时总是出错，搞了很久都不明白，后来才发现是在 public ViewHolder(View v) 方法中，绑定View与ViewHolder时，findViewById没有用 v.findViewById(R.id.iv_food_item)，而是直接findViewById(R.id.iv_food_item)了。


### 总体页面布局  

```
<?xml version="1.0" encoding="utf-8"?>

    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent">


        <android.support.v7.widget.Toolbar
            android:id="@+id/food_toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignParentTop="true"
            android:title="天天美食"
            android:background="?attr/colorPrimary"
            >
        </android.support.v7.widget.Toolbar>



        <android.support.v7.widget.RecyclerView
            android:id="@+id/food_recyclerview"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_below="@+id/food_toolbar"
            android:scrollbars="none" />

    </RelativeLayout>
``` 

### recyView的item布局  

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginBottom="@dimen/card_margin"
    android:layout_marginLeft="@dimen/card_margin"
    android:layout_marginRight="@dimen/card_margin"
    android:clickable="true"
    app:cardCornerRadius="10dp"
    app:cardElevation="10dp">

    <LinearLayout
        style="@style/CardView.Content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
    <ImageView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/iv_food_item"
        android:layout_width="109dp"
        android:layout_height="135dp"
        android:adjustViewBounds="true"
        android:background="@drawable/food6"
        android:gravity="center"
        android:layout_margin="4dp"
        />
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:orientation="vertical">

            <TextView
                android:id="@+id/tvTitle"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:textAppearance="@style/TextAppearance.AppCompat.Title"
                android:text="麻婆豆腐"
                android:textColor="@color/primary_text" />

         <TextView
                android:id="@+id/tvDesc"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="2dp"
                android:text="@string/book_description_1"
                android:textColor="@color/secondary_text" />
        </LinearLayout>


    </LinearLayout>
</android.support.v7.widget.CardView>
```   

### java代码实现  

```
package com.zj.materialfood;

import android.content.Intent;
import android.os.Bundle;
import android.support.design.widget.Snackbar;
import android.support.v4.app.NavUtils;
import android.support.v4.app.TaskStackBuilder;
import android.support.v7.app.ActionBar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.Toolbar;
import android.util.TypedValue;
import android.view.LayoutInflater;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;

/**
 * Created by jjx on 2016/4/10.
 */
public class FoodCardActivity extends AppCompatActivity{

    Toolbar toolbar;
    RecyclerView recyclerView;
    MyAdapter mAdapter;
    int img_ids[]=new int[]{R.drawable.lvfood1,R.drawable.lvfood2,R.drawable.lvfood3,R.drawable.lvfood4,R.drawable.lvfood5,R.drawable.lvfood6,R.drawable.lvfood7,R.drawable.lvfood8,R.drawable.lvfood9,R.drawable.lvfood10};
    String titles[]=new String[]{"麻婆豆腐","灯影牛肉","夫妻肺片","蒜泥白肉","白油豆腐","鱼香肉丝","泉水豆花","宫保鸡丁 ","东坡墨鱼 ","麻辣香锅"};
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_foodcard);
        toolbar= (Toolbar) findViewById(R.id.food_toolbar);
        setSupportActionBar(toolbar);
        setTitle("美食菜单");
        ActionBar actionBar=getSupportActionBar();
        actionBar.setDisplayHomeAsUpEnabled(true);

        recyclerView= (RecyclerView) findViewById(R.id.food_recyclerview);
        recyclerView.setHasFixedSize(true);
        LinearLayoutManager linearLayoutManager=new LinearLayoutManager(FoodCardActivity.this);
        recyclerView.setLayoutManager(linearLayoutManager);



        recyclerView.setItemAnimator(new DefaultItemAnimator());
        mAdapter = new MyAdapter();
        recyclerView.setAdapter(mAdapter);


    }
    public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {



        private final TypedValue mTypedValue = new TypedValue();

        // Provide a reference to the views for each data item
        // Complex data items may need more than one view per item, and
        // you provide access to all the views for a data item in a view holder
        public class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
            // each data item is just a string in this case
            public TextView mTextView;
            public ImageView iv_food;
            public TextView textDesc;

            public ViewHolder(View v) {
                super(v);
                mTextView = (TextView) v.findViewById(R.id.tvTitle);
                iv_food= (ImageView) v.findViewById(R.id.iv_food_item);
                textDesc= (TextView) v.findViewById(R.id.tvDesc);
                v.setOnClickListener(this);
            }

            @Override
            public void onClick(View view) {
                String text = "I Love " + mTextView.getText() + ".";
                Snackbar.make(view, text, Snackbar.LENGTH_SHORT).show();
            }
        }

        // Provide a suitable constructor (depends on the kind of dataset)

        @Override
        public MyAdapter.ViewHolder onCreateViewHolder(ViewGroup parent,
                                                       int viewType) {
            // create a new view
            View v = LayoutInflater.from(parent.getContext())
                    .inflate(R.layout.food_item, parent, false);

            // set the view's size, margins, paddings and layout parameters
            ViewHolder vh = new ViewHolder(v);
            return vh;
        }

        // Replace the contents of a view (invoked by the layout manager)
        @Override
        public void onBindViewHolder(ViewHolder holder, int position) {
            // - get element from your dataset at this position
            // - replace the contents of the view with that element
            //holder.mTextView.setText(mDataset[position]);
            holder.mTextView.setText(titles[position]);
            holder.iv_food.setBackgroundResource(img_ids[position]);
            holder.textDesc.setText("很好吃的一道菜 \n东南第一佳味，天下之至美 \n地址：武珞路30号 \n234人吃过 \n价格: 88.00元");
        }

        // Return the size of your dataset (invoked by the layout manager)
        @Override
        public int getItemCount() {
            return 10;
        }
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch(item.getItemId())
        {
            case android.R.id.home:
                Intent upIntent = NavUtils.getParentActivityIntent(this);
                if (NavUtils.shouldUpRecreateTask(this, upIntent)) {
                    TaskStackBuilder.create(this)
                            .addNextIntentWithParentStack(upIntent)
                            .startActivities();
                } else {
                    upIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
                    NavUtils.navigateUpTo(this, upIntent);
                }
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }
}
```  


### 完成














