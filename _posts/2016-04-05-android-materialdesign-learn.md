---
layout: post
title: Material Design入门
date: 2016-4-5
categories: blog
tags: [android]
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













