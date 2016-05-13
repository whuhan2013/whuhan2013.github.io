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


### 实例二  

实现一个比较简单的Google彩虹进度条

![](http://upload-images.jianshu.io/upload_images/188580-10e6e15466f2d2f6.png?imageView2/2/w/1240/q/100)

整个View的绘制流程我们已经介绍完了，还有一个很重要的知识，自定义控件属性，我们都知道View已经有一些基本的属性，比如layout_width，layout_height，background等，我们往往需要定义自己的属性，那么具体可以这么做。

1.在values文件夹下，打开attrs.xml，其实这个文件名称可以是任意的，写在这里更规范一点，表示里面放的全是view的属性。
2.因为我们下面的实例会用到2个长度，一个颜色值的属性，所以我们这里先创建3个属性。

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- 声明属性集的名称 -->
    <declare-styleable name="rainbowbar">
  <attr name="rainbowbar_hspace" format="dimension"></attr>
  <attr name="rainbowbar_vspace" format="dimension"></attr>
  <attr name="rainbowbar_color" format="color"></attr>
</declare-styleable>
    
</resources>
```

### 自定义View

```
package com.zj.secondattr;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.view.View;

public class RainbowBar extends View {
  
  //progress bar color
    int barColor = Color.parseColor("#1E88E5");
    //every bar segment width
    int hSpace = Utils.dpToPx(80, getResources());
    //every bar segment height
    int vSpace = Utils.dpToPx(4, getResources());
    //space among bars
    int space = Utils.dpToPx(10, getResources());
    float startX = 0;
    float delta = 10f;
    Paint mPaint;

    public RainbowBar(Context context) {
      super(context);
    }

    public RainbowBar(Context context, AttributeSet attrs) {
      this(context, attrs, 0);
    }

    public RainbowBar(Context context, AttributeSet attrs, int defStyleAttr) {
      super(context, attrs, defStyleAttr);
      //read custom attrs
      TypedArray t = context.obtainStyledAttributes(attrs,
              R.styleable.rainbowbar, 0, 0);
      hSpace = t.getDimensionPixelSize(R.styleable.rainbowbar_rainbowbar_hspace, hSpace);
      vSpace = t.getDimensionPixelOffset(R.styleable.rainbowbar_rainbowbar_vspace, vSpace);
      barColor = t.getColor(R.styleable.rainbowbar_rainbowbar_color, barColor);
      t.recycle();   // we should always recycle after used
      mPaint = new Paint();
      mPaint.setAntiAlias(true);
      mPaint.setColor(barColor);
      mPaint.setStrokeWidth(vSpace);
    }
    
    int index = 0;
    
    @Override
  protected void onDraw(Canvas canvas) {
    // TODO Auto-generated method stub
    super.onDraw(canvas);
    
    float sw = this.getMeasuredWidth();
      if (startX >= sw + (hSpace + space) - (sw % (hSpace + space))) {
          startX = 0;
      } else {
          startX += delta;
      }
      float start = startX;
      // draw latter parse
      while (start < sw) {
          canvas.drawLine(start, 5, start + hSpace, 5, mPaint);
          start += (hSpace + space);
      }

      start = startX - space - hSpace;

      // draw front parse
      while (start >= -hSpace) {
          canvas.drawLine(start, 5, start + hSpace, 5, mPaint);
          start -= (hSpace + space);
      }
      if (index >= 700000) {
          index = 0;
      }
      invalidate();
  }


}
```

View有了三个构造方法需要我们重写，这里介绍下三个方法会被调用的场景，

- 第一个方法，一般我们这样使用时会被调用，View view = new View(context); 

- 第二个方法，当我们在xml布局文件中使用View时，会在inflate布局时被调用，
<View layout_width="match_parent" layout_height="match_parent"/>。

- 第三个方法，跟第二种类似，但是增加style属性设置，这时inflater布局时会调用第三个构造方法。
<View style="@styles/MyCustomStyle" layout_width="match_parent" layout_height="match_parent"/>。


上面大家可能会感觉到有点困惑的是，我把初始化读取自定义属性hspace，vspace，和barcolor的代码写在第三个构造方法里面，但是我RainbowBar在线性布局中没有加style属性（），那按照我们上面的解释，inflate布局时应该会invoke第二个构造方法啊，但是我们在第二个构造方法里面调用了第三个构造方法，this(context, attrs, 0); 所以在第三个构造方法中读取自定义属性，没有问题，这是一点小细节，避免代码冗余-，-


### 布局文件中的使用 

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="${relativePackage}.${activityClass}" >

    <com.zj.secondattr.RainbowBar
     android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:rainbowbar_color="@color/abc_search_url_text_holo"
    app:rainbowbar_hspace="80dp"
    app:rainbowbar_vspace="10dp"
  
        >
   
    </com.zj.secondattr.RainbowBar>

</RelativeLayout>
```

### 效果如下  

![](http://upload-images.jianshu.io/upload_images/188580-c1fe83e12ed75041.gif?imageView2/2/w/1240/q/100)























