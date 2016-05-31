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
6. Android底部导航栏开源库的使用    
7. ViewPager+Fragment+FragmentTabHost实现底部菜单



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


### Android底部导航栏开源库的使用

传送门：[IconTabPageIndicator](https://github.com/msdx/IconTabPageIndicator)


先按照它README.md说明的配置好，接下来就要按照自己的需要修改。
布局文件按照demo里来：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"    android:layout_width="match_parent"    android:layout_height="match_parent" >    
<com.githang.viewpagerindicator.IconTabPageIndicator   
     android:id="@+id/indicator"        
     android:layout_alignParentBottom="true"           
     android:layout_width="match_parent"      
     android:layout_height="wrap_content"/>    
<View android:layout_width="match_parent"        
      android:id="@+id/divider"      
      android:layout_above="@id/indicator"       
      android:layout_height="1px"       
      android:background="#ababab"/>    
<android.support.v4.view.ViewPager        
    android:layout_above="@id/divider"        
    android:id="@+id/view_pager"        
    android:layout_width="match_parent"        
    android:layout_height="match_parent"/>
</RelativeLayout>
```

Activity和BaseFragment也直接用demo中的即可，在这里就不放出来了。
需要更改的地方主要在fragment上。我根据需要新建fg1.java、fg2.java、fg3.java、fg4.java,继承BaseFragment。

```
public class Fg1 extends BaseFragment {    
@Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fg1,container,false);  
      return view; 
   }
}
```

在MainActivity中对initFragments()进行修改，将fragment修改成自己创建的。控件中导航栏的名称直接在userFragment.setTitle()修改即可。

```
private List<BaseFragment> initFragments() {   
 List<BaseFragment> fragments = new ArrayList<BaseFragment>();    
BaseFragment userFragment = new Fg1();    
userFragment.setTitle("主页");    
userFragment.setIconId(R.drawable.tab_index_selector);    
fragments.add(userFragment);    
BaseFragment noteFragment = new Fg2();    
noteFragment.setTitle("我的商品");    
noteFragment.setIconId(R.drawable.tab_record_selector);    
fragments.add(noteFragment);    
......
   }
```

图标通过userFragment.setIconId()控制。在drawable中新建selector.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">  
  <item android:drawable="@drawable/icon_index_pressed" android:state_selected="true" />   
 <item android:drawable="@drawable/icon_index_pressed" android:state_pressed="true" />   
 <item android:drawable="@drawable/icon_index_pressed" android:state_checked="true" />    
<item android:drawable="@drawable/icon_index_normal" />
</selector>
```

选中、点击、check都使用了区别于普通样式的图片，在demo中有示例的png格式图片，根据其大小直接修改成自己需要的即可。在MainActivity中修改initFragments()，在noteFragment.setIconId()中控制导航栏图标。

最后有人可能需要对图标下的文字颜色进行修改。打开library中的color.xml直接在里面修改即可。

```
<?xml version="1.0" encoding="utf-8"?>
<resources>    
   <color name="tab_text_normal">#696969</color> 
   <color name="tab_text_selected">#E8860A</color>
</resources>
```

![](http://upload-images.jianshu.io/upload_images/2013918-2c865a18e5ff90b8.gif?imageMogr2/auto-orient/strip)

### 参考链接

[Android底部导航栏控件的使用 - 简书](http://www.jianshu.com/p/9683e3112159)


### ViewPager+Fragment+FragmentTabHost实现底部菜单


**参考链接**     
[Android开发之ViewPager+Fragment+FragmentTabHost实现底部菜单 - 简书](http://www.jianshu.com/p/f7351d4ee903)

### 完成






























