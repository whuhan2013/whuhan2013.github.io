---
layout: post
title: 百度地图之添加覆盖物
date: 2016-4-17
categories: blog
tags: [android,地图开发]
description: 百度地图之添加覆盖物
---

### 本文主要讲解如何实现在百度地图上添加覆盖物 


### 1.承载数据的实体
我们从服务器返回的数据部分，最终可能是个Json数组，我们需要转换为实体集合，即下面的Info.java     
我直接在实体类中声明了一个静态列表集合，模拟从服务器返回的数据Info.infos


```
package com.zj.map2;

import java.io.Serializable;  
import java.util.ArrayList;  
import java.util.List;  
  
public class Info implements Serializable  
{  
    private static final long serialVersionUID = -758459502806858414L;  
    /** 
     * 精度 
     */  
    private double latitude;  
    /** 
     * 纬度 
     */  
    private double longitude;  
    /** 
     * 图片ID，真实项目中可能是图片路径 
     */  
    private int imgId;  
    /** 
     * 商家名称 
     */  
    private String name;  
    /** 
     * 距离 
     */  
    private String distance;  
    /** 
     * 赞数量 
     */  
    private int zan;  
  
    public static List<Info> infos = new ArrayList<Info>();  
  
    static  
    {  
        infos.add(new Info(34.242652, 108.971171, R.drawable.a01, "英伦贵族小旅馆",  
                "距离209米", 1456));  
        infos.add(new Info(34.242952, 108.972171, R.drawable.a02, "沙井国际洗浴会所",  
                "距离897米", 456));  
        infos.add(new Info(34.242852, 108.973171, R.drawable.a03, "五环服装城",  
                "距离249米", 1456));  
        infos.add(new Info(34.242152, 108.971971, R.drawable.a04, "老米家泡馍小炒",  
                "距离679米", 1456));  
    }  
  
    public Info()  
    {  
    }  
  
    public Info(double latitude, double longitude, int imgId, String name,  
            String distance, int zan)  
    {  
        super();  
        this.latitude = latitude;  
        this.longitude = longitude;  
        this.imgId = imgId;  
        this.name = name;  
        this.distance = distance;  
        this.zan = zan;  
    }  
  
    public double getLatitude()  
    {  
        return latitude;  
    }  
  
    public void setLatitude(double latitude)  
    {  
        this.latitude = latitude;  
    }  
  
    public double getLongitude()  
    {  
        return longitude;  
    }  
  
    public void setLongitude(double longitude)  
    {  
        this.longitude = longitude;  
    }  
  
    public String getName()  
    {  
        return name;  
    }  
  
    public int getImgId()  
    {  
        return imgId;  
    }  
  
    public void setImgId(int imgId)  
    {  
        this.imgId = imgId;  
    }  
  
    public void setName(String name)  
    {  
        this.name = name;  
    }  
  
    public String getDistance()  
    {  
        return distance;  
    }  
  
    public void setDistance(String distance)  
    {  
        this.distance = distance;  
    }  
  
    public int getZan()  
    {  
        return zan;  
    }  
  
    public void setZan(int zan)  
    {  
        this.zan = zan;  
    }  
  
}  
```   

### 在地图中添加overlay 

```
/** 
     * 初始化图层 
     */  
    public void addInfosOverlay(List<Info> infos)  
    {  
        mBaiduMap.clear();  
        LatLng latLng = null;  
        OverlayOptions overlayOptions = null;  
        Marker marker = null;  
        for (Info info : infos)  
        {  
            // 位置  
            latLng = new LatLng(info.getLatitude(), info.getLongitude());  
            // 图标  
            overlayOptions = new MarkerOptions().position(latLng)  
                    .icon(mIconMaker).zIndex(5);  
            marker = (Marker) (mBaiduMap.addOverlay(overlayOptions));  
            Bundle bundle = new Bundle();  
            bundle.putSerializable("info", info);  
            marker.setExtraInfo(bundle);  
        }  
        // 将地图移到到最后一个经纬度位置  
        MapStatusUpdate u = MapStatusUpdateFactory.newLatLng(latLng);  
        mBaiduMap.setMapStatus(u);  
    }  
```    


### 为地图上的maker添加点击事件  

（记得生成TextView的时候，先设置背景，再设置padding，不然可能会失效~~~）

```
private void initMarkerClickEvent()
    {
        // 对Marker的点击
        mBaiduMap.setOnMarkerClickListener(new OnMarkerClickListener()
        {
            @Override
            public boolean onMarkerClick(final Marker marker)
            {
                // 获得marker中的数据
                Info info = (Info) marker.getExtraInfo().get("info");

                InfoWindow mInfoWindow;
                // 生成一个TextView用户在地图中显示InfoWindow
                TextView location = new TextView(getApplicationContext());
                location.setBackgroundResource(R.drawable.location_tips);
                location.setPadding(30, 20, 30, 50);
                location.setText(info.getName());
                // 将marker所在的经纬度的信息转化成屏幕上的坐标
                final LatLng ll = marker.getPosition();
                Point p = mBaiduMap.getProjection().toScreenLocation(ll);
                p.y -= 47;
                LatLng llInfo = mBaiduMap.getProjection().fromScreenLocation(p);
                // 为弹出的InfoWindow添加点击事件
                //mInfoWindow=new InfoWindow(location);
                mInfoWindow = new InfoWindow(location, llInfo,0);
                location.setOnClickListener(new OnClickListener() {
                    
                    @Override
                    public void onClick(View v) {
                        // TODO Auto-generated method stub
                        mBaiduMap.hideInfoWindow();
                    }
                });
                // 显示InfoWindow
                mBaiduMap.showInfoWindow(mInfoWindow);
                // 设置详细信息布局为可见
                mMarkerInfoLy.setVisibility(View.VISIBLE);
                // 根据商家信息为详细信息布局设置信息
                popupInfo(mMarkerInfoLy, info);
                return true;
            }
        });
    }

```  


### 布局文件  

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="${relativePackage}.${activityClass}" >

    <com.baidu.mapapi.map.MapView
        android:id="@+id/id_bmapView"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:clickable="true" />
    
     <RelativeLayout  
        android:id="@+id/id_marker_info"  
        android:visibility="gone"  
        android:layout_width="fill_parent"  
        android:layout_height="220dp"  
        android:layout_alignParentBottom="true"  
        android:background="#CC4e5a6b"  
        android:clickable="true" >  
  
        <ImageView  
            android:id="@+id/info_img"  
            android:layout_width="fill_parent"  
            android:layout_height="150dp"  
            android:layout_marginBottom="10dp"  
            android:layout_marginLeft="12dp"  
            android:layout_marginRight="12dp"  
            android:layout_marginTop="10dp"  
            android:alpha="1.0"  
            android:background="@drawable/map_image_border_white"  
            android:clickable="true"  
            android:scaleType="fitXY"  
            android:src="@drawable/a04" />  
  
        <RelativeLayout  
            android:layout_width="fill_parent"  
            android:layout_height="50dp"  
            android:layout_alignParentBottom="true"  
            android:background="@drawable/bg_map_bottom" >  
  
            <LinearLayout  
                android:layout_width="fill_parent"  
                android:layout_height="wrap_content"  
                android:layout_centerVertical="true"  
                android:layout_marginLeft="20dp"  
                android:orientation="vertical" >  
  
                <TextView  
                    android:id="@+id/info_name"  
                    android:layout_width="wrap_content"  
                    android:layout_height="wrap_content"  
                    android:text="老米家泡馍小炒"  
                    android:textColor="#FFF5EB" />  
  
                <TextView  
                    android:id="@+id/info_distance"  
                    android:layout_width="wrap_content"  
                    android:layout_height="wrap_content"  
                    android:text="距离200米"  
                    android:textColor="#FFF5EB" />  
            </LinearLayout>  
  
            <LinearLayout  
                android:layout_width="wrap_content"  
                android:layout_height="wrap_content"  
                android:layout_alignParentRight="true"  
                android:layout_centerVertical="true"  
                android:layout_marginRight="20dp"  
                android:orientation="horizontal" >  
  
                <ImageView  
                    android:layout_width="wrap_content"  
                    android:layout_height="wrap_content"  
                    android:onClick="zan"  
                    android:src="@drawable/map_zan" />  
  
                <TextView  
                    android:id="@+id/info_zan"  
                    android:layout_width="wrap_content"  
                    android:layout_height="wrap_content"  
                    android:layout_gravity="center"  
                    android:text="652"  
                    android:textColor="#FFF5EB" />  
            </LinearLayout>  
        </RelativeLayout>  
    </RelativeLayout>  

</RelativeLayout>
```    

### 注意imageView的background为map_image_border_white，是白色边框的背景

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >

    <stroke
        android:width="2dp"
        android:color="#AAffffff" />

    <padding
        android:bottom="2dp"
        android:left="2dp"
        android:right="2dp"
        android:top="2dp" />

</shape>
```



###  MainActivity完整代码  

```
package com.zj.map2;

import java.util.List;

import com.baidu.mapapi.SDKInitializer;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.BaiduMap.OnMapClickListener;
import com.baidu.mapapi.map.BaiduMap.OnMarkerClickListener;
import com.baidu.mapapi.map.BitmapDescriptor;
import com.baidu.mapapi.map.BitmapDescriptorFactory;
import com.baidu.mapapi.map.InfoWindow;
import com.baidu.mapapi.map.MapPoi;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.map.Marker;
import com.baidu.mapapi.map.MarkerOptions;
import com.baidu.mapapi.map.OverlayOptions;
import com.baidu.mapapi.model.LatLng;

import android.app.Activity;
import android.graphics.Point;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.ImageView;
import android.widget.RelativeLayout;
import android.widget.TextView;

public class MainActivity extends Activity {
    
    MapView mapView;
    BaiduMap mBaiduMap;
    // 初始化全局 bitmap 信息，不用时及时 recycle
    private BitmapDescriptor mIconMaker;
    private RelativeLayout mMarkerInfoLy;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SDKInitializer.initialize(getApplicationContext());
        setContentView(R.layout.activity_main);
        
        mapView=(MapView) findViewById(R.id.id_bmapView);
        mMarkerInfoLy = (RelativeLayout) findViewById(R.id.id_marker_info);
        mBaiduMap=mapView.getMap();
        
        mIconMaker = BitmapDescriptorFactory.fromResource(R.drawable.maker);
        MapStatusUpdate msu = MapStatusUpdateFactory.zoomTo(15.0f);
        mBaiduMap.setMapStatus(msu);
        
        
        initMarkerClickEvent();
        initMapClickEvent();

    }
    
    private void initMapClickEvent()
    {
        mBaiduMap.setOnMapClickListener(new OnMapClickListener()
        {

            @Override
            public boolean onMapPoiClick(MapPoi arg0)
            {
                return false;
            }

            @Override
            public void onMapClick(LatLng arg0)
            {
                mMarkerInfoLy.setVisibility(View.GONE);
                mBaiduMap.hideInfoWindow();

            }
        });
    }
    
    
    private void initMarkerClickEvent()
    {
        // 对Marker的点击
        mBaiduMap.setOnMarkerClickListener(new OnMarkerClickListener()
        {
            @Override
            public boolean onMarkerClick(final Marker marker)
            {
                // 获得marker中的数据
                Info info = (Info) marker.getExtraInfo().get("info");

                InfoWindow mInfoWindow;
                // 生成一个TextView用户在地图中显示InfoWindow
                TextView location = new TextView(getApplicationContext());
                location.setBackgroundResource(R.drawable.location_tips);
                location.setPadding(30, 20, 30, 50);
                location.setText(info.getName());
                // 将marker所在的经纬度的信息转化成屏幕上的坐标
                final LatLng ll = marker.getPosition();
                Point p = mBaiduMap.getProjection().toScreenLocation(ll);
                p.y -= 47;
                LatLng llInfo = mBaiduMap.getProjection().fromScreenLocation(p);
                // 为弹出的InfoWindow添加点击事件
                //mInfoWindow=new InfoWindow(location);
                mInfoWindow = new InfoWindow(location, llInfo,0);
                location.setOnClickListener(new OnClickListener() {
                    
                    @Override
                    public void onClick(View v) {
                        // TODO Auto-generated method stub
                        mBaiduMap.hideInfoWindow();
                    }
                });
                // 显示InfoWindow
                mBaiduMap.showInfoWindow(mInfoWindow);
                // 设置详细信息布局为可见
                mMarkerInfoLy.setVisibility(View.VISIBLE);
                // 根据商家信息为详细信息布局设置信息
                popupInfo(mMarkerInfoLy, info);
                return true;
            }
        });
    }
    
    
    /** 
     * 根据info为布局上的控件设置信息 
     *  
     * @param mMarkerInfo2 
     * @param info 
     */  
    protected void popupInfo(RelativeLayout mMarkerLy, Info info)  
    {  
        ViewHolder viewHolder = null;  
        if (mMarkerLy.getTag() == null)  
        {  
            viewHolder = new ViewHolder();  
            viewHolder.infoImg = (ImageView) mMarkerLy  
                    .findViewById(R.id.info_img);  
            viewHolder.infoName = (TextView) mMarkerLy  
                    .findViewById(R.id.info_name);  
            viewHolder.infoDistance = (TextView) mMarkerLy  
                    .findViewById(R.id.info_distance);  
            viewHolder.infoZan = (TextView) mMarkerLy  
                    .findViewById(R.id.info_zan);  
  
            mMarkerLy.setTag(viewHolder);  
        }  
        viewHolder = (ViewHolder) mMarkerLy.getTag();  
        viewHolder.infoImg.setImageResource(info.getImgId());  
        viewHolder.infoDistance.setText(info.getDistance());  
        viewHolder.infoName.setText(info.getName());  
        viewHolder.infoZan.setText(info.getZan() + "");  
    }  
    
    /** 
     * 复用弹出面板mMarkerLy的控件 
     *  
     * @author zhy 
     *  
     */  
    private class ViewHolder  
    {  
        ImageView infoImg;  
        TextView infoName;  
        TextView infoDistance;  
        TextView infoZan;  
    }  
    
    

    
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // TODO Auto-generated method stub
        getMenuInflater().inflate(R.menu.main, menu);
        return super.onCreateOptionsMenu(menu);
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // TODO Auto-generated method stub
        switch (item.getItemId()) {
        case R.id.id_menu_map_addMaker:
             addInfosOverlay(Info.infos);
            break;

        default:
            break;
        }
        return super.onOptionsItemSelected(item);
    }
    
    /**
     * 初始化图层
     */
    public void addInfosOverlay(List<Info> infos)
    {
        mBaiduMap.clear();
        LatLng latLng = null;
        OverlayOptions overlayOptions = null;
        Marker marker = null;
        for (Info info : infos)
        {
            // 位置
            latLng = new LatLng(info.getLatitude(), info.getLongitude());
            // 图标
            overlayOptions = new MarkerOptions().position(latLng)
                    .icon(mIconMaker).zIndex(5);
            marker = (Marker) (mBaiduMap.addOverlay(overlayOptions));
            Bundle bundle = new Bundle();
            bundle.putSerializable("info", info);
            marker.setExtraInfo(bundle);
        }
        // 将地图移到到最后一个经纬度位置
        MapStatusUpdate u = MapStatusUpdateFactory.newLatLng(latLng);
        mBaiduMap.setMapStatus(u);
    }
}
```    


### 参考链接：
[Android 百度地图 SDK v3.0.0 （三） 添加覆盖物Marker与InfoWindow的使用 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/37737213)

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160417152356403)




























