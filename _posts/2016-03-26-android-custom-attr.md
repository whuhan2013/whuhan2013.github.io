---
layout: post
title: Android之自定义属性
date: 2016-3-26
categories: blog
tags: [android]
description: 安卓。
---

### 安卓自定义属性主要有3个步骤    

### 1.在values文件夹新建attrs.xml文件中声明属性，包括属性名和格式，format常用属性有string ,integer,reference等     

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- 声明属性集的名称 -->
    <declare-styleable name="MyToggleButtton">
        <!-- 声明属性的name与类型 -->
        <attr name="my_background" format="reference"/>
        <attr name="my_slide_btn" format="reference"/>
        <attr name="curr_state" format="boolean"/>
    </declare-styleable>
    
</resources>
```     

### 2.在布局文件中使用，使用之前必须先声明命名空间，前面是固定不变的内容，后面是包名.       

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:zj="http://schemas.android.com/apk/res/com.zj.switchbutton"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="${relativePackage}.${activityClass}" >

    <com.zj.switchbutton.MyTrouggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        zj:my_background="@drawable/switch_background"
        zj:my_slide_btn="@drawable/slide_button"
        zj:curr_state="true"
        />

</RelativeLayout>
```     

### 3.在自定义view的构造方法中，通过解析AttributeSet方法，获得所需要的属性值,解析AttributeSet主要有两种方法  

### 第一种:通过attrs.getAttributeValue获得    

```
int counts=attrs.getAttributeCount();
    for(int i=0;i<counts;i++)
    {
      attrs.getAttributeName(i);
      attrs.getAttributeValue(i);
    }


public SettingItemView(Context context, AttributeSet attrs) {
    super(context, attrs);
    // TODO Auto-generated constructor stub
    iniView(context);
    String title = attrs.getAttributeValue("http://schemas.android.com/apk/res/com.zj.mobilesafe", "mytitle");
    desc_on = attrs.getAttributeValue("http://schemas.android.com/apk/res/com.zj.mobilesafe", "desc_on");
    desc_off = attrs.getAttributeValue("http://schemas.android.com/apk/res/com.zj.mobilesafe", "desc_off");
    
    tv_title.setText(title);
    setDesc(desc_off);
    
  }
```    

### 第二种:通过TypedArray获得  

```
public MyTrouggleButton(Context context, AttributeSet attrs) {
    super(context, attrs);
    // TODO Auto-generated constructor stub
    //获得自定义属性
    TypedArray ta=context.obtainStyledAttributes(attrs,R.styleable.MyToggleButtton);
    int N=ta.getIndexCount();
    for(int i=0;i<N;i++)
    {
      int itemId=ta.getIndex(i);
      switch (itemId) {
      case R.styleable.MyToggleButtton_curr_state:
        current_state=ta.getBoolean(itemId, false);
        
        break;
      case R.styleable.MyToggleButtton_my_background:
        backgroundID=ta.getResourceId(itemId, -1);
        if(backgroundID==-1)
        {
          throw new RuntimeException("请设置背景图片");
        }
        backgroundBitmap=BitmapFactory.decodeResource(getResources(),backgroundID);
        break;
      case R.styleable.MyToggleButtton_my_slide_btn:
        slideButtonID=ta.getResourceId(itemId, -1);
        if(backgroundID==-1)
        {
          throw new RuntimeException("请设置图片");
        }
        slideBtnBitmap=BitmapFactory.decodeResource(getResources(), slideButtonID);

      default:
        break;
      }
    }
    init();
  }

```    

### 自定义属性到底有什么用呢？当界面上的自定义元素有一些值需要改变并且大量重复的时候，自定义属性可以有效的提高代码的重用性，下面是一个简单的例子

### 声明属性  

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
     <declare-styleable name="TextView">
        <attr name="mytitle" format="string" />
        <attr name="desc_on" format="string" />
        <attr name="desc_off" format="string" />
    </declare-styleable>
</resources>
```  

### 在xml文件中定义

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:zj="http://schemas.android.com/apk/res/com.zj.mobilesafe"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <TextView
        android:id="@+id/textView1"
        android:layout_width="fill_parent"
        android:layout_height="55dip"
        android:background="#8866ff00"
        android:gravity="center"
        android:text="设置中心"
        android:textColor="#000000"
        android:textSize="22sp" />

    <com.zj.mobilesafe.ui.SettingItemView
        android:id="@+id/siv_update"
        android:layout_width="wrap_content"
        android:layout_height="65dip"
        zj:desc_off="设置自动更新已经关闭"
        zj:desc_on="设置自动更新已经开启"
        zj:mytitle="设置自动更新" >
    </com.zj.mobilesafe.ui.SettingItemView>
    
     <com.zj.mobilesafe.ui.SettingItemView
        android:id="@+id/siv_show_address"
        android:layout_width="wrap_content"
        android:layout_height="65dip"
        zj:desc_off="设置显示号码归属地已经关闭"
        zj:desc_on="设置显示号码归属地已经开启"
        zj:mytitle="设置显示号码归属地" >
    </com.zj.mobilesafe.ui.SettingItemView>
    
     <com.zj.mobilesafe.ui.SettingClickView
         android:id="@+id/scv_changebg"
         android:layout_width="wrap_content"
        android:layout_height="65dip"
         >
         
     </com.zj.mobilesafe.ui.SettingClickView>
     
     <com.zj.mobilesafe.ui.SettingItemView
         android:id="@+id/siv_callsms_safe"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        zj:desc_off="黑名单拦截已经关闭"
        zj:desc_on="黑名单拦截已经开启"
        zj:mytitle="黑名单拦截设置" >
    </com.zj.mobilesafe.ui.SettingItemView>
    
     <com.zj.mobilesafe.ui.SettingItemView
        android:id="@+id/siv_watchdog"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        zj:desc_off="看门狗已经关闭"
        zj:desc_on="看门狗已经开启"
        zj:mytitle="程序锁设置" >
    </com.zj.mobilesafe.ui.SettingItemView>
     

</LinearLayout>
```  

### 解析属性并且改变属性  

```
/**
 * 自定义的组合控件
 * @author Administrator
 *
 */
public class SettingItemView extends RelativeLayout {
  
  private CheckBox cb_status;
  private TextView tv_desc;
  private TextView tv_title;
  private  String desc_on;
  private String desc_off;
  /**
   * 初始化布局文件
   * @param context
   */
  private void iniView(Context context) {
    // TODO Auto-generated method stub
    View.inflate(context, R.layout.setting_item_view, SettingItemView.this);
    cb_status=(CheckBox) this.findViewById(R.id.cb_status);
    tv_desc=(TextView) this.findViewById(R.id.tv_desc);
    tv_title=(TextView) this.findViewById(R.id.tv_title);
  }

  public SettingItemView(Context context, AttributeSet attrs, int defStyle) {
    super(context, attrs, defStyle);
    // TODO Auto-generated constructor stub
    
    iniView(context);
  }

  /**
   * 带有两个参数的构造方法,布局文件使用的时候调用 
   * @param context
   * @param attrs
   */

  public SettingItemView(Context context, AttributeSet attrs) {
    super(context, attrs);
    // TODO Auto-generated constructor stub
    iniView(context);
    String title = attrs.getAttributeValue("http://schemas.android.com/apk/res/com.zj.mobilesafe", "mytitle");
    desc_on = attrs.getAttributeValue("http://schemas.android.com/apk/res/com.zj.mobilesafe", "desc_on");
    desc_off = attrs.getAttributeValue("http://schemas.android.com/apk/res/com.zj.mobilesafe", "desc_off");
    
    tv_title.setText(title);
    setDesc(desc_off);
    
  }

  public SettingItemView(Context context) {
    super(context);
    // TODO Auto-generated constructor stub
    
    iniView(context);
  }
  
  /**
   * 
   * 检验组合和控件是否有焦点
   */
  
  public boolean isChecked()
  {
    return cb_status.isChecked();
  }
  
  /**
   * 设置组合控件的是否选中
   */
  
  public void setChecked(boolean checked)
  {
    if(checked)
    {
      setDesc(desc_on);
    }else
    {
      setDesc(desc_off);
    }
    
    cb_status.setChecked(checked);
  }
  
  /**
   * 组合控件 的内容发生改变
   * 
   */
  
  public void setDesc(String text)
  {
    tv_desc.setText(text);
  }
  
  

}
```

### 效果如下   
![图片效果](http://img.blog.csdn.net/20160326203327699)












