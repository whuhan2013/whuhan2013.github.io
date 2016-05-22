---
layout: post
title: Android之Tab类总结
date: 2016-5-22
categories: blog
tags: [android]
description: Android之Tab类总结
---   
### 本文主要包括以下Tab类实现方式 

1. FragmentTabHost+Fragment实现
2. 传统的ViewPager实现           
3. FragmentManager+Fragment实现
4.  ViewPager+FragmentPagerAdapter实现
5. TabPageIndicator+ViewPager+FragmentPagerAdapter



### FragmentTabHost+Fragment实现

### 布局文件 

```
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="vertical">  
      
    <android.support.v4.app.FragmentTabHost  
        android:id="@android:id/tabhost"  
        android:layout_width="match_parent"  
        android:layout_height="wrap_content">  
        
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:gravity="center"
            android:orientation="vertical" >

            <FrameLayout
                android:id="@android:id/tabcontent"
                android:layout_width="match_parent"
                android:layout_height="0dp"
                android:layout_weight="1" >
            </FrameLayout>

            <TabWidget
                android:id="@android:id/tabs"
                android:layout_width="match_parent"
                android:layout_height="60dip" />
        </LinearLayout> 
    </android.support.v4.app.FragmentTabHost>  
      
</LinearLayout> 
```


### MainActivity

```
package com.darna.wmxfx;

import android.os.Bundle;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentTabHost;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.ImageView;
import android.widget.RelativeLayout;
import android.widget.TabHost.OnTabChangeListener;
import android.widget.TextView;

import com.darna.wmxfx.fragment.Frg_Cart;
import com.darna.wmxfx.fragment.Frg_Index;
import com.darna.wmxfx.fragment.Frg_UserCenter;


public class MainActivity extends FragmentActivity {

	private FragmentTabHost mTabHost = null;
	private int iTabIndex = 0;
	private int[][] iTab = new int[][] { { R.drawable.indexunused, R.drawable.cartunused, R.drawable.usercenterunused },
			{ R.drawable.indexused, R.drawable.cartused, R.drawable.usercenterused } };
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initViews();
    }
    
    private void initViews(){
    	String[] strTab = getResources().getStringArray(R.array.bottom_tab);
    	RelativeLayout[] relativeLayout = new RelativeLayout[strTab.length];
    	for(int i = 0; i < strTab.length; i++){
    		if (i == 0) {
    			relativeLayout[i] = (RelativeLayout) LayoutInflater.from(this).inflate(R.layout.tab_bottom, null);
    			relativeLayout[i].findViewById(R.id.iv_bottom_tab).setBackgroundResource(iTab[1][i]);
    			TextView tView =  (TextView) relativeLayout[i].findViewById(R.id.tv_bottom_tab);
    			tView.setText(strTab[i]);
    			tView.setTextColor(getResources().getColor(R.color.textused));
			}else {
				relativeLayout[i] = (RelativeLayout) LayoutInflater.from(this).inflate(R.layout.tab_bottom, null);
    			relativeLayout[i].findViewById(R.id.iv_bottom_tab).setBackgroundResource(iTab[0][i]);
    			TextView t =  (TextView) relativeLayout[i].findViewById(R.id.tv_bottom_tab);
    			t.setText(strTab[i]);
    			t.setTextColor(getResources().getColor(R.color.textunused));
			}
    	}
    	
    	mTabHost = (FragmentTabHost) findViewById(android.R.id.tabhost);  
        mTabHost.setup(this, getSupportFragmentManager(), android.R.id.tabcontent);
    	
        mTabHost.addTab(mTabHost.newTabSpec(Config.KEY_INDEX)  
                .setIndicator(relativeLayout[0]), Frg_Index.class, null); 
        
        mTabHost.addTab(mTabHost.newTabSpec(Config.KEY_CART)  
                .setIndicator(relativeLayout[1]), Frg_Cart.class, null);
        
        mTabHost.addTab(mTabHost.newTabSpec(Config.KEY_USERCENTER)  
                .setIndicator(relativeLayout[2]), Frg_UserCenter.class, null);
        
        mTabHost.setOnTabChangedListener(new OnTabChangeListener() {

			@Override
			public void onTabChanged(String tabId) {
				// TODO Auto-generated method stub
				View v = mTabHost.getTabWidget().getChildAt(iTabIndex);
				((ImageView) v.findViewById(R.id.iv_bottom_tab)).setBackgroundResource(iTab[0][iTabIndex]);
				((TextView) v.findViewById(R.id.tv_bottom_tab)).setTextColor(getResources().getColor(R.color.textunused));
				v = mTabHost.getCurrentTabView();
				iTabIndex = mTabHost.getCurrentTab();
				((ImageView) v.findViewById(R.id.iv_bottom_tab)).setBackgroundResource(iTab[1][iTabIndex]);
				((TextView) v.findViewById(R.id.tv_bottom_tab)).setTextColor(getResources().getColor(R.color.textused));
			}
		});
		
    }
}
```



### 效果如下  


![这里写图片描述](http://img.blog.csdn.net/20160522131834126)

### 传统的ViewPager实现

![](http://img.blog.csdn.net/20140429222133796)


### FragmentManager+Fragment实现

![](http://img.blog.csdn.net/20140429222032734)

### ViewPager+Fragment实现


![](http://img.blog.csdn.net/20140429221600875)


### TabPageIndicator+ViewPager+FragmentPagerAdapter

实现方式和3是一致的，但是使用了TabPageIndicator作为tab的指示器，效果还是不错的，这个之前写过，就不再贴代码了。
效果图：

![](http://img.blog.csdn.net/20140412094426906)

### 后四种方式详细可以参见  

[Android项目Tab类型主界面大总结 Fragment+TabPageIndicator+ViewPager - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/24740977)


### 完成






























