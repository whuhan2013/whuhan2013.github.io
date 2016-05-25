---
layout: post
title: Android屏幕适配总结
date: 2016-5-25
categories: blog
tags: [android]
description: Android屏幕适配总结
---

### 本文主要包括以下内容  

1. 百分比方法屏幕适配 
2. Google官方Android-percent-support解决适配问题
3. 利用AndroidAutoLayout方法解决终极适配问题   



### 重要概念

什么是屏幕尺寸、屏幕分辨率、屏幕像素密度？        
什么是dp、dip、dpi、sp、px？他们之间的关系是什么？       
什么是mdpi、hdpi、xdpi、xxdpi？如何计算和区分？           

在下面的内容中我们将介绍这些概念。

### 屏幕尺寸

屏幕尺寸指屏幕的对角线的长度，单位是英寸，1英寸=2.54厘米

比如常见的屏幕尺寸有2.4、2.8、3.5、3.7、4.2、5.0、5.5、6.0等

### 屏幕分辨率

屏幕分辨率是指在横纵向上的像素点数，单位是px，1px=1个像素点。一般以纵向像素*横向像素，如1960*1080。

### 屏幕像素密度

屏幕像素密度是指每英寸上的像素点数，单位是dpi，即“dot per inch”的缩写。屏幕像素密度与屏幕尺寸和屏幕分辨率有关，在单一变化条件下，屏幕尺寸越小、分辨率越高，像素密度越大，反之越小。

### dp、dip、dpi、sp、px

px我们应该是比较熟悉的，前面的分辨率就是用的像素为单位，大多数情况下，比如UI设计、Android原生API都会以px作为统一的计量单位，像是获取屏幕宽高等。

dip和dp是一个意思，都是Density Independent Pixels的缩写，即密度无关像素，上面我们说过，dpi是屏幕像素密度，假如一英寸里面有160个像素，这个屏幕的像素密度就是160dpi，那么在这种情况下，dp和px如何换算呢？在Android中，规定以160dpi为基准，1dip=1px，如果密度是320dpi，则1dip=2px，以此类推。

假如同样都是画一条320px的线，在480*800分辨率手机上显示为2/3屏幕宽度，在320*480的手机上则占满了全屏，如果使用dp为单位，在这两种分辨率下，160dp都显示为屏幕一半的长度。这也是为什么在Android开发中，写布局的时候要尽量使用dp而不是px的原因。

而sp，即scale-independent pixels，与dp类似，但是可以根据文字大小首选项进行放缩，是设置字体大小的御用单位。

### mdpi、hdpi、xdpi、xxdpi

其实之前还有个ldpi，但是随着移动设备配置的不断升级，这个像素密度的设备已经很罕见了，所在现在适配时不需考虑。

mdpi、hdpi、xdpi、xxdpi用来修饰Android中的drawable文件夹及values文件夹，用来区分不同像素密度下的图片和dimen值。

那么如何区分呢？Google官方指定按照下列标准进行区分：

名称 像素密度范围           
mdpi  120dpi~160dpi            
hdpi  160dpi~240dpi          
xhdpi 240dpi~320dpi         
xxhdpi   320dpi~480dpi                
xxxhdpi  480dpi~640dpi                  

### 参考链接

[Android屏幕适配全攻略(最权威的官方适配指导) - 赵凯强 - 博客频道 - CSDN.NET](http://blog.csdn.net/zhaokaiqiang1992/article/details/45419023#重要概念)




### 解决方案 

### 百分比方法 

其实我们的解决方案，就是在项目中针对你所需要适配的手机屏幕的分辨率各自简历一个文件夹。

![](http://img.blog.csdn.net/20150503174449732)


然后我们根据一个基准，为基准的意思就是：

比如480*320的分辨率为基准        

- 宽度为320，将任何分辨率的宽度分为320份，取值为x1-x320
- 高度为480，将任何分辨率的高度分为480份，取值为y1-y480       
例如对于800*480的宽度480：


![](http://img.blog.csdn.net/20150503180400049)


可以看到x1 = 480 / 基准 = 480 / 320 = 1.5 ;

其他分辨率类似~~ 
你可能会问，这么多文件，难道我们要手算，然后自己编写？不要怕，下文会说。

那么，你可能有个疑问，这么写有什么好处呢？

假设我现在需要在屏幕中心有个按钮，宽度和高度为我们屏幕宽度的1/2，我可以怎么编写布局文件呢？




```
<FrameLayout >

    <Button
        android:layout_gravity="center"
        android:gravity="center"
        android:text="@string/hello_world"
        android:layout_width="@dimen/x160"
        android:layout_height="@dimen/x160"/>

</FrameLayout>
```



可以看到我们的宽度和高度定义为x160，其实就是宽度的50%； 
那么效果图：


![](http://img.blog.csdn.net/20150503180542414)


### 自动生成工具

下载地址见文末，内置了常用的分辨率，默认基准为480*320，当然对于特殊需求，通过命令行指定即可：

例如：基准 1280 * 800 ，额外支持尺寸：1152 * 735；4500 * 3200；


![](http://img.blog.csdn.net/20150503173911632)


按照

```
java -jar xx.jar width height width,height_width,height
```

使用即可  

### 下载jar包  

[下载](https://github.com/hongyangAndroid/Android_Blog_Demos/tree/master/blogcodes/src/main/java/com/zhy/blogcodes/genvalues)

### 参考链接

[Android 屏幕适配方案 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/45460089)



### Google已经添加了百分比支持库

### 使用 

当然了Android-percent-support这个库，基本可以解决上述问题，是不是有点小激动，稍等，我们先描述下这个support-lib。

这个库提供了：

两种布局供大家使用：                   
PercentRelativeLayout、PercentFrameLayout，通过名字就可以看出，这是继承自FrameLayout和RelativeLayout两个容器类；

支持的属性有：            

layout_widthPercent、layout_heightPercent、 
layout_marginPercent、layout_marginLeftPercent、 
layout_marginTopPercent、layout_marginRightPercent、 
layout_marginBottomPercent、layout_marginStartPercent、layout_marginEndPercent。

可以看到支持宽高，以及margin。

也就是说，大家只要在开发过程中使用PercentRelativeLayout、PercentFrameLayout替换FrameLayout、RelativeLayout即可。

是不是很简单，不过貌似没有LinearLayout，有人会说LinearLayout有weight属性呀。但是，weight属性只能支持一个方向呀~~哈，没事，刚好给我们一个机会去自定义一个PercentLinearLayout。


### 使用 

github上也有例子，[android-percent-support-lib-sample](https://github.com/JulienGenoud/android-percent-support-lib-sample)


首先记得在build.gradle添加：

```
 compile 'com.android.support:percent:22.2.0'
```



（一）PercentFrameLayout


```
<?xml version="1.0" encoding="utf-8"?>
<android.support.percent.PercentFrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_gravity="left|top"
        android:background="#44ff0000"
        android:text="width:30%,height:20%"
        app:layout_heightPercent="20%"
        android:gravity="center"
        app:layout_widthPercent="30%"/>

    <TextView
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_gravity="right|top"
        android:gravity="center"
        android:background="#4400ff00"
        android:text="width:70%,height:20%"
        app:layout_heightPercent="20%"
        app:layout_widthPercent="70%"/>


    <TextView
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_gravity="bottom"
        android:background="#770000ff"
        android:text="width:100%,height:10%"
        android:gravity="center"
        app:layout_heightPercent="10%"
        app:layout_widthPercent="100%"/>


</android.support.percent.PercentFrameLayout> 
```


### 效果如下 

![](http://img.blog.csdn.net/20150630142729580)


(二) PercentRelativeLayout


```
<?xml version="1.0" encoding="utf-8"?>
<android.support.percent.PercentRelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:clickable="true">

    <TextView
        android:id="@+id/row_one_item_one"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_alignParentTop="true"
        android:background="#7700ff00"
        android:text="w:70%,h:20%"
        android:gravity="center"
        app:layout_heightPercent="20%"
        app:layout_widthPercent="70%"/>

    <TextView
        android:id="@+id/row_one_item_two"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_toRightOf="@+id/row_one_item_one"
        android:background="#396190"
        android:text="w:30%,h:20%"
        app:layout_heightPercent="20%"
        android:gravity="center"
        app:layout_widthPercent="30%"/>


    <ImageView
        android:id="@+id/row_two_item_one"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:src="@drawable/tangyan"
        android:scaleType="centerCrop"
        android:layout_below="@+id/row_one_item_one"
        android:background="#d89695"
        app:layout_heightPercent="70%"/>

    <TextView
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_below="@id/row_two_item_one"
        android:background="#770000ff"
        android:gravity="center"
        android:text="width:100%,height:10%"
        app:layout_heightPercent="10%"
        app:layout_widthPercent="100%"/>


</android.support.percent.PercentRelativeLayout>
```


![](http://img.blog.csdn.net/20150630142802272)



### 源码分析与自定义详见 

[Android 百分比布局库(percent-support-lib) 解析与扩展 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/46695347)


###  Android AutoLayout全新的适配方式 堪称适配终结者


第三种适配方法

Android屏幕适配方案，直接填写设计图上的像素尺寸即可完成适配，最大限度解决适配问题。


效果图

![](https://github.com/hongyangAndroid/AndroidAutoLayout/raw/master/autolayout_08.png)



你没有看错，拿到设计稿，在布局文件里面直接填写对应的px即可，px:这里的px并非是Google不建议使用的px，在内部会进行转化处理。


引入

Android Studio
将autolayout引入

```
dependencies {
    compile project(':autolayout')
}
```

也可以直接

```
dependencies {
    compile 'com.zhy:autolayout:1.4.3'
}
```

Eclipse
建议使用As，方便版本更新。实在不行，只有复制粘贴源码了。

### 用法

第一步：

在你的项目的AndroidManifest中注明你的设计稿的尺寸。

```
<meta-data android:name="design_width" android:value="768">
</meta-data>
<meta-data android:name="design_height" android:value="1280">
</meta-data>
```

第二步：

让你的Activity继承自AutoLayoutActivity.

非常简单的两个步骤，你就可以开始愉快的编写布局了，详细可以参考sample。

其他用法

如果你不希望继承AutoLayoutActivity，可以在编写布局文件时，将

LinearLayout -> AutoLinearLayout       
RelativeLayout -> AutoRelativeLayout            
FrameLayout -> AutoFrameLayout             
这样也可以完成适配。             

目前支持属性

layout_width          
layout_height                    
layout_margin(left,top,right,bottom)               
pading(left,top,right,bottom)                
textSize            
maxWidth, minWidth, maxHeight, minHeight         

配置

默认使用的高度是设备的可用高度，也就是不包括状态栏和底部的操作栏的，如果你希望拿设备的物理高度进行百分比化：

可以在Application的onCreate方法中进行设置:

```
public class UseDeviceSizeApplication extends Application
{
    @Override
    public void onCreate()
    {
        super.onCreate();
        AutoLayoutConifg.getInstance().useDeviceSize();
    }
}
```

预览

大家都知道，写布局文件的时候，不能实时的去预览效果，那么体验真的是非常的不好，也在很大程度上降低开发效率，所以下面教大家如何用好，用对PreView（针对该库）。

首先，你要记得你设计稿的尺寸，比如 768 * 1280

然后在你的PreView面板，选择于设计图分辨率一致的设备：


### 扩展

对于其他继承系统的FrameLayout、LinearLayout、RelativeLayout的控件，比如CardView，如果希望再其内部直接支持"px"百分比化，可以自己扩展，扩展方式为下面的代码，也可参考issue#21：

```
package com.zhy.sample.view;

import android.content.Context;
import android.support.v7.widget.CardView;
import android.util.AttributeSet;

import com.zhy.autolayout.AutoFrameLayout;
import com.zhy.autolayout.utils.AutoLayoutHelper;

/**
 * Created by zhy on 15/12/8.
 */
public class AutoCardView extends CardView
{
    private final AutoLayoutHelper mHelper = new AutoLayoutHelper(this);

    public AutoCardView(Context context)
    {
        super(context);
    }

    public AutoCardView(Context context, AttributeSet attrs)
    {
        super(context, attrs);
    }

    public AutoCardView(Context context, AttributeSet attrs, int defStyleAttr)
    {
        super(context, attrs, defStyleAttr);
    }

    @Override
    public AutoFrameLayout.LayoutParams generateLayoutParams(AttributeSet attrs)
    {
        return new AutoFrameLayout.LayoutParams(getContext(), attrs);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
    {
        if (!isInEditMode())
        {
            mHelper.adjustChildren();
        }
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }


}
```



### 注意事项

- ListView、RecyclerView类的Item的适配

sample中包含ListView、RecyclerView例子，具体查看sample

对于ListView            

对于ListView这类控件的item，默认根局部写“px”进行适配是无效的，因为外层非AutoXXXLayout，而是ListView。但是，不用怕，一行代码就可以支持了：

```
@Override
public View getView(int position, View convertView, ViewGroup parent)
{
    ViewHolder holder = null;
    if (convertView == null)
    {
        holder = new ViewHolder();
        convertView = LayoutInflater.from(mContext).inflate(R.layout.list_item, parent, false);
        convertView.setTag(holder);
        //对于listview，注意添加这一行，即可在item上使用高度
        AutoUtils.autoSize(convertView);
    } else
    {
        holder = (ViewHolder) convertView.getTag();
    }

    return convertView;
}
```

注意AutoUtils.autoSize(convertView);这行代码的位置即可。demo中也有相关实例。

对于RecyclerView

```
public ViewHolder(View itemView)
{
      super(itemView);
      AutoUtils.autoSize(itemView);
}

//...
@Override
public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)
{
     View convertView = LayoutInflater.from(mContext).inflate(R.layout.recyclerview_item, parent, false);
     return new ViewHolder(convertView);
}
```

一定要记得LayoutInflater.from(mContext).inflate使用三个参数的方法！

### 指定设置的值参考宽度或者高度

由于该库的特点，布局文件中宽高上的1px是不相等的，于是如果需要宽高保持一致的情况，布局中使用属性：

app:layout_auto_basewidth="height"，代表height上编写的像素值参考宽度。

app:layout_auto_baseheight="width"，代表width上编写的像素值参考高度。

如果需要指定多个值参考宽度即：

app:layout_auto_basewidth="height|padding"

用|隔开，类似gravity的用法，取值为：

```
width,height
margin,marginLeft,marginTop,marginRight,marginBottom
padding,paddingLeft,paddingTop,paddingRight,paddingBottom
textSize.
```

### TextView的高度问题

设计稿一般只会标识一个字体的大小，比如你设置textSize="20px"，实际上TextView所占据的高度肯定大于20px，字的上下都会有一定的间隙，所以一定要灵活去写字体的高度，比如对于text上下的margin可以选择尽可能小一点。或者选择别的约束条件去定位（比如上例，选择了marginBottom）


### 参考链接

[hongyangAndroid/AndroidAutoLayout: Android屏幕适配方案，直接填写设计图上的像素尺寸即可完成适配，最大限度解决适配问题](https://github.com/hongyangAndroid/AndroidAutoLayout#注意事项)


### 完成


