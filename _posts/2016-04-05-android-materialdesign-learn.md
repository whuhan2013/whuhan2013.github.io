---
layout: post
title: Material Design入门
date: 2016-4-5
categories: blog
tags: [Material Design]
description: Material Design入门
---

### 本文主要包括以下内容

1. ToolBar的使用      
2.  RecyclerView的定义与使用

### ToolBar  

风格 (style)  
界面 (layout)        
程序 (java)       

### 首先自定义一个theme,并将AppTheme的parent改成我们自定义的theme

```
(style.xml)
<resources>

    <style name="BaseAppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/primary</item>
        <item name="colorPrimaryDark">@color/primary_dark</item>
        <item name="colorAccent">@color/accent</item>
        <item name="android:windowBackground">@color/window_background</item>
    </style>

    <style name="AppTheme" parent="BaseAppTheme" />

</resources>
```  

其中，colorPrimaryDark是状态栏底色    
App bar 底色     
navigationBarColor 导航栏底色   
主视窗底色：windowBackground    

### 在这里定义整个界面的底色

### 在layout中定义toolbar 

```
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:background="?attr/colorPrimary"
        >
    </android.support.v7.widget.Toolbar>
```  

### 在java代码中 

有以下方法  

setNavigationIcon: 
即设定 up button 的图标，因为 Material 的介面，在 Toolbar这里的 up button样式也就有別于过去的 ActionBar 哦。

setLogo   
APP 的图标。       

setTitle   
主标题。     

setSubtitle     
副标题。  

setOnMenuItemClickListener
设定菜单各按鈕的动作。  

### 效果如下
![这里写图片描述](http://img.blog.csdn.net/20160405213515054)

### 实现

```
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        toolbar=(Toolbar)findViewById(R.id.toolbar);
        mRecyclerView= (RecyclerView) findViewById(R.id.recyclerView);

        // App Logo
        toolbar.setLogo(R.drawable.ic_launcher);
// Title
        toolbar.setTitle("My Title2");
// Sub Title
        toolbar.setSubtitle("Sub title");

        setSupportActionBar(toolbar);

// Navigation Icon 要設定在 setSupoortActionBar 才有作用
// 否則會出現 back button
        toolbar.setNavigationIcon(R.drawable.ab_android);


        toolbar.setOnMenuItemClickListener(onMenuItemClick);

        

    }
```  

### 在menu中定义菜单  

```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      tools:context=".MainActivity">
      <item android:id="@+id/action_edit"
          android:title="edit"
          android:orderInCategory="80"
          android:icon="@drawable/ab_edit"
          app:showAsAction="ifRoom" />

      <item android:id="@+id/action_share"
          android:title="share"
          android:orderInCategory="90"
          android:icon="@drawable/ab_share"
          app:showAsAction="ifRoom" />

      <item android:id="@+id/action_settings"
          android:title="setting"
          android:orderInCategory="100"
          app:showAsAction="never"/>
</menu>
```

### 设置图标与图标响应事件
1. 并且在onCreateOptionsMenu方法中绑定，     
2. 然后用OnMenuItemClickListener 设置监听       
3. toolbar.setOnMenuItemClickListener(onMenuItemClick)，将监听赋给toolbar  


```
 private Toolbar.OnMenuItemClickListener onMenuItemClick = new Toolbar.OnMenuItemClickListener() {
        @Override
        public boolean onMenuItemClick(MenuItem menuItem) {
            String msg = "";
            switch (menuItem.getItemId()) {
                case R.id.action_edit:
                    msg += "Click edit";
                    break;
                case R.id.action_share:
                    msg += "Click share";
                    break;
                case R.id.action_settings:
                    msg += "Click setting";
                    break;
            }

            if(!msg.equals("")) {
                Toast.makeText(MainActivity.this, msg, Toast.LENGTH_SHORT).show();
            }
            return true;
        }
    };

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {

        getMenuInflater().inflate(R.menu.menu_main,menu);
        return super.onCreateOptionsMenu(menu);
    }
```

### 参考链接：
[android：ToolBar详解（手把手教程） - 泡在网上的日子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1118/2006.html)


### RecycleView的使用   

### 注意：RecycleView的使用要在dependencies中加入compile 'com.android.support:design:23.2.1'，不然会报找不到的错误

RecyclerView出现已经有一段时间了，相信大家肯定不陌生了，大家可以通过导入support-v7对其进行使用。 
据官方的介绍，该控件用于在有限的窗口中展示大量数据集，其实这样功能的控件我们并不陌生，例如：ListView、GridView。

那么有了ListView、GridView为什么还需要RecyclerView这样的控件呢？整体上看RecyclerView架构，提供了一种插拔式的体验，高度的解耦，异常的灵活，通过设置它提供的不同LayoutManager，ItemDecoration , ItemAnimator实现令人瞠目的效果。


- 你想要控制其显示的方式，请通过布局管理器LayoutManager        
- 你想要控制Item间的间隔（可绘制），请通过ItemDecoration     
- 你想要控制Item增删的动画，请通过ItemAnimator        
- 你想要控制点击、长按事件，请自己写    

### 基本使用   

```
mRecyclerView = findView(R.id.id_recyclerview);
//设置布局管理器
mRecyclerView.setLayoutManager(layout);
//设置adapter
mRecyclerView.setAdapter(adapter)
//设置Item增加、移除动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
//添加分割线
mRecyclerView.addItemDecoration(new DividerItemDecoration(
                getActivity(), DividerItemDecoration.HORIZONTAL_LIST));
```

### 简单实现  

```
package com.zj.material2recy;

import android.os.Bundle;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.StaggeredGridLayoutManager;
import android.support.v7.widget.Toolbar;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    RecyclerView recyclerView;

    private List<String> mDatas;
    private HomeAdapter mAdapter;
    Toolbar toolbar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        toolbar= (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        setTitle("KSFHFSSF");

        initData();
        recyclerView= (RecyclerView) findViewById(R.id.id_recyclerview);
        recyclerView.setHasFixedSize(true);


        //recyclerView.setLayoutManager(new LinearLayoutManager(this));
        //recyclerView.setLayoutManager(new GridLayoutManager(MainActivity.this,3));
        recyclerView.setLayoutManager(new StaggeredGridLayoutManager(4,StaggeredGridLayoutManager.VERTICAL));
        mAdapter = new HomeAdapter();
        recyclerView.setAdapter(mAdapter);

        recyclerView.setItemAnimator(new DefaultItemAnimator());





    }

    public void add(View view)
    {
       mAdapter.add(view);
    }

    public void remove(View view)
    {
        mAdapter.remove(view);
    }



   List<Integer> mHeights;
    protected void initData()
    {
        mDatas = new ArrayList<String>();
        for (int i = 'A'; i < 'z'; i++)
        {
            mDatas.add("" + (char) i);
        }

        mHeights = new ArrayList<Integer>();
        for (int i = 0; i < mDatas.size(); i++)
        {
            mHeights.add( (int) (100 + Math.random() * 300));
        }
    }

    class HomeAdapter extends RecyclerView.Adapter<HomeAdapter.MyViewHolder>
    {

        public void add(View view)
        {
            mDatas.add(2,"insert");
            notifyItemInserted(2);
        }

        public void remove(View view)
        {
            mDatas.remove(8);
            notifyItemRemoved(8);
        }


        @Override
        public HomeAdapter.MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

            MyViewHolder holder=new MyViewHolder(LayoutInflater.from(MainActivity.this).inflate(R.layout.item_main,parent,false));
            return holder;
        }

        @Override
        public void onBindViewHolder(HomeAdapter.MyViewHolder holder, int position) {
            int temp= (int) (Math.random()*5+10);
            //holder.tv.setHeight(temp);

            ViewGroup.LayoutParams lp =  holder.tv.getLayoutParams();
            lp.height = mHeights.get(position);
            holder.tv.setLayoutParams(lp);


            holder.tv.setText(mDatas.get(position));

        }

        @Override
        public int getItemCount() {
            return mDatas.size();
        }

        class MyViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener
        {

            TextView tv;

            public MyViewHolder(View view)
            {
                super(view);
                tv = (TextView) view.findViewById(R.id.id_num);
                view.setOnClickListener(this);
            }

            @Override
            public void onClick(View view) {

                String text=tv.getText().toString();
                int positon=recyclerView.getChildAdapterPosition(view);
                Snackbar.make(view, text+positon, Snackbar.LENGTH_SHORT).show();
            }
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }
}
```

### 布局文件  

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">


    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:title="afbd"
        android:background="?attr/colorPrimary"
        >
    </android.support.v7.widget.Toolbar>
    <LinearLayout
        android:layout_below="@id/toolbar"
        android:id="@+id/bt_ctr"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="add"
        android:onClick="add"/>
        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="remove"
            android:onClick="remove"/>
    </LinearLayout>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/id_recyclerview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/bt_ctr"
        android:scrollbars="none" />

</RelativeLayout>
```  

### recycleView的item布局

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minHeight="60dp"
    android:clickable="true"
    android:gravity="center_vertical"
    android:orientation="vertical"
    >

    <TextView
        android:id="@+id/id_num"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="10dp"
        android:text="abfd"
        android:gravity="center"

        android:textColor="@android:color/black"
        android:textSize="18sp" />
</RelativeLayout>
```  

### 效果如下：实现了瀑布流，添加，删除时的动画    
  
### 注意，实现瀑布流时一定要将item布局文件的高度设置为wrap content，而不是定长，不然无法改变高度，
导致所有节点一样高，无法达到瀑布流效果     
![这里写图片描述](http://img.blog.csdn.net/20160406171619047)


### 分割线的实现详见参考链接:
[Android RecyclerView 使用完全解析 体验艺术般的控件 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/45059587) 


### layoutManger的用法

RecyclerView.LayoutManager吧，这是一个抽象类，好在系统提供了3个实现类,分别实现listView,grideView,瀑布流：

- LinearLayoutManager 现行管理器，支持横向、纵向。   
- GridLayoutManager 网格布局管理器   
- StaggeredGridLayoutManager 瀑布就式布局管理器     

```
//mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        mRecyclerView.setLayoutManager(new GridLayoutManager(this,4));
```  

### ItemAnimator的用法  

ItemAnimator也是一个抽象类，好在系统为我们提供了一种默认的实现类，期待系统多 
添加些默认的实现。

借助默认的实现，当Item添加和移除的时候，添加动画效果很简单:

```
// 设置item动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());
```  

- 注意，这里更新数据集不是用adapter.notifyDataSetChanged()而是 
notifyItemInserted(position)与notifyItemRemoved(position) 
否则没有动画效果。 
上述为adapter中添加了两个方法：


```
public void addData(int position) {
        mDatas.add(position, "Insert One");
        notifyItemInserted(position);
    }

    public void removeData(int position) {
            mDatas.remove(position);
        notifyItemRemoved(position);
    }
```    

### 点击事件监听实现 

```
class MyViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener
        {

            TextView tv;

            public MyViewHolder(View view)
            {
                super(view);
                tv = (TextView) view.findViewById(R.id.id_num);
                view.setOnClickListener(this);
            }

            @Override
            public void onClick(View view) {

                String text=tv.getText().toString();
                int positon=recyclerView.getChildAdapterPosition(view);
                Snackbar.make(view, text+positon, Snackbar.LENGTH_SHORT).show();
            }
        }
    }
```    

### 首先setOnclickListener，由MyViewHolder 接收并处理  

### 完成：参考链接如下

[Android RecyclerView 使用完全解析 体验艺术般的控件 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/45059587)

[动画效果源码](https://github.com/gabrielemariotti/RecyclerViewItemAnimators)














