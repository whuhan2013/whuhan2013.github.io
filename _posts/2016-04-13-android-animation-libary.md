---
layout: post
title: 安卓开源库之动画篇
date: 2016-4-13
categories: blog
tags: [android]
description: 安卓开源库之动画篇
---

### 本文主要介绍收集了笔者所用过的开源动画库，达到一些比较好看的效果。

### 一个富有动感的 Sheet  

### 链接：         
[zzz40500/AndroidSweetSheet: 一个富有动感的Sheet(选择器)](https://github.com/zzz40500/AndroidSweetSheet)

### 效果如下 

![这里写图片描述](http://img.blog.csdn.net/20160413132555245)

### 示例代码 

```
package com.zj.testsheet;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.RelativeLayout;
import android.widget.Toast;

import com.mingle.entity.MenuEntity;
import com.mingle.sweetpick.BlurEffect;
import com.mingle.sweetpick.RecyclerViewDelegate;
import com.mingle.sweetpick.SweetSheet;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    SweetSheet mSweetSheet;
    RelativeLayout rl;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        rl= (RelativeLayout) findViewById(R.id.rl);
        // SweetSheet 控件,根据 rl 确认位置
        mSweetSheet = new SweetSheet(rl);


        final ArrayList<MenuEntity> list = new ArrayList<>();
        //添加假数据
        MenuEntity menuEntity1 = new MenuEntity();
        menuEntity1.iconId = R.drawable.ic_account_child;
        //menuEntity1.titleColor = 0xff000000;
        menuEntity1.title = "code";
        MenuEntity menuEntity = new MenuEntity();
        menuEntity.iconId = R.drawable.ic_account_child;
        //menuEntity.titleColor = 0xffb3b3b3;
        menuEntity.title = "QQ";
        list.add(menuEntity1);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);
        list.add(menuEntity);

//设置数据源 (数据源支持设置 list 数组,也支持从menu 资源中获取)
        mSweetSheet.setMenuList(list);

        SweetSheet mSweetSheet2 = new SweetSheet(rl);

        //从menu 中设置数据源
        mSweetSheet2.setMenuList(R.menu.menu_sweet);
//根据设置不同的 Delegate 来显示不同的风格.
        mSweetSheet.setDelegate(new RecyclerViewDelegate(true));
//根据设置不同Effect来设置背景效果:BlurEffect 模糊效果.DimEffect 变暗效果,NoneEffect 没有效果.
        mSweetSheet.setBackgroundEffect(new BlurEffect(8));
//设置菜单点击事件
        mSweetSheet.setOnMenuItemClickListener(new SweetSheet.OnMenuItemClickListener() {
            @Override
            public boolean onItemClick(int position, MenuEntity menuEntity1) {

                //根据返回值, true 会关闭 SweetSheet ,false 则不会.
                Toast.makeText(MainActivity.this, menuEntity1.title + "  " + position, Toast.LENGTH_SHORT).show();
                return true;
            }
        });

        mSweetSheet.show();
    }
}
``` 


### A fluent Android animation library。安卓动画库
主要包括一些进度条效果 

### 链接：
[gzu-liyujiang/ViewAnimator: A fluent Android animation library。安卓动画库，加入了一些不错的动画，如：fall、shake、flash、fadeIn、rollOut……支持任意路径动画（示例动画为不断冒出来的桃心及飘雪），支持按SVG格式的path运动。部分已合并到原作者florent37的主分支](https://github.com/gzu-liyujiang/ViewAnimator) 

### 效果如下 

![这里写图片描述](http://img.blog.csdn.net/20160413132910547)

### 示例代码 

```
private void animateParallel() {
        ViewAnimator.animate(mountain, image)
                .dp().translationY(-1000, 0)
                .alpha(0, 1)

                .andAnimate(percent)
                .scale(0, 1)

                .andAnimate(text)
                .dp().translationY(1000, 0)
                .textColor(Color.BLACK, Color.WHITE)
                .backgroundColor(Color.WHITE, Color.BLACK)

                .waitForHeight()
                .interpolator(new AccelerateDecelerateInterpolator())
                .duration(1000)

                .thenAnimate(percent)
                .custom(new AnimationListener.Update<TextView>() {
                    @Override
                    public void update(TextView view, float value) {
                        value = value * 100;
                        view.setText(String.format(Locale.US, "%.0f%%", value));
                    }
                }, 0, 1)

                .andAnimate(image)
                .rotation(0, 360)
                .onStop(new AnimationListener.Stop() {
                    @Override
                    public void onStop() {
                        Intent intent = new Intent(StartActivity.this, MainActivity.class);
                        startActivity(intent);
                    }
                })

                .duration(1000)



                .start();
}
```

### recycleView动画效果 

### 链接：
[gabrielemariotti/RecyclerViewItemAnimators: An Android library which provides simple Item animations to RecyclerView items](https://github.com/gabrielemariotti/RecyclerViewItemAnimators)

### 效果如下：
![这里写图片描述](http://img.blog.csdn.net/20160413134947958)

### 示例代码  

```

recyclerView= (RecyclerView) findViewById(R.id.id_recyclerview);
recyclerView.setLayoutManager(new GridLayoutManager(this, 4));
mAdapter=new HomeAdapter();
slideInBottomAnimatorAdapter=new SlideInBottomAnimatorAdapter(mAdapter,recyclerView);
 recyclerView.setAdapter(slideInBottomAnimatorAdapter);

 ```























