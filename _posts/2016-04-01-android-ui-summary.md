---
layout: post
title: Android之UI控件
date: 2016-4-1
categories: blog
tags: [android]
description: Android之UI控件
---
### 本文主要包括以下内容  

1. Spinner的使用  
2.  Gallery的使用  


### Spinner的使用    

Spinner的实现过程是    
1. 在xml文件中定义Spinner的控件              
2. 在activity中获取Spinner控件
3. 定义Spinner下拉列表项数组并将下拉项的内容添加到这个数组中，通过这个数组建立一个下拉列表的适配器                       
4. 将上3中的适配器配置给获取的Spinner控件                
5. 设置下拉列表的显示样式                  
6. 为获得的Spinner控件添加事件监听                      

### 在XML文件中定义  

```
//在主XML中<include android:id="@+id/sp_chose" layout="@layout/spinner_down"/>

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
         android:layout_height="30dip"
         android:orientation="horizontal"
         android:background="@drawable/filter_bg"
         android:layout_marginTop="5dip"
         android:layout_marginLeft="5dip"
         android:layout_marginRight="5dip">

      <Spinner
           android:id="@+id/nearby_distance_spinner"
           style="@style/nearby_spinner_style" />
      <Spinner
           android:id="@+id/nearby_class_spinner"
           style="@style/nearby_spinner_style" />
      <Spinner
           android:id="@+id/nearby_away_spinner"
           style="@style/nearby_spinner_style" />
   

</LinearLayout>
```   

### 其中背景图片为
![这里写图片描述](http://img.blog.csdn.net/20160401155845786)

### nearby_spinner_style为  

```
<style name="nearby_spinner_style">
        <item name="android:layout_width">0.0dip</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:background">@null</item>
        <item name="android:layout_marginTop">6dip</item>
        <item name="android:layout_weight">1.0</item>
    </style>
```   

### 找到Spinner并初始化适配器  

```
private void init() {
    // TODO Auto-generated method stub
    topText=(TextView) findViewById(R.id.tv_chose_shop);
    topText.setText(getIntent().getStringExtra("type"));
    disSpi=(Spinner) findViewById(R.id.nearby_distance_spinner);
    claSpi=(Spinner) findViewById(R.id.nearby_class_spinner);
    awaySpi=(Spinner) findViewById(R.id.nearby_away_spinner);
    
     disAdapter=new ArrayAdapter<String>(this, R.layout.nearby_spinner_text, DIS_DATE);
     claAdapter=new ArrayAdapter<String>(this, R.layout.nearby_spinner_text, CLASS_DATE);
     awayAdapter=new ArrayAdapter<String>(this, R.layout.nearby_spinner_text, AWAY_DATE);
    
    
    
  }
```  

### 其中nearby_spinner_text为 

```
<?xml version="1.0" encoding="utf-8"?>

 <TextView xmlns:android="http://schemas.android.com/apk/res/android"
     android:id="@android:id/text1"
     style="?android:attr/spinnerDropDownItemStyle"
     android:singleLine="true"
     android:layout_width="fill_parent"
     android:layout_height="wrap_content"
     android:gravity="center_vertical"
     android:textColor="#ffffff"
     android:textSize="12sp"/>
```    

### 设置下拉列表的显示样式并且将适配器配置给spinner

```
//设置列表项显示风格为完全显示
  disAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
    claAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
    awayAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
    
    
    disSpi.setAdapter(disAdapter);
    claSpi.setAdapter(claAdapter);
    awaySpi.setAdapter(awayAdapter);
    
    disSpi.setSelection(2);
    claSpi.setSelection(0);
    awaySpi.setSelection(0);
```   

### 设置监听事件  

```
disSpi.setOnItemSelectedListener(new OnItemSelectedListener() {

      @Override
      public void onItemSelected(AdapterView<?> parent, View view,
          int position, long id) {
        // TODO Auto-generated method stub
        Toast.makeText(getApplicationContext(), DIS_DATE[position], 0).show();
      }

      @Override
      public void onNothingSelected(AdapterView<?> parent) {
        // TODO Auto-generated method stub
        
      }
      
    });
```  

### 完成，效果如下  
![这里写图片描述](http://img.blog.csdn.net/20160401160908165)
![这里写图片描述](http://img.blog.csdn.net/20160401160924305)











