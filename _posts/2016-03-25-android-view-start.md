---
layout: post
title: Android之自定义控件入门
date: 2016-3-25
categories: blog
tags: [android]
description: 安卓。
---

### 本文主要讲述了实现安卓button点击变色与利用ViewPager实现图片自动轮播效果  

### 我们可以看到在很多应用中，安卓按钮按下时与正常时状态是不同的，这种效果也很容易达到。    

### 第一步：创建XML文件定义不同事件的不同效果    
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
          android:drawable="@drawable/function_greenbutton_pressed" /> <!-- pressed -->
          
    <item android:state_focused="true"
          android:drawable="@drawable/function_greenbutton_pressed" /> <!-- focused -->
          
  
    <item android:drawable="@drawable/function_greenbutton_normal" /> <!-- default -->
</selector>
```   


### 在上面就定义了在pressed与normal情况下，安卓的图片会自动替换的效果。    

###  第二步：在布局文件中加入定义好的按钮就可以了     
```   
  <Button  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:text="按下文字会变效果"  
        android:textColor="@drawable/btn_color"  
        android:background="@drawable/btn_bg"  
        />  
```   

### 利用ViewPager实现自动轮播图片     

```    
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="200dp" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignBottom="@id/viewpager"
        android:background="#33000000"
        android:orientation="vertical" >

        <TextView
            android:id="@+id/image_desc"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:text="@string/app_name"
            android:textColor="@android:color/white"
            android:textSize="18sp" />

        <LinearLayout
            android:id="@+id/point_group"
            android:layout_width="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_height="wrap_content" >
        </LinearLayout>
    </LinearLayout>

</RelativeLayout>
```   

### 小圆点分为点击状态与正常状态，定义如下   

```
//dot_normal.xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <solid android:color="#ffffff"/>
    <corners android:radius="8dip"/>

</shape>

//dot_focus.xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <solid android:color="#ff0000"/>
    <corners android:radius="8dip"/>

</shape>

```

### 注意viewPager引入时要用全类名，上面还定义了图片介绍与图片切换时小圆点也会切换    

### viewPager的主要方法   

```  
//设置适配器adapter
 viewPager.setAdapter(new MyPagerAdapter());
//设置当前的位置，轮播到哪一个图片了    
viewPager.setCurrentItem(Integer.MAX_VALUE/2 - (Integer.MAX_VALUE/2%imageList.size())) ;
//设置viewPager的监听事件     
viewPager.setOnPageChangeListener(new OnPageChangeListener() 
```   

### 实现  

### 适配器  

```
private class MyPagerAdapter extends PagerAdapter {

    @Override
    /**
     * 获得页面的总数
     */
    public int getCount() {
      return Integer.MAX_VALUE;
    }

    @Override
    /**
     * 获得相应位置上的view
     * container  view的容器，其实就是viewpager自身
     * position   相应的位置
     */
    public Object instantiateItem(ViewGroup container, int position) {
      
      System.out.println("instantiateItem  ::"+position);
      
      // 给 container 添加一个view
      container.addView(imageList.get(position%imageList.size()));
      //返回一个和该view相对的object
      return imageList.get(position%imageList.size());
    }

    @Override
    /**
     * 判断 view和object的对应关系 
     */
    public boolean isViewFromObject(View view, Object object) {
      if(view == object){
        return true;
      }else{
        return false;
      }
    }

    @Override
    /**
     * 销毁对应位置上的object
     */
    public void destroyItem(ViewGroup container, int position, Object object) {
      System.out.println("destroyItem  ::"+position);
      
      container.removeView((View) object);
      object = null;
    }
  }
```   

### 将当前页面总数设大一点，就可以实现无限循环了，viewpager总是只保持三个窗口，循环利用，并不会造成内存浪费，将当前项设置在中间，就可以左右自动循环了  

### 事件监听实现    
```   
viewPager.setOnPageChangeListener(new OnPageChangeListener() {
      

      @Override
      /**
       * 页面切换后调用 
       * position  新的页面位置
       */
      public void onPageSelected(int position) {
        
        position = position%imageList.size();
        
        //设置文字描述内容
        iamgeDesc.setText(imageDescriptions[position]);
        
        //改变指示点的状态
        //把当前点enbale 为true 
        pointGroup.getChildAt(position).setEnabled(true);
        //把上一个点设为false
        pointGroup.getChildAt(lastPosition).setEnabled(false);
        lastPosition = position;
        
      }
      
      @Override
      /**
       * 页面正在滑动的时候，回调
       */
      public void onPageScrolled(int position, float positionOffset,
          int positionOffsetPixels) {
      }
      
      @Override
      /**
       * 当页面状态发生变化的时候，回调
       */
      public void onPageScrollStateChanged(int state) {
        
      }
    });
```    

### 实现自动播放   
实现循环有以下几种方法  
 自动循环：    
1、定时器：Timer                
2、开子线程 while  true 循环       
3、ColckManager                       
 4、 用handler 发送延时信息，实现循环        

### 这里采用第四种     
### 在onCreate方法中  

```
     isRunning = true;
   handler.sendEmptyMessageDelayed(0, 2000);
```   

### 定义handler  
      
```    
/**
   * 判断是否自动滚动
   */
  private boolean isRunning = false;
  
  private Handler handler = new Handler(){
    public void handleMessage(android.os.Message msg) {
      
      //让viewPager 滑动到下一页
      viewPager.setCurrentItem(viewPager.getCurrentItem()+1);
      if(isRunning){
        handler.sendEmptyMessageDelayed(0, 2000);
      }
    };
  };
  
  protected void onDestroy() {
    
    isRunning = false;
  };
```       

### 另外，关于layoutParams 的一些小知识      
```            
//添加指示点         
ImageView point =new ImageView(this);
LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT);
params.rightMargin = 20;
point.setLayoutParams(params);
point.setBackgroundResource(R.drawable.point_bg);
```           

### 当元素是在LinearLayout中时，就要用LinearLayout.LayoutParams,在relativeLayout中时，就要用relativeLayout.LayoutParams        

### 实现效果         
![完成](http://img.blog.csdn.net/20160325201153660)













