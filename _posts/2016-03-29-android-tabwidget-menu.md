---
layout: post
title: 利用TabWidget实现底部菜单
date: 2016-3-29
categories: blog
tags: [android]
description: 利用TabWidget实现底部菜单
---
### TabWidget类似于通话记录的界面，通过切换多个标签从而显示出多个不同内容，能够展示内容丰富的页面信息，而且彼此之间不会干扰，有利于展示。下面，通过一个例子来学习用法  

### 首先用一个类来继承TabActivity
 在开发之前，我们要首先了解，TabHost是整个Tab的容器，包括两部分，TabWidget和FrameLayout。
TabWidget就是每个tab的标签，FrameLayout则是tab内容。
接着我们开始初始化main.xml
首先声明TabHost，包含TabWidget，FrameLayout元素。

```    
    <TabHost 
    android:id="@android:id/tabhost"  //声明控件ID
    android:layout_width="fill_parent"    //控件宽度与父控件一致
     android:layout_height="fill_parent">  //控件高度与父控件一致
声明TabWidget，tab标签页
            <TabWidget 
            android:layout_width="fill_parent"      //控件宽度与父控件一致
            android:layout_height="wrap_content"   //控件高度与自身适应
            android:id="@android:id/tabs">    //声明控件ID
声明FrameLayout，tab页里的内容信息
            <FrameLayout 
            android:layout_width="fill_parent"   //控件宽度与父控件一致
            android:layout_height="wrap_content"  //控件高度与自身适应
            android:id="@android:id/tabcontent">   //声明控件ID
```   

 
注意下：
如果我们使用extends TabAcitivty，如同ListActivity，TabHost必须设置为@android:id/tabhost  
TabWidget必须设置android:id为@android:id/tabs        
FrameLayout需要设置android:id为@android:id/tabcontent  
如果不设置正确就会报错，笔者就在这里出错了


#### 布局文件 

```
<?xml version="1.0" encoding="utf-8"?>
<TabHost xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@android:id/tabhost" >
    
    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:orientation="vertical" >
        <FrameLayout android:id="@android:id/tabcontent" 
            android:layout_width="fill_parent"
            android:layout_height="0.0dip"
            android:layout_weight="1.0"/>
        <TabWidget android:id="@android:id/tabs" 
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:visibility="gone"
             />
           <RadioGroup
             android:id="@+id/tab_items"
             android:gravity="center_vertical"
             android:layout_width="fill_parent"
             android:layout_height="wrap_content"
             android:orientation="horizontal"
             android:layout_gravity="bottom"
             android:background="@drawable/tab_bg"
             >
                <RadioButton
                   android:id="@+id/tab_item_home"
                   android:checked="true"                     
                   style="@style/main_tab_bottom"
                   android:background="@drawable/item_home_bg" />
                <RadioButton
                   android:id="@+id/tab_item_nearby"  
                   style="@style/main_tab_bottom"
                   android:background="@drawable/item_near_bg"
                    />
                <RadioButton
                   android:id="@+id/tab_item_sort"   
                   style="@style/main_tab_bottom"
                   android:background="@drawable/item_sort_bg" />
                <RadioButton
                   android:id="@+id/tab_item_mine"  
                   style="@style/main_tab_bottom"
                   android:background="@drawable/item_mine_bg"/>                
                <RadioButton
                   android:id="@+id/tab_item_more"  
                   style="@style/main_tab_bottom" 
                   android:background="@drawable/item_more_bg" />
           </RadioGroup>
         
    </LinearLayout>
    

</TabHost>
```

### 其中有些控件的图片点击与正常情况下是不同的,如item_home_bg.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android" >
    
    <item android:state_checked="true" android:drawable="@drawable/but_index_r_v2" />
    <item android:drawable="@drawable/but_index_v2"/>
</selector>
```  

### style文件在values文件夹下的styles.xml文件中定义

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="main_tab_bottom">
        <item name="android:gravity">center_horizontal</item>
        <item name="android:layout_width">fill_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:button">@null</item>
        
        
        <item name="android:layout_weight">1.0</item>
    </style>
   </resources>  
```   

### 函数实现  

```
public class MyTab extends TabActivity{
  
  private final static String TAG = "TabShow";
  private TabHost mHost;
  private RadioGroup tabItems;
  private RadioButton mineBut;
  
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    // TODO Auto-generated method stub
    super.onCreate(savedInstanceState);
    
    setContentView(R.layout.tablay);
    
    
    initResourceRefs();
      initSettings();
  }

  private void initSettings() {
    // TODO Auto-generated method stub
    
    tabItems.setOnCheckedChangeListener(new OnCheckedChangeListener() {
      
      @Override
      public void onCheckedChanged(RadioGroup group, int checkedId) {
        // TODO Auto-generated method stub
        switch(checkedId){
        
         case R.id.tab_item_home :
           mHost.setCurrentTabByTag("HOME");
           break;
         case R.id.tab_item_nearby :
           mHost.setCurrentTabByTag("NEAR");
           break;
         case R.id.tab_item_sort :
           mHost.setCurrentTabByTag("SORT");
           break;     
         case R.id.tab_item_more :
           mHost.setCurrentTabByTag("MORE");
           break;
        
        }
      }
    });
    
  }

  private void initResourceRefs() {
    // TODO Auto-generated method stub
    
     mHost = getTabHost();
      
      mHost.addTab(mHost.newTabSpec("HOME").setIndicator("HOME")
            .setContent(new Intent(this , HomeActivity.class)));
        
      mHost.addTab(mHost.newTabSpec("NEAR").setIndicator("NEAR")
            .setContent(new Intent(this , NearByActivity.class)));
        
      mHost.addTab(mHost.newTabSpec("SORT").setIndicator("SORT")
            .setContent(new Intent(this , SortActivity.class)));
      mHost.addTab(mHost.newTabSpec("My").setIndicator("My")
            .setContent(new Intent(this , MyActivity.class)));
          
      mHost.addTab(mHost.newTabSpec("MORE").setIndicator("MORE")
            .setContent(new Intent(this , MoreActivity.class)));
       
      tabItems = (RadioGroup)findViewById(R.id.tab_items);
      mineBut = (RadioButton)findViewById(R.id.tab_item_mine);
    
  }

}
```   

### 效果如下 
![如下](http://img.blog.csdn.net/20160329192430525)






