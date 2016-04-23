---
layout: post
title: Android之ExpandableListView
date: 2016-4-23
categories: blog
tags: [android]
description: Android之ExpandableListView
---

### ExpandableListView可以用来表现多层级的listView，本文主要是ExpandableListView的一个简单实现  

### 布局文件  

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
   <LinearLayout 
       android:layout_height="50dip"
       android:layout_width="match_parent"
       android:background="#297DC6">
       <TextView 
           android:layout_marginLeft="100dip"
           android:layout_height="wrap_content"
           android:layout_width="wrap_content"
           android:text="Email"
           android:textColor="#ffffff"
           android:textSize="20sp"
           android:layout_gravity="center_vertical"/>
   </LinearLayout>
   <ExpandableListView 
       android:id="@+id/list"
       android:layout_height="match_parent"
       android:layout_width="match_parent"/>

</LinearLayout>
```   


### MainActivity实现  

```
package com.zj.expandandview;





import android.annotation.SuppressLint;
import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseExpandableListAdapter;
import android.widget.ExpandableListView;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.ExpandableListView.OnChildClickListener;
import android.widget.ExpandableListView.OnGroupClickListener;

public class MainActivity extends Activity {
  
  private ExpandableListView expendView;
  private int []group_click=new int[5];

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    
        final MyExpendAdapter adapter=new MyExpendAdapter();
    
    expendView=(ExpandableListView) findViewById(R.id.list);
    expendView.setGroupIndicator(null);  //设置默认图标不显示
    expendView.setAdapter(adapter);
    
    //一级点击事件
    expendView.setOnGroupClickListener(new OnGroupClickListener() {
      @Override
      public boolean onGroupClick(ExpandableListView parent, View v,
          int groupPosition, long id) {
      
        group_click[groupPosition]+=1;
        adapter.notifyDataSetChanged();
        return false;
      }
    });
    
    //二级点击事件
    expendView.setOnChildClickListener(new OnChildClickListener() { 
      @Override
      public boolean onChildClick(ExpandableListView parent, View v,
          int groupPosition, int childPosition, long id) {
        //可在这里做点击事件
        if(groupPosition==0&&childPosition==1){
          
        }else if(groupPosition==0&&childPosition==0){
          //Intent intent=new Intent(MainActivity.this, MailConstactsActivity.class);
          //startActivity(intent);
        }else if(groupPosition==1&&childPosition==0){
          //Intent intent=new Intent(MainActivity.this, MailEditActivity.class);
          //startActivity(intent);
        }else if(groupPosition==1&&childPosition==1){
          //Intent intent=new Intent(MainActivity.this, MailCaogaoxiangActivity.class);
          //startActivity(intent);
        }else if(groupPosition==2&&childPosition==0){
          
        }else if(groupPosition==2&&childPosition==1){
          //Intent intent=new Intent(MainActivity.this, MailBoxActivity.class);
          //intent.putExtra("TYPE", "INBOX");
          //intent.putExtra("status", 1);//未读
          //startActivity(intent);
        }else if(groupPosition==2&&childPosition==2){
          //Intent intent=new Intent(MainActivity.this, MailBoxActivity.class);
          //intent.putExtra("TYPE", "INBOX");
          //intent.putExtra("status", 2);//已读
          //startActivity(intent);
        }
        adapter.notifyDataSetChanged();
        return false;
      }
    });
    
  }
  
  
  /**
   * 适配器
   * @author Administrator
   *
   */
  private class MyExpendAdapter extends BaseExpandableListAdapter{
    
    /**
     * pic state
     */
    //int []group_state=new int[]{R.drawable.group_right,R.drawable.group_down};
    
    /**
     * group title
     */
    String []group_title=new String[]{"联系人","写邮件","收件箱"};
    
    /**
     * child text
     */
    String [][] child_text=new String [][]{
        {"联系人列表","添加联系人"},
        {"新邮件","草稿箱"},
        {"全部邮件","未读邮件","已读邮件"},};
    int [][] child_icons=new int[][]{
        {R.drawable.listlianxiren,R.drawable.tianjia},
        {R.drawable.xieyoujian,R.drawable.caogaoxiang},
        {R.drawable.all,R.drawable.notread,R.drawable.hasread},
    };
        
    /**
     * 获取一级标签中二级标签的内容
     */
    @Override
    public Object getChild(int groupPosition, int childPosition) {
      return child_text[groupPosition][childPosition];
    }
        
    /**
     * 获取二级标签ID
     */
    @Override
    public long getChildId(int groupPosition, int childPosition) {
      return childPosition;
    }
    /**
     * 对一级标签下的二级标签进行设置
     */
    @SuppressLint("SimpleDateFormat")
    @Override
    public View getChildView(int groupPosition, int childPosition,
        boolean isLastChild, View convertView, ViewGroup parent) {
      convertView=getLayoutInflater().inflate(R.layout.email_child, null);
      TextView tv=(TextView) convertView.findViewById(R.id.tv);
      tv.setText(child_text[groupPosition][childPosition]);
      
      ImageView iv=(ImageView) convertView.findViewById(R.id.child_icon);
      iv.setImageResource(child_icons[groupPosition][childPosition]);
      return convertView;
    }
        
    /**
     * 一级标签下二级标签的数量
     */
    @Override
    public int getChildrenCount(int groupPosition) {
      return child_text[groupPosition].length;
    }
        
    /**
     * 获取一级标签内容
     */
    @Override
    public Object getGroup(int groupPosition) {
      return group_title[groupPosition];
    }
        
    /**
     * 一级标签总数
     */
    @Override
    public int getGroupCount() {
      return group_title.length;
    }
        
    /**
     * 一级标签ID
     */
    @Override
    public long getGroupId(int groupPosition) {
      return groupPosition;
    }
    /**
     * 对一级标签进行设置
     */
    @Override
    public View getGroupView(int groupPosition, boolean isExpanded,
        View convertView, ViewGroup parent) {
      convertView=getLayoutInflater().inflate(R.layout.email_group, null);
      
      ImageView icon=(ImageView) convertView.findViewById(R.id.icon);
      ImageView iv=(ImageView) convertView.findViewById(R.id.iv);
      TextView tv=(TextView) convertView.findViewById(R.id.iv_title);
      
      iv.setImageResource(R.drawable.group_right);
      tv.setText(group_title[groupPosition]);
      
      if(groupPosition==0){
        icon.setImageResource(R.drawable.constants);
      }else if(groupPosition==1){
        icon.setImageResource(R.drawable.mailto);
      }else if(groupPosition==2){
        icon.setImageResource(R.drawable.mailbox);
      }
      
      if(group_click[groupPosition]%2==0){
        iv.setImageResource(R.drawable.group_right);
      }else{
        iv.setImageResource(R.drawable.group_down);
      }
      
      
      return convertView;
    }
    /**
     * 指定位置相应的组视图
     */
    @Override
    public boolean hasStableIds() {
      return true;
    }
        
    /**
     *  当选择子节点的时候，调用该方法
     */
    @Override
    public boolean isChildSelectable(int groupPosition, int childPosition) {
      return true;
    }
    
  }
}
```  


### 参考链接 :
[Android中ExpandableListView控件基本使用 - Android-Idea - 博客频道 - CSDN.NET](http://blog.csdn.net/cjjky/article/details/6903504)

### 完成，效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160423091143488)






























