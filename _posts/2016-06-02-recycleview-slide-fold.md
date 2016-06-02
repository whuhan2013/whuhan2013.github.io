---
layout: post
title: RecyclerView实现滑动折叠效果
date: 2016-6-2
categories: blog
tags: [自定义控件]
description: RecyclerView实现滑动折叠效果
---

### 转载   

**效果预览，真机上看到的效果会很清晰**  

![](http://upload-images.jianshu.io/upload_images/697635-f8a052530e974b4c.gif?imageMogr2/auto-orient/strip)


### View选择   

上文中提到的效果是使用ListView实现的，我也尝试在它的基础上完善功能，但是各种奇葩的问题随之而来，焦点问题，快滑问题，还有辛苦添加的HeaderView居然无法动态更新数据和UI，最后想到了Google的新控件RecyclerView，本文的主角正是RecyclerView,可以选择是否添加headView与footView,可自由添加与删除.

**效果实现关键点**           
只需关心界面上第一个可见的ItemView和界面上第一个完全可见的ItemView

```
//界面上第一个可见的ItemView
linearLayoutManager.findFirstVisibleItemPosition()
//界面上第一个完全可见的ItemView
linearLayoutManager.findFirstCompletelyVisibleItemPosition()
```


通过ViewHolder初始化所有ItemView，第一个ItemView高度最高，其余的都是普通高度

```
 @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        if (isFirst && position == 0) {
            isFirst = false;
            holder.itemView.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, item_max_height));
        } else {
          holder.itemView.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, item_normal_height));
        }
        holder.imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
        Glide.with(holder.imageView.getContext()).load(walls.get(position)).into(holder.imageView);

    }
```


图片高度固定为最高Item高度，图片放缩类型为scaleType，不然折叠效果出不来

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/image"
        android:layout_width="match_parent"
        android:layout_height="@dimen/item_max_height"
        android:scaleType="centerCrop" />
</FrameLayout>
```


纵向滑动距离监听，RecyclerView提供了这样的接口回调

```
   recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                // TODO: 2015/12/6 根据纵向滑动距离dy实现滑动折叠，阴影渐变，文字放缩效果 
            }
        });
```

关于怎么样利用dy来实现炫酷的动画效果我就不细说了，我会稍后将代码传到github，


### 有两个adapter,一个是真正的，一个是实现加头部与尾部的adapter

### 真正的adapter

```
package com.yyydjk.miaojiedemo;

import android.content.Context;
import android.support.design.widget.Snackbar;
import android.support.v7.widget.RecyclerView;
import android.util.TypedValue;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;

import com.bumptech.glide.Glide;

import java.util.List;

/**
 * Created by dongjunkun on 2015/12/1.
 */
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
    private boolean isFirst = true;
    private List<Integer> walls;

    private int item_normal_height;
    private int item_max_height;
    private float item_normal_alpha;
    private float item_max_alpha;
    private float item_normal_font_size;
    private float item_max_font_size;
    Context mContext;
    static RecyclerView mrecyclerView;

    public MyAdapter(Context context, List<Integer> walls,RecyclerView recyclerView) {
        this.walls = walls;
        item_max_height = (int) context.getResources().getDimension(R.dimen.item_max_height);
        item_normal_height = (int) context.getResources().getDimension(R.dimen.item_normal_height);
        item_normal_font_size = context.getResources().getDimension(R.dimen.item_normal_font_size);
        item_max_font_size = context.getResources().getDimension(R.dimen.item_max_font_size);
        item_normal_alpha = context.getResources().getFraction(R.fraction.item_normal_mask_alpha, 1, 1);
        item_max_alpha = context.getResources().getFraction(R.fraction.item_max_mask_alpha, 1, 1);
        mContext=context;
        mrecyclerView=recyclerView;
    }

    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_wall, null);

        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        if (isFirst && position == 0) {
            isFirst = false;
            holder.mark.setAlpha(item_max_alpha);
            holder.text.setTextSize(TypedValue.COMPLEX_UNIT_PX, item_max_font_size);
            holder.itemView.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, item_max_height));
        } else {
            holder.mark.setAlpha(item_normal_alpha);
            holder.text.setTextSize(TypedValue.COMPLEX_UNIT_PX, item_normal_font_size);
            holder.itemView.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, item_normal_height));
        }
        holder.text.setText(String.format("光谷天地%d", position));
        holder.imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
        Glide.with(holder.imageView.getContext()).load(walls.get(position)).into(holder.imageView);

    }

    @Override
    public int getItemCount() {
        return walls.size();
    }

    public static class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener{
        public ImageView imageView;
        public View mark;
        public TextView text;

        public ViewHolder(View itemView) {
            super(itemView);
            imageView = (ImageView) itemView.findViewById(R.id.image);
            mark = itemView.findViewById(R.id.mark);
            text = (TextView) itemView.findViewById(R.id.text);
            itemView.setOnClickListener(this);
        }


        @Override
        public void onClick(View view) {
            String text1=text.getText().toString();
            int positon=mrecyclerView.getChildAdapterPosition(view);

            Snackbar.make(view, text1 + positon, Snackbar.LENGTH_SHORT).show();
        }
    }
}
```


**将myadapter传到HeaderAndFooterRecyclerViewAdapter中，然后recyclerView设置其为adapter**

```
HeaderAndFooterRecyclerViewAdapter headerAndFooterRecyclerViewAdapter = new HeaderAndFooterRecyclerViewAdapter(myAdapter);
        recyclerView.setAdapter(headerAndFooterRecyclerViewAdapter);

```

### 源代码

[代码下载](https://github.com/dongjunkun/miaojiedemo)     


### 原文链接：

[android版高仿喵街主页滑动效果 - 简书](http://www.jianshu.com/p/a2c3c21e3b99)




