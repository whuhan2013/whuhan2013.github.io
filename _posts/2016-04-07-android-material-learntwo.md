---
layout: post
title: Material Design入门(二）
date: 2016-4-7
categories: blog
tags: [android]
description: Material Design入门(二）
---

### 本文主要包括以下内容

1. 侧滑菜单DrawerLayout实现  

### DrawerLayout介绍    
drawerLayout是Support Library包中实现了侧滑菜单效果的控件，可以说drawerLayout是因为第三方控件如MenuDrawer等的出现之后，google借鉴而出现的产物。drawerLayout分为侧边菜单和主内容区两部分，侧边菜单可以根据手势展开与隐藏（drawerLayout自身特性），主内容区的内容可以随着菜单的点击而变化（这需要使用者自己实现）


### drawlayout实现   

### main布局文件      
```
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">


       <LinearLayout
           android:layout_width="match_parent"
           android:layout_height="match_parent">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            app:layout_collapseMode="pin"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
       </LinearLayout>



    <android.support.design.widget.NavigationView
        android:id="@+id/navigation_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:headerLayout="@layout/navigation_header"
        app:menu="@menu/drawer" />


</android.support.v4.widget.DrawerLayout>
```    

### 其中侧滑菜单位置是start,并且包括了headerLayout与menu  

### headerLayout实现

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/lib/com.zj.material3navigation"
    android:layout_width="match_parent"
    android:layout_height="192dp"
    android:background="?attr/colorPrimaryDark"
    android:gravity="center"
    android:orientation="vertical"
    android:padding="16dp"
    android:theme="@style/ThemeOverlay.AppCompat.Dark">


    <de.hdodenhof.circleimageview.CircleImageView
        android:id="@+id/profile_image"
        android:layout_width="72dp"
        android:layout_height="72dp"
        android:layout_marginTop="20dp"
        android:src="@mipmap/profile"
        app:border_color="@color/primary_light"
        app:border_width="2dp" />


    <TextView
        android:layout_marginTop="10dp"
        android:textSize="18sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="APP开发者"
        android:gravity="center"
        android:textAppearance="@style/TextAppearance.AppCompat.Body1"
        />

</LinearLayout>
```    

### 注意，由于使用了CircleImageView，要在depencyies中加入   

```
compile 'de.hdodenhof:circleimageview:1.3.0'
```   
并且由于jcenter中库有限，可能还要加入  

```

allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
    }
}
```

### menu菜单实现  

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group android:checkableBehavior="single">
        <item
            android:id="@+id/navigation_item_example"
            android:icon="@drawable/ic_favorite"
            android:title="@string/navigation_example" />
        <item
            android:id="@+id/navigation_item_blog"
            android:icon="@drawable/ic_favorite"
            android:title="@string/navigation_my_blog" />

        <item
            android:id="@+id/navigation_item_about"
            android:icon="@drawable/ic_favorite"
            android:title="@string/navigation_about" />
    </group>

</menu>
```   

### menu也可以设置子menu,下面是一个例子  

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
 
    <group
        android:checkableBehavior="single">
 
        <item
            android:id="@+id/drawer_home"
            android:checked="true"
            android:icon="@drawable/ic_home_black_24dp"
            android:title="@string/home"/>
 
 
        <item
            android:id="@+id/section"
            android:icon="@drawable/ic_more_horiz_black_24dp"
            android:title="分组1">
            <menu>
                <item
                    android:id="@+id/drawer_favourite"
                    android:icon="@drawable/ic_favorite_black_24dp"
                    android:title="@string/favourite"/>
 
                <item
                    android:id="@+id/drawer_downloaded"
                    android:icon="@drawable/ic_file_download_black_24dp"
                    android:title="@string/downloaded"/>
            </menu>
        </item>
 
        <item
            android:id="@+id/section2"
            android:title="分组2">
            <menu>
                <item
                    android:id="@+id/drawer_more"
                    android:icon="@drawable/ic_more_horiz_black_24dp"
                    android:title="@string/more"/>
 
                <item
                    android:id="@+id/drawer_settings"
                    android:icon="@drawable/ic_settings_black_24dp"
                    android:title="@string/settings"/>
            </menu>
        </item>
 
 
    </group>
</menu>
```  

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160407125259281)

### java代码的实现  

```
package com.zj.material3navigation;


import android.os.Bundle;
import android.support.design.widget.NavigationView;
import android.support.v4.widget.DrawerLayout;
import android.support.v7.app.ActionBarDrawerToggle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.MenuItem;
import android.view.View;


public class MainActivity extends AppCompatActivity {
    Toolbar mToolbar;
    DrawerLayout mDrawerLayout;
    ActionBarDrawerToggle mDrawerToggle;
    NavigationView mNavigationView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //设置Toolbar
        mToolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(mToolbar);
        setTitle("startNow");

        //设置DrawerLayout
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        mDrawerToggle = new ActionBarDrawerToggle(this, mDrawerLayout, mToolbar,
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


    }
   //点击侧边栏菜单的响应事件
    private void setupDrawerContent(NavigationView navigationView) {
        navigationView.setNavigationItemSelectedListener(
                new NavigationView.OnNavigationItemSelectedListener() {
                    @Override
                    public boolean onNavigationItemSelected(MenuItem menuItem) {
                        switch (menuItem.getItemId()) {
                            case R.id.navigation_item_example:
                                //switchToExample();
                                break;
                            case R.id.navigation_item_blog:
                                //switchToBlog();
                                break;
                            case R.id.navigation_item_about:
                                //switchToAbout();
                                break;

                        }
                        menuItem.setChecked(true);
                        mDrawerLayout.closeDrawers();
                        return true;
                    }
                });
    }


}
```

### 实现了打开与关闭侧边栏的响应事件，点击侧边栏按钮的响应事件等

### 参考链接：   
[Design Support Library (I): Navigation View的使用 - 泡在网上的日子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0608/3011.html)

[android官方侧滑菜单DrawerLayout详解 - 泡在网上的日子](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/0925/1713.html)     

### 效果如下：

![这里写图片描述](http://img.blog.csdn.net/20160407125741283)











