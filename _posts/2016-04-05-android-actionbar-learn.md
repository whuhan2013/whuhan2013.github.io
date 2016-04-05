---
layout: post
title: Android之ActionBar
date: 2016-4-5
categories: blog
tags: [android]
description: Android之ActionBar
---

 ### 本文主要包括以下内容  

1. ActionBar的显示及隐藏，添加图标，返回主页      
2.  ActionBar添加ActionView,添加ActionProvider    
3.  ActionBar实现Tab   
4.  ActionBar添加下拉列表    


### ActionBar介绍  

在Android 3.0中除了我们重点讲解的Fragment外，Action Bar也是一个非常重要的交互元素，Action Bar取代了传统的tittle bar和menu，在程序运行中一直置于顶部，对于Android平板设备来说屏幕更大它的标题使用Action Bar来设计可以展示更多丰富的内容，方便操控  

### ActioinBar功能  

<1> ActionBar的图标，可显示软件图标，也可用其他图标代替。当软件不在最高级页面时，图标左侧会显示一个左箭头，用户可以通过这个箭头向上导航；
 
　　<2> 如果你的应用要在不同的View中显示数据，这部分允许用户来切换视图。一般的作法是用一个下拉菜单或者是Tab选项卡。如果只有一个界面，那这里可以显示应用程序的标题或者是更长一点的商标信息；
 
　　<3> 两个action按钮，这里放重要的按钮功能，为用户进行某项操作提供直接的访问；
 
　　<4> overflow按钮，放不下的按钮会被置于“更多...”菜单项中，“更多...”菜单项是以下拉形式实现的


### ActionBar修改文字与图标

第一种方法  

```
 <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:logo="@drawable/cnblog_icon"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        >
```

在mainfest文件中设置label与logo,但图片却显示不出来，所心采用第二种方式，在代码中改变   

```
ActionBar actionBar = getSupportActionBar();
        actionBar.setDisplayShowHomeEnabled(true);
        actionBar.setLogo(R.mipmap.ic_launcher);
        actionBar.setDisplayUseLogoEnabled(true);

        setTitle("abc");
```   

### 添加按钮  

ActionBar还可以根据应用程序当前的功能来提供与其相关的Action按钮，这些按钮都会以图标或文字的形式直接显示在ActionBar上。当然，如果按钮过多，ActionBar上显示不完，多出的一些按钮可以隐藏在overflow里面（最右边的三个点就是overflow按钮），点击一下overflow按钮就可以看到全部的Action按钮了。
 
当Activity启动的时候，系统会调用Activity的onCreateOptionsMenu()方法来取出所有的Action按钮，我们只需要在这个方法中去加载一个menu资源，并把所有的Action按钮都定义在资源文件里面就可以了。
 
那么我们先来看下menu资源文件该如何定义，代码如下所示：

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android" >

    <item
        android:id="@+id/user_p"
        android:icon="@drawable/icon_user_p"
        android:showAsAction="always"
        android:title="用户"/>
    <item
        android:id="@+id/write_p"
        android:icon="@drawable/icon_write_p"
        android:showAsAction="always"
        android:title="发布"/>
    <item
        android:id="@+id/favo_p"
        android:icon="@drawable/icon_favo_p"
        android:showAsAction="never"
        android:title="收藏"/>

</menu>
```

接着，重写Activity的onCreateOptionsMenu()方法，代码如下所示：

```
@Override 
public boolean onCreateOptionsMenu(Menu menu) {  
    MenuInflater inflater = getMenuInflater();  
    inflater.inflate(R.menu.menu_main, menu);  
return super.onCreateOptionsMenu(menu);  
}
```  

### 响应Action按钮的点击事件

当用户点击Action按钮的时候，系统会调用Activity的onOptionsItemSelected()方法，通过方法传入的MenuItem参数，我们可以调用它的getItemId()方法和menu资源中的id进行比较，从而辨别出用户点击的是哪一个Action按钮，比如：

```
@Override
    public boolean onOptionsItemSelected(MenuItem item) {
         switch (item.getItemId()) {  
         case R.id.user_p:  
                Toast.makeText(this, "你点击了“用户”按键！", Toast.LENGTH_SHORT).show();  
                return true;  
            case R.id.write_p:  
                Toast.makeText(this, "你点击了“发布”按键！", Toast.LENGTH_SHORT).show();  
                return true;  
            case R.id.favo_p:  
                Toast.makeText(this, "你点击了“收藏”按键！", Toast.LENGTH_SHORT).show();  
                return true;  
            default:  
                return super.onOptionsItemSelected(item);  
            }  
    }
```

### 效果如下 

![这里写图片描述](http://img.blog.csdn.net/20160405193055053)


### 返回主页实现 

首先设置setDisplayHomeAsUpEnable

```
ActionBar actionBar = getActionBar();  
actionBar.setDisplayHomeAsUpEnabled(true); 
``` 

第二步需要在AndroidManifest.xml中配置父Activity，如下所示：

```
<activity 
    android:name="com.yanis.actionbar.TabActivity"
    android:parentActivityName="com.yanis.actionbar.MainActivity" > 
</activity> 
```  

第三步则需要对android.R.id.home这个事件进行一些特殊处理，如下所示,重写方法：

```
@Override 
public boolean onOptionsItemSelected(MenuItem item) {  
    switch (item.getItemId()) {  
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
        ......  
    }  
}
```  

### 添加ActionView  

ActionView是一种可以在ActionBar中替换Action按钮的控件，它可以允许用户在不切换界面的情况下通过ActionBar完成一些较为丰富的操作。比如说，你需要完成一个搜索功能，就可以将SeachView这个控件添加到ActionBar中。
 
为了声明一个ActionView，我们可以在menu资源中通过actionViewClass属性来指定一个控件

```
<item
        android:id="@+id/action_search"
        android:actionViewClass="android.widget.SearchView"
        android:showAsAction="always"
        android:title="搜索"/>
```  

但这种写法似乎有兼容性问题，无法达到效果，在item中将actionViewClass去除，在代码中实现  

```
  MenuItem item=menu.findItem(R.id.action_search);
        item.setActionView(R.layout.searchview);
```

其中searchView如下  

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <SearchView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"></SearchView>

</LinearLayout>
``` 

### 添加Action Provider

和Action View有点类似，Action Provider也可以将一个Action按钮替换成一个自定义的布局。但不同的是，Action Provider能够完全控制事件的所有行为，并且还可以在点击的时候显示子菜单。
 
为了添加一个Action Provider，我们需要在<item>标签中指定一个actionViewClass属性，在里面填入Action Provider的完整类名。我们可以通过继承ActionProvider类的方式来创建一个自己的Action Provider，同时，Android也提供好了几个内置的Action Provider，比如说ShareActionProvider。
 
由于每个Action Provider都可以自由地控制事件响应，所以它们不需要在onOptionsItemSelected()方法中再去监听点击事件，而是应该在onPerformDefaultAction()方法中去执行相应的逻辑。
 
那么我们就先来看一下ShareActionProvider的简单用法吧，编辑menu资源文件，在里面加入ShareActionProvider的声明，如下所示：

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android" >

    <item 
        android:id="@+id/action_share" 
        android:actionProviderClass="android.widget.ShareActionProvider" 
        android:showAsAction="ifRoom" 
        android:title="分享" /> 
...
</menu>
``` 

接着剩下的事情就是通过Intent来定义出你想分享哪些东西了，我们只需要在onCreateOptionsMenu()中调用MenuItem的getActionProvider()方法来得到该ShareActionProvider对象，再通过setShareIntent()方法去选择构建出什么样的一个Intent就可以了。代码如下所示：

```
@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu_main, menu);
        MenuItem shareItem = menu.findItem(R.id.action_share);
        ShareActionProvider provider = (ShareActionProvider) shareItem
                .getActionProvider();
        provider.setShareIntent(getDefaultIntent());
        return super.onCreateOptionsMenu(menu);
    }

    private Intent getDefaultIntent() {
        Intent intent = new Intent(Intent.ACTION_SEND);
        intent.setType("image/*");
        return intent;
    }
``` 

### 但shareItem.getActionProvider()会得到空指针异常，所以修改为 

```
@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu_main, menu);
        MenuItem shareItem = menu.findItem(R.id.action_share);
        ShareActionProvider provider = (ShareActionProvider) MenuItemCompat.getActionProvider(shareItem);


        provider.setShareIntent(getDefaultIntent());
        return super.onCreateOptionsMenu(menu);
    }

    private Intent getDefaultIntent() {
        Intent intent = new Intent(Intent.ACTION_SEND);
        intent.setType("image/*");
        return intent;
    }
```

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160405194546465)


### 添加导航Tab 

Tabs的应用可以算是非常广泛了，它可以使得用户非常轻松地在你的应用程序中切换不同的视图。而Android官方更加推荐使用ActionBar中提供的Tabs功能，因为它更加的智能，可以自动适配各种屏幕的大小。比如说，在平板上屏幕的空间非常充足，Tabs会和Action按钮在同一行显示


### 主要有三步  

下面我们就来看一下如何使用ActionBar提供的Tab功能，大致可以分为以下几步：
 
1. 实现ActionBar.TabListener接口，这个接口提供了Tab事件的各种回调，比如当用户点击了一个Tab时，你就可以进行切换Tab的操作。
 
2.为每一个你想添加的Tab创建一个ActionBar.Tab的实例，并且调用setTabListener()方法来设置ActionBar.TabListener。除此之外，还需要调用setText()方法来给当前Tab设置标题。
 
3.最后调用ActionBar的addTab()方法将创建好的Tab添加到ActionBar中。


详情见[【Android UI设计与开发】8.顶部标题栏（一）ActionBar 奥义·详解 - 叶超Luka - 博客园](http://www.cnblogs.com/yc-755909659/p/4290784.html)

主要需要注意到在用Android Studio开发时，使用ActionBar actionBar = getActionBar();会得到空指针异常，所以要全部改成ActionBar actionBar = getSupportActionBar();其中ActionBar来自于import android.support.v7.app.ActionBar;

### 效果如下 

![这里写图片描述](http://img.blog.csdn.net/20160405195644048)


### 添加下拉列表导航

.1 简单介绍
 
作为Activity内部的另一种导航（或过滤）模式，操作栏提供了内置的下拉列表。下拉列表能够提供Activity中内容
 
的不同排序模式。
 
启用下拉式导航的基本过程如下：
 
<1> 创建一个给下拉提供可选项目的列表，以及描画列表项目时所使用的布局；
 
<2> 实现ActionBar.OnNavigationListener回调，在这个回调中定义当用户选择列表中一个项目时所发生的行为；
 
<3> 用setNavigationMode()方法该操作栏启用导航模式；
 
<4> 用setListNavigationCallbacks()方法给下拉列表设置回调方法。

### 实现 

准备列表数据（ strings.xml）

```
  <string-array name="action_list">
        <item>Fragment1</item>
        <item>Fragment2</item>
        <item>Fragment3</item>
    </string-array>
 ```  

主界面代码  

```
package com.yanis.actionbar;

import android.app.ActionBar;
import android.app.ActionBar.OnNavigationListener;
import android.app.Activity;
import android.app.Fragment;
import android.app.TaskStackBuilder;
import android.content.Intent;
import android.os.Bundle;
import android.support.v4.app.NavUtils;
import android.view.MenuItem;
import android.widget.ArrayAdapter;
import android.widget.SpinnerAdapter;

public class ListActivity extends Activity {
    private OnNavigationListener mOnNavigationListener;
    private String[] arry_list;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_list);

        initView();
    }

    /**
     * 初始化组件
     */
    private void initView() {
        ActionBar actionBar = getActionBar();
        actionBar.setDisplayHomeAsUpEnabled(true);
        // //导航模式必须设为NAVIGATION_MODE_LIST
        actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_LIST);

        // 定义一个下拉列表数据适配器
        SpinnerAdapter mSpinnerAdapter = ArrayAdapter.createFromResource(this,
                R.array.action_list,
                android.R.layout.simple_spinner_dropdown_item);
        arry_list = getResources().getStringArray(R.array.action_list);
        mOnNavigationListener = new OnNavigationListener() {

            @Override
            public boolean onNavigationItemSelected(int position, long itemId) {
                Fragment newFragment = null;
                switch (position) {
                case 0:
                    newFragment = new Fragment1();
                    break;
                case 1:
                    newFragment = new Fragment2();
                    break;
                case 2:
                    newFragment = new Fragment3();
                    break;
                default:
                    break;
                }
                getFragmentManager()
                        .beginTransaction()
                        .replace(R.id.container, newFragment,
                                arry_list[position]).commit();
                return true;
            }
        };
        actionBar.setListNavigationCallbacks(mSpinnerAdapter,
                mOnNavigationListener);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
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

### 主要是要将getFragmentManager改成getSupportFragmentManager应对版本问题  

### 效果如下:
![这里写图片描述](http://img.blog.csdn.net/20160405200659317)

### 参考链接:

[【Android UI设计与开发】8.顶部标题栏（一）ActionBar 奥义·详解 - 叶超Luka - 博客园](http://www.cnblogs.com/yc-755909659/p/4290784.html)

[github代码地址](https://github.com/YeXiaoChao/Yc_ui_actionbar)













