---
layout: post
title: FragmentPagerAdapter实现刷新
date: 2016-6-6
categories: blog
tags: [android]
description: FragmentPagerAdapter实现刷新
---

在fragmentpageadapter的instantiateItem方法里，他会先去FragmentManager里面去查找有没有相关的fragment如果有就直接使用如果没有才会触发fragmentpageadapter的getItem方法获取一个fragment。所以你更新fragments集合是没有作用的。


### 所以要用新的方法实现刷新功能 

**主要思路**   

就是用新的fragment替换FragmentManager里缓存的旧的fragment，

在系统的代码中     

```
        String name = makeFragmentName(container.getId(), position);
        Fragment fragment = mFragmentManager.findFragmentByTag(name);
```


   说明fragmentpageadapter内部是用tag识别fragment的，并且有它自己的一套算法用于生成tag，所以创建是它已经有了自己的tag,不用我们赋值。 

所以我们这里必须用它生成的tag来添加新的fragment,否则fragmentpageadapter就无法识别这个新的fragment。



### 实例 


**更换fragment**

```
List<Fragment> fragments = new ArrayList<>();
                fragments.add(new PoliceFragment());
                fragments.add(new GirlFragment());
                fragments.add(new ThirdFragment());
                boolean[] fragmentsUpdateFlag = { false, false, true};

                TabsPagerAdapter adapter = new TabsPagerAdapter(getSupportFragmentManager(), fragments,fragmentsUpdateFlag);
                mVP.setAdapter(adapter);
                mVP.getAdapter().notifyDataSetChanged();
                mTab.setupWithViewPager(mVP);
```

**自定义Adapter的实现**


```
package com.zj.adapter;

import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.app.FragmentTransaction;
import android.view.ViewGroup;

import java.util.List;

/**
 * Created by CoXier on 2016/5/2.
 */

public class TabsPagerAdapter extends FragmentPagerAdapter {
    List<Fragment>  mFragments;
    FragmentManager fm;
    private int curUpdatePager;
    String[] titles = {"警察风采","在逃嫌犯","新闻资讯"};
    boolean[] fragmentsUpdateFlag;

    public TabsPagerAdapter(FragmentManager fm, List<Fragment> mFragments,boolean[] fragmentsUpdateFlag) {
        super(fm);
        this.fm=fm;
        this.mFragments = mFragments;
        this.fragmentsUpdateFlag=fragmentsUpdateFlag;
    }

    @Override
    public Fragment getItem(int position) {

        return mFragments.get(position);

    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        //得到缓存的fragment
        Fragment fragment = (Fragment) super.instantiateItem(container,
                position);
        //得到tag，这点很重要
        String fragmentTag = fragment.getTag();

        if (fragmentsUpdateFlag[position % fragmentsUpdateFlag.length]) {
            //如果这个fragment需要更新

            FragmentTransaction ft = fm.beginTransaction();
            //移除旧的fragment
            ft.remove(fragment);
            //换成新的fragment
            fragment = mFragments.get(position % mFragments.size());
            //添加新fragment时必须用前面获得的tag，这点很重要
            ft.add(container.getId(), fragment, fragmentTag);
            ft.attach(fragment);
            ft.commit();

            //复位更新标志
            fragmentsUpdateFlag[position % fragmentsUpdateFlag.length] = false;
        }

        return fragment;
    }

    @Override
    public int getCount() {
        return mFragments.size();
    }



    @Override
    public CharSequence getPageTitle(int position) {
        return titles[position];
    }
}
```


### 参考链接

[FragmentPagerAdapter刷新fragment最完美解决方案 - z13759561330的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/z13759561330/article/details/40737381)