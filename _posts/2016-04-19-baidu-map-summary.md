---
layout: post
title: 百度地图综合
date: 2016-4-19
categories: blog
tags: [android,地图开发]
description: 百度地图综合
---

### 本文主要包括百度地图API的综合应用，主要内容如下  

1. 地图图层展示，包括热力图与实时路况图   
2. 添加覆盖物，包括图片，文字，折线等
3. 地图控制，包括俯视，旋转，放大，缩小等
4. 定位，并且用图标标示出来
5. POI检索，检索出范围内的兴趣点
6. 公交线路查询
7. 路线规划，包括驾车，公交，步行。

![这里写图片描述](http://img.blog.csdn.net/20160419200937576)


### 主界面

```
package com.zj.mapall;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.ArrayAdapter;
import android.widget.ListView;

/**
 * 程序启动引导页，选择不同的功能进入不同的界面
 * 
 * @author ys
 *
 */
public class LaunchActivity extends Activity {

    private ListView listview;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_launch);
        listview = (ListView) findViewById(R.id.activity_listview);
        init();
    }

    /**
     * 初始化listview列表
     */
    private void init() {
        final Class[] clazz = { BasisMapActivity.class,AddOverlayActivity.class, MapControllActivity.class
                ,LocationActivity.class,PoiSearchActivity.class,BusLineSearchActivity.class,RoutePlanningActivity.class};
        String arr[] = { "地图图层展示" ,"添加覆盖物" ,"地图控制 ","定位","POI检索","公交线路查询","路线规划"};
        ArrayAdapter<String> adapter = new ArrayAdapter<String>(this,
                android.R.layout.simple_list_item_1, arr);
        listview.setAdapter(adapter);
        listview.setOnItemClickListener(new OnItemClickListener() {

            @Override
            public void onItemClick(AdapterView<?> parent, View view,
                    int position, long id) {
                startActivity(new Intent(LaunchActivity.this, clazz[position]));
            }
        });
    }
}
```    


###  注意，加载地图前应初始化SDKInitializer，可放在Application中初始化  

```
package com.zj.mapall;

import com.baidu.mapapi.SDKInitializer;

import android.app.Application;

public class MyApplication extends Application{
    
    @Override
    public void onCreate() {
        // TODO Auto-generated method stub
        super.onCreate();
        SDKInitializer.initialize(getApplicationContext());
    }

}
```   


### 基础地图展示   

```
package com.zj.mapall;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.Window;
import android.widget.Button;

import com.baidu.mapapi.SDKInitializer;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;

public class BasisMapActivity extends Activity implements OnClickListener {
    // 百度地图控件
    private MapView mMapView = null;
    // 百度地图对象
    private BaiduMap bdMap;
    // 普通地图
    private Button normalMapBtn;
    // 卫星地图
    private Button satelliteMapBtn;
    // 实时路况交通图
    private Button trafficMapBtn;
    // 热力图
    private Button headMapBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        
        setContentView(R.layout.activity_basis_map);
        
        init();
    }

    /**
     * 初始化方法
     */
    private void init() {
        mMapView = (MapView) findViewById(R.id.bmapview);
        mMapView.showZoomControls(false);// 不显示默认的缩放控件

        MapStatusUpdate msu = MapStatusUpdateFactory.zoomTo(15.0f);
        bdMap = mMapView.getMap();
        bdMap.setMapStatus(msu);

        normalMapBtn = (Button) findViewById(R.id.normal_map_btn);
        satelliteMapBtn = (Button) findViewById(R.id.satellite_map_btn);
        trafficMapBtn = (Button) findViewById(R.id.traffic_map_btn);
        headMapBtn = (Button) findViewById(R.id.heat_map_btn);
        
        normalMapBtn.setOnClickListener(this);
        satelliteMapBtn.setOnClickListener(this);
        trafficMapBtn.setOnClickListener(this);
        headMapBtn.setOnClickListener(this);

        //
        normalMapBtn.setEnabled(false);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.normal_map_btn:
            bdMap.setMapType(BaiduMap.MAP_TYPE_NORMAL);
            normalMapBtn.setEnabled(false);
            satelliteMapBtn.setEnabled(true);
            break;
        case R.id.satellite_map_btn:
            bdMap.setMapType(BaiduMap.MAP_TYPE_SATELLITE);
            satelliteMapBtn.setEnabled(false);
            normalMapBtn.setEnabled(true);
            break;
        case R.id.traffic_map_btn:
            if (!bdMap.isTrafficEnabled()) {
                bdMap.setTrafficEnabled(true);
                trafficMapBtn.setText("关闭实时路况");
            } else {
                bdMap.setTrafficEnabled(false);
                trafficMapBtn.setText("打开实时路况");
            }
            break;
        case R.id.heat_map_btn:
            if (!bdMap.isBaiduHeatMapEnabled()) {
                bdMap.setBaiduHeatMapEnabled(true);
                headMapBtn.setText("关闭热力图");
            } else {
                bdMap.setBaiduHeatMapEnabled(false);
                headMapBtn.setText("打开热力图");
            }
            break;
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        mMapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mMapView.onPause();
    }

    @Override
    protected void onDestroy() {
        mMapView.onDestroy();
        mMapView = null;
        super.onDestroy();
    }

}
```   

### 参考链接：
[Android百度地图开发（一）之初体验 - crazy_jack - 博客频道 - CSDN.NET](http://blog.csdn.net/crazy1235/article/details/42614603)

### 添加覆盖物实现   

```
package com.zj.mapall;

import java.util.ArrayList;
import java.util.List;

import com.baidu.mapapi.map.ArcOptions;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.BitmapDescriptor;
import com.baidu.mapapi.map.BitmapDescriptorFactory;
import com.baidu.mapapi.map.CircleOptions;
import com.baidu.mapapi.map.DotOptions;
import com.baidu.mapapi.map.GroundOverlayOptions;
import com.baidu.mapapi.map.InfoWindow;
import com.baidu.mapapi.map.InfoWindow.OnInfoWindowClickListener;
import com.baidu.mapapi.map.MapPoi;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.map.Marker;
import com.baidu.mapapi.map.MarkerOptions;
import com.baidu.mapapi.map.OverlayOptions;
import com.baidu.mapapi.map.PolygonOptions;
import com.baidu.mapapi.map.PolylineOptions;
import com.baidu.mapapi.map.Stroke;
import com.baidu.mapapi.map.TextOptions;
import com.baidu.mapapi.map.BaiduMap.OnMapClickListener;
import com.baidu.mapapi.map.BaiduMap.OnMapDoubleClickListener;
import com.baidu.mapapi.map.BaiduMap.OnMarkerClickListener;
import com.baidu.mapapi.map.BaiduMap.OnMarkerDragListener;
import com.baidu.mapapi.model.LatLng;
import com.baidu.mapapi.model.LatLngBounds;
import com.baidu.mapapi.search.core.SearchResult;
import com.baidu.mapapi.search.geocode.GeoCodeResult;
import com.baidu.mapapi.search.geocode.GeoCoder;
import com.baidu.mapapi.search.geocode.OnGetGeoCoderResultListener;
import com.baidu.mapapi.search.geocode.ReverseGeoCodeOption;
import com.baidu.mapapi.search.geocode.ReverseGeoCodeResult;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.Toast;

/**
 * 添加覆盖物(marker、polygon、text、polyline、dot、circle、arc、ground、)
 * 地图的单击事件、双击事件、marker的拖拽事件    +  地理编码与反地理编码
 * 
 * @author ys
 *
 */
public class AddOverlayActivity extends Activity implements OnClickListener {
    // 百度地图控件
    private MapView mMapView = null;
    // 百度地图对象
    private BaiduMap bdMap;
    // 覆盖物按钮
    private Button overlayBtn;
    // marker
    private Marker marker1;
    // 标记显示第几个覆盖物 1->marker 2->polygon 3->text 4->GroundOverlay(地形图图层) 5->dot
    // 6->circle 7->arc 8->polyline
    private int overlayIndex = 0;
    // 经纬度
    private double latitude = 39.9401752;
    private double longitude = 116.400244;

    // 初始化全局 bitmap 信息，不用时及时 recycle
    // 构建marker图标
    BitmapDescriptor bitmap = BitmapDescriptorFactory
            .fromResource(R.drawable.icon_marka);
    // GroundOptions
    BitmapDescriptor bitmap2 = BitmapDescriptorFactory
            .fromResource(R.drawable.csdn_blog);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_add_overlay);
        mMapView = (MapView) findViewById(R.id.bmapview);

        MapStatusUpdate msu = MapStatusUpdateFactory.zoomTo(15.0f);
        bdMap = mMapView.getMap();
        bdMap.setMapStatus(msu);

        // 对marker覆盖物添加点击事件
        bdMap.setOnMarkerClickListener(new OnMarkerClickListener() {
            @Override
            public boolean onMarkerClick(Marker arg0) {
                if (arg0 == marker1) {
                    final LatLng latLng = arg0.getPosition();
                    // 将经纬度转换成屏幕上的点
                    // Point point =
                    // bdMap.getProjection().toScreenLocation(latLng);
                    Toast.makeText(AddOverlayActivity.this, latLng.toString(),
                            Toast.LENGTH_SHORT).show();
                }
                return false;
            }
        });

        overlayBtn = (Button) findViewById(R.id.overlay_btn);
        overlayBtn.setOnClickListener(this);

        /**
         * 地图单击事件
         */
        bdMap.setOnMapClickListener(new OnMapClickListener() {

            @Override
            public boolean onMapPoiClick(MapPoi arg0) {
                return false;
            }

            @Override
            public void onMapClick(LatLng latLng) {
                displayInfoWindow(latLng);
            }
        });

        /**
         * 地图双击事件
         */
        bdMap.setOnMapDoubleClickListener(new OnMapDoubleClickListener() {
            @Override
            public void onMapDoubleClick(LatLng arg0) {

            }
        });

        /**
         * Marker拖拽事件
         */
        bdMap.setOnMarkerDragListener(new OnMarkerDragListener() {
            @Override
            public void onMarkerDragStart(Marker arg0) {

            }

            @Override
            public void onMarkerDragEnd(Marker arg0) {
                Toast.makeText(
                        AddOverlayActivity.this,
                        "拖拽结束，新位置：" + arg0.getPosition().latitude + ", "
                                + arg0.getPosition().longitude,
                        Toast.LENGTH_LONG).show();
                reverseGeoCode(arg0.getPosition());
            }

            @Override
            public void onMarkerDrag(Marker arg0) {

            }
        });
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.overlay_btn:
            switch (overlayIndex) {
            case 0:
                overlayBtn.setText("显示多边形覆盖物");
                addMarkerOverlay();
                break;
            case 1:
                overlayBtn.setText("显示文字覆盖物");
                addPolygonOptions();
                break;
            case 2:
                overlayBtn.setText("显示地形图图层覆盖物");
                addTextOptions();
                break;
            case 3:
                overlayBtn.setText("显示折线覆盖物");
                addGroundOverlayOptions();
                break;
            case 4:
                overlayBtn.setText("显示圆点覆盖物");
                addPolylineOptions();
                break;
            case 5:
                overlayBtn.setText("显示圆（空心）覆盖物");
                addDotOptions();
                break;
            case 6:
                overlayBtn.setText("显示折线覆盖物");
                addCircleOptions();
                break;
            case 7:
                overlayBtn.setText("显示marker覆盖物");
                addArcOptions();
                break;
            }
            overlayIndex = (overlayIndex + 1) % 8;
            break;
        }
    }

    /**
     * 添加标注覆盖物
     */
    private void addMarkerOverlay() {
        bdMap.clear();
        // 定义marker坐标点
        LatLng point = new LatLng(latitude, longitude);

        // 构建markerOption，用于在地图上添加marker
        OverlayOptions options = new MarkerOptions()//
                .position(point)// 设置marker的位置
                .icon(bitmap)// 设置marker的图标
                .zIndex(9)// 設置marker的所在層級
                .draggable(true);// 设置手势拖拽
        // 在地图上添加marker，并显示
        marker1 = (Marker) bdMap.addOverlay(options);
    }

    /**
     * 添加多边形覆盖物
     */
    private void addPolygonOptions() {
        bdMap.clear();
        // 定义多边形的五个顶点
        LatLng pt1 = new LatLng(latitude + 0.02, longitude);
        LatLng pt2 = new LatLng(latitude, longitude - 0.03);
        LatLng pt3 = new LatLng(latitude - 0.02, longitude - 0.01);
        LatLng pt4 = new LatLng(latitude - 0.02, longitude + 0.01);
        LatLng pt5 = new LatLng(latitude, longitude + 0.03);
        List<LatLng> points = new ArrayList<LatLng>();
        points.add(pt1);
        points.add(pt2);
        points.add(pt3);
        points.add(pt4);
        points.add(pt5);
        //
        PolygonOptions polygonOptions = new PolygonOptions();
        polygonOptions.points(points);
        polygonOptions.fillColor(0xAAFFFF00);
        polygonOptions.stroke(new Stroke(2, 0xAA00FF00));
        bdMap.addOverlay(polygonOptions);
    }

    /**
     * 添加文字覆盖物
     */
    private void addTextOptions() {
        bdMap.clear();
        LatLng latLng = new LatLng(latitude, longitude);
        TextOptions textOptions = new TextOptions();
        textOptions.bgColor(0xAAFFFF00) // 設置文字覆蓋物背景顏色
                .fontSize(28) // 设置字体大小
                .fontColor(0xFFFF00FF)// 设置字体颜色
                .text("我在这里啊！！！！") // 文字内容
                .rotate(-30) // 设置文字的旋转角度
                .position(latLng);// 设置位置
        bdMap.addOverlay(textOptions);
    }

    /**
     * 添加地形图图层
     */
    private void addGroundOverlayOptions() {
        bdMap.clear();
        LatLng southwest = new LatLng(latitude - 0.01, longitude - 0.012);// 西南
        LatLng northeast = new LatLng(latitude + 0.01, longitude + 0.012);// 东北
        LatLngBounds bounds = new LatLngBounds.Builder().include(southwest)
                .include(northeast).build();// 得到一个地理范围对象

        GroundOverlayOptions groundOverlayOptions = new GroundOverlayOptions();
        groundOverlayOptions.image(bitmap2);// 显示的图片
        groundOverlayOptions.positionFromBounds(bounds);// 显示的位置
        groundOverlayOptions.transparency(0.7f);// 显示的透明度
        bdMap.addOverlay(groundOverlayOptions);
    }

    /**
     * 添加折线覆盖物
     */
    private void addPolylineOptions() {
        bdMap.clear();
        // 点
        LatLng pt1 = new LatLng(latitude + 0.01, longitude);
        LatLng pt2 = new LatLng(latitude, longitude - 0.01);
        LatLng pt3 = new LatLng(latitude - 0.01, longitude - 0.01);
        LatLng pt5 = new LatLng(latitude, longitude + 0.01);
        List<LatLng> points = new ArrayList<LatLng>();
        points.add(pt1);
        points.add(pt2);
        points.add(pt3);
        points.add(pt5);
        //
        PolylineOptions polylineOptions = new PolylineOptions();
        polylineOptions.points(points);
        polylineOptions.color(0xFF000000);
        polylineOptions.width(4);// 折线线宽
        bdMap.addOverlay(polylineOptions);
    }

    /**
     * 添加圆点覆盖物
     */
    private void addDotOptions() {
        bdMap.clear();
        DotOptions dotOptions = new DotOptions();
        dotOptions.center(new LatLng(latitude, longitude));// 设置圆心坐标
        dotOptions.color(0XFFfaa755);// 颜色
        dotOptions.radius(25);// 设置半径
        bdMap.addOverlay(dotOptions);
    }

    /**
     * 添加圆（空心）覆盖物
     */
    private void addCircleOptions() {
        bdMap.clear();
        CircleOptions circleOptions = new CircleOptions();
        circleOptions.center(new LatLng(latitude, longitude));// 设置圆心坐标
        circleOptions.fillColor(0XFFfaa755);// 圆的填充颜色
        circleOptions.radius(150);// 设置半径
        circleOptions.stroke(new Stroke(5, 0xAA00FF00));// 设置边框
        bdMap.addOverlay(circleOptions);
    }

    /**
     * 添加弧线覆盖物
     */
    private void addArcOptions() {
        bdMap.clear();
        LatLng pt1 = new LatLng(latitude, longitude - 0.01);
        LatLng pt2 = new LatLng(latitude - 0.01, longitude - 0.01);
        LatLng pt3 = new LatLng(latitude, longitude + 0.01);
        ArcOptions arcOptions = new ArcOptions();
        arcOptions.points(pt1, pt2, pt3);// 设置弧线的起点、中点、终点坐标
        arcOptions.width(5);// 线宽
        arcOptions.color(0xFF000000);
        bdMap.addOverlay(arcOptions);
    }

    /**
     * 显示弹出窗口覆盖物
     */
    private void displayInfoWindow(final LatLng latLng) {
        // 创建infowindow展示的view
        Button btn = new Button(getApplicationContext());
        btn.setBackgroundResource(R.drawable.popup);
        btn.setText("点我点我~");
        btn.setTextColor(0xAA000000);
        BitmapDescriptor bitmapDescriptor = BitmapDescriptorFactory
                .fromView(btn);
        // infowindow点击事件
        OnInfoWindowClickListener infoWindowClickListener = new OnInfoWindowClickListener() {
            @Override
            public void onInfoWindowClick() {
                reverseGeoCode(latLng);
                // 隐藏InfoWindow
                bdMap.hideInfoWindow();
            }
        };
        // 创建infowindow
        InfoWindow infoWindow = new InfoWindow(bitmapDescriptor, latLng, -47,
                infoWindowClickListener);

        // 显示InfoWindow
        bdMap.showInfoWindow(infoWindow);
    }

    /**
     * 反地理编码得到地址信息
     */
    private void reverseGeoCode(LatLng latLng) {
        // 创建地理编码检索实例
        GeoCoder geoCoder = GeoCoder.newInstance();
        //
        OnGetGeoCoderResultListener listener = new OnGetGeoCoderResultListener() {
            // 反地理编码查询结果回调函数
            @Override
            public void onGetReverseGeoCodeResult(ReverseGeoCodeResult result) {
                if (result == null
                        || result.error != SearchResult.ERRORNO.NO_ERROR) {
                    // 没有检测到结果
                    Toast.makeText(AddOverlayActivity.this, "抱歉，未能找到结果",
                            Toast.LENGTH_LONG).show();
                }
                Toast.makeText(AddOverlayActivity.this,
                        "位置：" + result.getAddress(), Toast.LENGTH_LONG).show();
            }

            // 地理编码查询结果回调函数
            @Override
            public void onGetGeoCodeResult(GeoCodeResult result) {
                if (result == null
                        || result.error != SearchResult.ERRORNO.NO_ERROR) {
                    // 没有检测到结果
                }
            }
        };
        // 设置地理编码检索监听者
        geoCoder.setOnGetGeoCodeResultListener(listener);
        //
        geoCoder.reverseGeoCode(new ReverseGeoCodeOption().location(latLng));
        // 释放地理编码检索实例
        // geoCoder.destroy();
    }

    @Override
    protected void onResume() {
        super.onResume();
        mMapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mMapView.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mMapView.onDestroy();
        // 回收bitmip资源
        bitmap.recycle();
        bitmap2.recycle();
    }
}
```    

### 参考链接 
[百度地图开发（二）之添加覆盖物 + 地理编码和反地理编码 - crazy_jack - 博客频道 - CSDN.NET](http://blog.csdn.net/crazy1235/article/details/43377545)  





### 地图控制实现  

```
package com.zj.mapall;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

import com.baidu.location.BDLocation;
import com.baidu.location.BDLocationListener;
import com.baidu.location.LocationClient;
import com.baidu.location.LocationClientOption;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.BaiduMap.OnMapClickListener;
import com.baidu.mapapi.map.BaiduMap.SnapshotReadyCallback;
import com.baidu.mapapi.map.BitmapDescriptor;
import com.baidu.mapapi.map.MapPoi;
import com.baidu.mapapi.map.MapStatus;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.map.MyLocationConfiguration;
import com.baidu.mapapi.map.MyLocationConfiguration.LocationMode;
import com.baidu.mapapi.map.MyLocationData;
import com.baidu.mapapi.model.LatLng;

import android.app.Activity;
import android.graphics.Bitmap;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.Toast;

/**
 * 地图控制demo（点击、双击、长按、缩放、旋转、俯视） + 定位
 * 
 * @author ys
 *
 */
public class MapControllActivity extends Activity implements OnClickListener {
    // 地图控件对象
    private MapView mapView;
    // 百度地图对象
    private BaiduMap bdMap;
    // 经纬度
    private double latitude, longitude;
    // 缩小
    private Button zoomOutBtn;
    // 放大
    private Button zoomInBtn;
    // 旋转
    private Button rotateBtn;
    // 俯视
    private Button overlookBtn;
    // 截图
    private Button screenShotBtn;

    // 标记是否已经放大到最大或者缩小到最小级别
    private boolean isMaxOrMin = false;

    private float maxZoom = 0.0f;
    private float minZoom = 0.0f;
    // 记录当前地图的缩放级别
    private float currentZoom = 0.0f;
    // 描述地图状态将要发生的状态
    private MapStatusUpdate msu;
    // 用于生成地图将要发生的变化
    private MapStatusUpdateFactory msuFactory;
    // 定义地图状态
    private MapStatus mapStatus;

    // 旋转角度
    private float rotateAngle = 0.0f;
    // 俯视角度 （0 ~ -45°）
    private float overlookAngle = 0.0f;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_map_controll);
        init();
    }

    /**
     * 
     */
    private void init() {
        mapView = (MapView) findViewById(R.id.bd_mapview);
        bdMap = mapView.getMap();

        mapView.showZoomControls(false);// 不显示默认的缩放控件
        mapView.showScaleControl(false);// 不显示默认比例尺控件

        maxZoom = bdMap.getMaxZoomLevel();// 获得地图的最大缩放级别
        minZoom = bdMap.getMinZoomLevel();// 获得地图的最小缩放级别

        zoomInBtn = (Button) findViewById(R.id.zoom_in_btn);
        zoomOutBtn = (Button) findViewById(R.id.zoom_out_btn);
        rotateBtn = (Button) findViewById(R.id.rotate_btn);
        overlookBtn = (Button) findViewById(R.id.overlook_btn);
        screenShotBtn = (Button) findViewById(R.id.screen_shot_btn);

        zoomInBtn.setOnClickListener(this);
        zoomOutBtn.setOnClickListener(this);
        rotateBtn.setOnClickListener(this);
        overlookBtn.setOnClickListener(this);
        screenShotBtn.setOnClickListener(this);

        bdMap.setOnMapClickListener(new OnMapClickListener() {
            @Override
            public boolean onMapPoiClick(MapPoi arg0) {
                return false;
            }

            @Override
            public void onMapClick(LatLng arg0) {
                // 设置地图新中心点
                msu = msuFactory.newLatLng(arg0);
                bdMap.animateMapStatus(msu);
                Toast.makeText(MapControllActivity.this,
                        "地图中心点移动到：" + arg0.toString(), Toast.LENGTH_SHORT)
                        .show();
            }
        });

    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.zoom_out_btn:// 缩小
            msu = msuFactory.zoomOut();
            bdMap.animateMapStatus(msu);
            currentZoom = bdMap.getMapStatus().zoom;
            Toast.makeText(MapControllActivity.this,
                    "当前地图的缩放级别是：" + currentZoom, Toast.LENGTH_SHORT).show();
            break;
        case R.id.zoom_in_btn:// 放大
            msu = msuFactory.zoomIn();
            bdMap.animateMapStatus(msu);
            currentZoom = bdMap.getMapStatus().zoom;
            Toast.makeText(MapControllActivity.this,
                    "当前地图的缩放级别是：" + currentZoom, Toast.LENGTH_SHORT).show();
            break;
        case R.id.rotate_btn:// 旋转
            mapStatus = new MapStatus.Builder(bdMap.getMapStatus()).rotate(
                    rotateAngle += 30).build();
            msu = msuFactory.newMapStatus(mapStatus);
            bdMap.animateMapStatus(msu);
            break;
        case R.id.overlook_btn:// 俯视
            mapStatus = new MapStatus.Builder(bdMap.getMapStatus()).overlook(
                    overlookAngle -= 10).build();
            msu = msuFactory.newMapStatus(mapStatus);
            bdMap.animateMapStatus(msu);
            break;
        case R.id.screen_shot_btn:// 截图
            bdMap.snapshot(new SnapshotReadyCallback() {
                @Override
                public void onSnapshotReady(Bitmap bitmap) {
                    File file = new File("/mnt/sdcard/test.png");
                    FileOutputStream out;
                    try {
                        out = new FileOutputStream(file);
                        if (bitmap
                                .compress(Bitmap.CompressFormat.PNG, 100, out)) {
                            out.flush();
                            out.close();
                        }
                        Toast.makeText(MapControllActivity.this,
                                "屏幕截图成功，图片存在: " + file.toString(),
                                Toast.LENGTH_SHORT).show();
                    } catch (FileNotFoundException e) {
                        e.printStackTrace();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            });
            break;
        default:
            break;
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        mapView.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();
        mapView.onResume();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mapView.onDestroy();
    }

}
```

### 定位实现  

```
package com.zj.mapall;

import com.baidu.location.BDLocation;
import com.baidu.location.BDLocationListener;
import com.baidu.location.BDNotifyListener;
import com.baidu.location.LocationClient;
import com.baidu.location.LocationClientOption;
import com.baidu.location.LocationClientOption.LocationMode;
import com.baidu.mapapi.map.MyLocationConfiguration;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.BitmapDescriptor;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.map.MyLocationData;
import com.baidu.mapapi.model.LatLng;

import android.app.Activity;
import android.app.Service;
import android.os.Bundle;
import android.os.Vibrator;
import android.util.Log;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.Toast;

/**
 * 定位
 * 
 * @author ys
 *
 */
public class LocationActivity extends Activity implements OnClickListener {

    private MapView mapview;
    private BaiduMap bdMap;

    private LocationClient locationClient;
    private BDLocationListener locationListener;
    private BDNotifyListener notifyListener;

    private double longitude;// 精度
    private double latitude;// 维度
    private float radius;// 定位精度半径，单位是米
    private String addrStr;// 反地理编码
    private String province;// 省份信息
    private String city;// 城市信息
    private String district;// 区县信息
    private float direction;// 手机方向信息

    private int locType;

    // 定位按钮
    private Button locateBtn;
    // 定位模式 （普通-跟随-罗盘）
    private MyLocationConfiguration.LocationMode currentMode;
    // 定位图标描述
    private BitmapDescriptor currentMarker = null;
    // 记录是否第一次定位
    private boolean isFirstLoc = true;
    
    //振动器设备
    private Vibrator mVibrator;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_location);

        mapview = (MapView) findViewById(R.id.bd_mapview);
        bdMap = mapview.getMap();
        locateBtn = (Button) findViewById(R.id.locate_btn);
        locateBtn.setOnClickListener(this);
        currentMode = MyLocationConfiguration.LocationMode.NORMAL;
        locateBtn.setText("普通");
        mVibrator =(Vibrator)getApplicationContext().getSystemService(Service.VIBRATOR_SERVICE);
        init();
    }

    /**
     * 
     */
    private void init() {
        bdMap.setMyLocationEnabled(true);
        // 1. 初始化LocationClient类
        locationClient = new LocationClient(getApplicationContext());
        // 2. 声明LocationListener类
        locationListener = new MyLocationListener();
        // 3. 注册监听函数
        locationClient.registerLocationListener(locationListener);
        // 4. 设置参数
        LocationClientOption locOption = new LocationClientOption();
        locOption.setLocationMode(LocationMode.Hight_Accuracy);// 设置定位模式
        locOption.setCoorType("bd09ll");// 设置定位结果类型
        locOption.setScanSpan(20000);// 设置发起定位请求的间隔时间,ms
        locOption.setIsNeedAddress(true);// 返回的定位结果包含地址信息
        locOption.setNeedDeviceDirect(true);// 设置返回结果包含手机的方向

        locationClient.setLocOption(locOption);
        // 5. 注册位置提醒监听事件
        notifyListener = new MyNotifyListener();
        notifyListener.SetNotifyLocation(longitude, latitude, 3000, "bd09ll");//精度，维度，范围，坐标类型
        locationClient.registerNotify(notifyListener);
        // 6. 开启/关闭 定位SDK
        locationClient.start();
        // locationClient.stop();
        // 发起定位，异步获取当前位置，因为是异步的，所以立即返回，不会引起阻塞
        // 定位的结果在ReceiveListener的方法onReceive方法的参数中返回。
        // 当定位SDK从定位依据判定，位置和上一次没发生变化，而且上一次定位结果可用时，则不会发生网络请求，而是返回上一次的定位结果。
        // 返回值，0：正常发起了定位 1：service没有启动 2：没有监听函数
        // 6：两次请求时间太短（前后两次请求定位时间间隔不能小于1000ms）
        /*
         * if (locationClient != null && locationClient.isStarted()) {
         * requestResult = locationClient.requestLocation(); } else {
         * Log.d("LocSDK5", "locClient is null or not started"); }
         */

    }

    /**
     * 
     * @author ys
     *
     */
    class MyLocationListener implements BDLocationListener {
        // 异步返回的定位结果
        @Override
        public void onReceiveLocation(BDLocation location) {
            if (location == null) {
                return;
            }
            locType = location.getLocType();
            Toast.makeText(LocationActivity.this, "当前定位的返回值是："+locType, Toast.LENGTH_SHORT).show();
            longitude = location.getLongitude();
            latitude = location.getLatitude();
            if (location.hasRadius()) {// 判断是否有定位精度半径
                radius = location.getRadius();
            }
            if (locType == BDLocation.TypeGpsLocation) {//
                Toast.makeText(
                        LocationActivity.this,
                        "当前速度是：" + location.getSpeed() + "~~定位使用卫星数量："
                                + location.getSatelliteNumber(),
                        Toast.LENGTH_SHORT).show();
            } else if (locType == BDLocation.TypeNetWorkLocation) {
                addrStr = location.getAddrStr();// 获取反地理编码(文字描述的地址)
                Toast.makeText(LocationActivity.this, addrStr,
                        Toast.LENGTH_SHORT).show();
            }
            direction = location.getDirection();// 获取手机方向，【0~360°】,手机上面正面朝北为0°
            province = location.getProvince();// 省份
            city = location.getCity();// 城市
            district = location.getDistrict();// 区县
            Toast.makeText(LocationActivity.this,
                    province + "~" + city + "~" + district, Toast.LENGTH_SHORT)
                    .show();
            // 构造定位数据
            MyLocationData locData = new MyLocationData.Builder()
                    .accuracy(radius)//
                    .direction(direction)// 方向
                    .latitude(latitude)//
                    .longitude(longitude)//
                    .build();
            // 设置定位数据
            bdMap.setMyLocationData(locData);
            LatLng ll = new LatLng(latitude, longitude);
            MapStatusUpdate msu = MapStatusUpdateFactory.newLatLng(ll);
            bdMap.animateMapStatus(msu);

        }
    }

    /**
     * 位置提醒监听器
     * @author ys
     *
     */
    class MyNotifyListener extends BDNotifyListener {
        @Override
        public void onNotify(BDLocation bdLocation, float distance) {
            super.onNotify(bdLocation, distance);
            mVibrator.vibrate(1000);//振动提醒已到设定位置附近
            Toast.makeText(LocationActivity.this, "震动提醒", Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.locate_btn:// 定位
            switch (currentMode) {
            case NORMAL:
                locateBtn.setText("跟随");
                currentMode = MyLocationConfiguration.LocationMode.FOLLOWING;
                break;
            case FOLLOWING:
                locateBtn.setText("罗盘");
                currentMode = MyLocationConfiguration.LocationMode.COMPASS;
                break;
            case COMPASS:
                locateBtn.setText("普通");
                currentMode = MyLocationConfiguration.LocationMode.NORMAL;
                break;
            }
            bdMap.setMyLocationConfigeration(new MyLocationConfiguration(
                    currentMode, true, currentMarker));
            break;
        }
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        mapview.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mapview.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mapview.onDestroy();
        locationClient.unRegisterLocationListener(locationListener);
        //取消位置提醒
        locationClient.removeNotifyEvent(notifyListener);
        locationClient.stop();
    }
}
```   


### 参考链接
[百度地图开发（三）之地图控制 + 定位 - crazy_jack - 博客频道 - CSDN.NET](http://blog.csdn.net/crazy1235/article/details/43898451)

### 效果如下 
![这里写图片描述](http://img.blog.csdn.net/20160419201837364)

### POI检索实现  

```
package com.zj.mapall;

import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.model.LatLng;
import com.baidu.mapapi.model.LatLngBounds;
import com.baidu.mapapi.overlayutil.PoiOverlay;

import com.baidu.mapapi.search.core.PoiInfo;
import com.baidu.mapapi.search.core.SearchResult;
import com.baidu.mapapi.search.poi.OnGetPoiSearchResultListener;
import com.baidu.mapapi.search.poi.PoiBoundSearchOption;
import com.baidu.mapapi.search.poi.PoiCitySearchOption;
import com.baidu.mapapi.search.poi.PoiDetailResult;
import com.baidu.mapapi.search.poi.PoiDetailSearchOption;
import com.baidu.mapapi.search.poi.PoiNearbySearchOption;
import com.baidu.mapapi.search.poi.PoiResult;
import com.baidu.mapapi.search.poi.PoiSearch;
import com.baidu.mapapi.search.share.OnGetShareUrlResultListener;
import com.baidu.mapapi.search.share.ShareUrlResult;
import com.baidu.mapapi.search.share.ShareUrlSearch;

import android.app.Activity;
import android.os.Bundle;
import android.text.Editable;
import android.text.TextWatcher;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

/**
 * POI检索 1.周边检索 2. 范围检索 3. 城市检索 4.详细检索
 * 
 * @author ys
 *
 */
public class PoiSearchActivity extends Activity implements OnClickListener {

    private MapView mapView;
    private BaiduMap bdMap;
    //
    private PoiSearch poiSearch;
    
    private ShareUrlSearch shareUrlSearch;

    private EditText editCityEt, editSearchKeyEt;

    // 城市检索，区域检索，周边检索，下一组数据 按钮
    private Button citySearchBtn, boundSearchBtn, nearbySearchBtn, nextDataBtn;

    // 记录检索类型
    private int type;
    // 记录页标
    private int page = 1;
    private int totalPage = 0;

    private double latitude = 39.9361752;
    private double longitude = 116.400244;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_poi_search);
        init();
    }

    private void init() {
        mapView = (MapView) findViewById(R.id.mapview);
        mapView.showZoomControls(false);
        bdMap = mapView.getMap();
        // 实例化PoiSearch对象
        poiSearch = PoiSearch.newInstance();
        // 设置检索监听器
        poiSearch.setOnGetPoiSearchResultListener(poiSearchListener);
        editCityEt = (EditText) findViewById(R.id.city);
        editSearchKeyEt = (EditText) findViewById(R.id.searchkey);

        citySearchBtn = (Button) findViewById(R.id.city_search_btn);
        boundSearchBtn = (Button) findViewById(R.id.bound_search_btn);
        nearbySearchBtn = (Button) findViewById(R.id.nearby_search_btn);
        nextDataBtn = (Button) findViewById(R.id.next_data_btn);
        nextDataBtn.setEnabled(false);
        citySearchBtn.setOnClickListener(this);
        boundSearchBtn.setOnClickListener(this);
        nearbySearchBtn.setOnClickListener(this);
        nextDataBtn.setOnClickListener(this);

        editSearchKeyEt.addTextChangedListener(new TextWatcher() {

            @Override
            public void onTextChanged(CharSequence s, int start, int before,
                    int count) {
                citySearchBtn.setEnabled(true);
                boundSearchBtn.setEnabled(true);
                nearbySearchBtn.setEnabled(true);
                nextDataBtn.setEnabled(false);
                page = 1;
                totalPage = 0;
            }

            @Override
            public void beforeTextChanged(CharSequence s, int start, int count,
                    int after) {

            }

            @Override
            public void afterTextChanged(Editable s) {

            }
        });
        
        shareUrlSearch = ShareUrlSearch.newInstance();

    }

    /**
     * 
     */
    OnGetPoiSearchResultListener poiSearchListener = new OnGetPoiSearchResultListener() {
        @Override
        public void onGetPoiResult(PoiResult poiResult) {
            if (poiResult == null
                    || poiResult.error == SearchResult.ERRORNO.RESULT_NOT_FOUND) {// 没有找到检索结果
                Toast.makeText(PoiSearchActivity.this, "未找到结果",
                        Toast.LENGTH_LONG).show();
                return;
            }

            if (poiResult.error == SearchResult.ERRORNO.NO_ERROR) {// 检索结果正常返回
                bdMap.clear();
                MyPoiOverlay poiOverlay = new MyPoiOverlay(bdMap);
                poiOverlay.setData(poiResult);// 设置POI数据
                bdMap.setOnMarkerClickListener(poiOverlay);
                poiOverlay.addToMap();// 将所有的overlay添加到地图上
                poiOverlay.zoomToSpan();
                //
                totalPage = poiResult.getTotalPageNum();// 获取总分页数
                Toast.makeText(
                        PoiSearchActivity.this,
                        "总共查到" + poiResult.getTotalPoiNum() + "个兴趣点, 分为"
                                + totalPage + "页", Toast.LENGTH_SHORT).show();

            }
        }

        @Override
        public void onGetPoiDetailResult(PoiDetailResult poiDetailResult) {
            if (poiDetailResult.error != SearchResult.ERRORNO.NO_ERROR) {
                Toast.makeText(PoiSearchActivity.this, "抱歉，未找到结果",
                        Toast.LENGTH_SHORT).show();
            } else {// 正常返回结果的时候，此处可以获得很多相关信息
                Toast.makeText(
                        PoiSearchActivity.this,
                        poiDetailResult.getName() + ": "
                                + poiDetailResult.getAddress(),
                        Toast.LENGTH_LONG).show();
            }
        }
    };
    
    /**
     * 短串检索监听器
     */
    OnGetShareUrlResultListener shareUrlResultListener = new OnGetShareUrlResultListener() {
        
        @Override
        public void onGetPoiDetailShareUrlResult(ShareUrlResult arg0) {
            
        }
        
        @Override
        public void onGetLocationShareUrlResult(ShareUrlResult arg0) {
            
        }

        @Override
        public void onGetRouteShareUrlResult(ShareUrlResult arg0) {
            // TODO Auto-generated method stub
            
        }
    };

    class MyPoiOverlay extends PoiOverlay {

        public MyPoiOverlay(BaiduMap arg0) {
            super(arg0);
        }

        @Override
        public boolean onPoiClick(int arg0) {
            super.onPoiClick(arg0);
            PoiInfo poiInfo = getPoiResult().getAllPoi().get(arg0);
            poiSearch.searchPoiDetail(new PoiDetailSearchOption()
                    .poiUid(poiInfo.uid));
            return true;
        }

    }

    /**
     * 城市内搜索
     */
    private void citySearch(int page) {
        // 设置检索参数
        PoiCitySearchOption citySearchOption = new PoiCitySearchOption();
        citySearchOption.city(editCityEt.getText().toString());// 城市
        citySearchOption.keyword(editSearchKeyEt.getText().toString());// 关键字
        citySearchOption.pageCapacity(15);// 默认每页10条
        citySearchOption.pageNum(page);// 分页编号
        // 发起检索请求
        poiSearch.searchInCity(citySearchOption);
    }

    /**
     * 范围检索
     */
    private void boundSearch(int page) {
        PoiBoundSearchOption boundSearchOption = new PoiBoundSearchOption();
        LatLng southwest = new LatLng(latitude - 0.01, longitude - 0.012);// 西南
        LatLng northeast = new LatLng(latitude + 0.01, longitude + 0.012);// 东北
        LatLngBounds bounds = new LatLngBounds.Builder().include(southwest)
                .include(northeast).build();// 得到一个地理范围对象
        boundSearchOption.bound(bounds);// 设置poi检索范围
        boundSearchOption.keyword(editSearchKeyEt.getText().toString());// 检索关键字
        boundSearchOption.pageNum(page);
        poiSearch.searchInBound(boundSearchOption);// 发起poi范围检索请求
    }

    /**
     * 附近检索
     */
    private void nearbySearch(int page) {
        PoiNearbySearchOption nearbySearchOption = new PoiNearbySearchOption();
        nearbySearchOption.location(new LatLng(latitude, longitude));
        nearbySearchOption.keyword(editSearchKeyEt.getText().toString());
        nearbySearchOption.radius(1000);// 检索半径，单位是米
        nearbySearchOption.pageNum(page);
        poiSearch.searchNearby(nearbySearchOption);// 发起附近检索请求
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.city_search_btn:
            type = 0;
            page = 1;
            citySearchBtn.setEnabled(false);
            boundSearchBtn.setEnabled(true);
            nearbySearchBtn.setEnabled(true);
            nextDataBtn.setEnabled(true);
            bdMap.clear();
            citySearch(page);
            break;
        case R.id.bound_search_btn:
            type = 1;
            page = 1;
            citySearchBtn.setEnabled(true);
            boundSearchBtn.setEnabled(false);
            nearbySearchBtn.setEnabled(true);
            nextDataBtn.setEnabled(true);
            bdMap.clear();
            boundSearch(page);
            break;
        case R.id.nearby_search_btn:
            type = 2;
            page = 1;
            citySearchBtn.setEnabled(true);
            boundSearchBtn.setEnabled(true);
            nearbySearchBtn.setEnabled(false);
            nextDataBtn.setEnabled(true);
            bdMap.clear();
            nearbySearch(page);
            break;
        case R.id.next_data_btn:
            switch (type) {
            case 0:
                if (++page <= totalPage) {
                    citySearch(page);
                } else {
                    Toast.makeText(PoiSearchActivity.this, "已经查到了最后一页~",
                            Toast.LENGTH_SHORT).show();
                }
                break;
            case 1:
                if (++page <= totalPage) {
                    boundSearch(page);
                } else {
                    Toast.makeText(PoiSearchActivity.this, "已经查到了最后一页~",
                            Toast.LENGTH_SHORT).show();
                }
                break;
            case 2:
                if (++page <= totalPage) {
                    nearbySearch(page);
                } else {
                    Toast.makeText(PoiSearchActivity.this, "已经查到了最后一页~",
                            Toast.LENGTH_SHORT).show();
                }
                break;
            }
            break;
        default:
            break;
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        mapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mapView.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        poiSearch.destroy();// 释放poi检索对象
        mapView.onDestroy();
    }

}
```   

### 注意，在这里需要用到PoiOverlay，最新的jar包中已经没有这个类了，但已经开源，可以直接下载源代码加入

### 参考链接
[现在检索接口中没有PoiOverlay类了吗 - Android地图SDK - 百度地图开放平台 - Powered by Discuz!](http://bbs.lbsyun.baidu.com/forum.php?mod=viewthread&tid=104798)

[百度地图开发（四）之POI检索 - crazy_jack - 博客频道 - CSDN.NET](http://blog.csdn.net/crazy1235/article/details/44002459)

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160419202137852)


### 公交线路查询实现  

```
package com.zj.mapall;

import java.util.ArrayList;
import java.util.List;

import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.map.MarkerOptions;
import com.baidu.mapapi.map.OverlayOptions;
import com.baidu.mapapi.overlayutil.BusLineOverlay;
import com.baidu.mapapi.search.busline.BusLineResult;
import com.baidu.mapapi.search.busline.BusLineSearch;
import com.baidu.mapapi.search.busline.BusLineSearchOption;
import com.baidu.mapapi.search.busline.OnGetBusLineSearchResultListener;
import com.baidu.mapapi.search.core.PoiInfo;
import com.baidu.mapapi.search.core.SearchResult;
import com.baidu.mapapi.search.poi.OnGetPoiSearchResultListener;
import com.baidu.mapapi.search.poi.PoiCitySearchOption;
import com.baidu.mapapi.search.poi.PoiDetailResult;
import com.baidu.mapapi.search.poi.PoiResult;
import com.baidu.mapapi.search.poi.PoiSearch;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

/**
 * 公交线路查询
 * 
 * @author ys
 *
 */
public class BusLineSearchActivity extends Activity implements OnClickListener {

    private EditText cityEt;
    private EditText buslineEt;
    private Button searchBtn;
    private Button nextlineBtn;

    private MapView mapView;
    private BaiduMap bdMap;

    private String city;// 城市
    private String busline;// 公交路线
    private List<String> buslineIdList;// 存储公交线路的uid
    private int buslineIndex = 0;// 标记第几个路线

    private PoiSearch poiSearch;
    private BusLineSearch busLineSearch;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_busline_search);
        init();
    }

    /**
     * 初始化操作
     */
    private void init() {
        mapView = (MapView) findViewById(R.id.mapview);
        bdMap = mapView.getMap();

        cityEt = (EditText) findViewById(R.id.city_et);
        buslineEt = (EditText) findViewById(R.id.searchkey_et);
        searchBtn = (Button) findViewById(R.id.busline_search_btn);
        nextlineBtn = (Button) findViewById(R.id.nextline_btn);
        searchBtn.setOnClickListener(this);
        nextlineBtn.setOnClickListener(this);

        buslineIdList = new ArrayList<String>();

        poiSearch = PoiSearch.newInstance();
        poiSearch.setOnGetPoiSearchResultListener(poiSearchResultListener);

        busLineSearch = BusLineSearch.newInstance();
        busLineSearch
                .setOnGetBusLineSearchResultListener(busLineSearchResultListener);

    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.busline_search_btn:
            city = cityEt.getText().toString();
            busline = buslineEt.getText().toString();
            poiSearch.searchInCity((new PoiCitySearchOption()).city(city)
                    .keyword(busline));
            break;
        case R.id.nextline_btn:
            searchBusline();
            break;
        }
    }

    private void searchBusline() {
        if (buslineIndex >= buslineIdList.size()) {
            buslineIndex = 0;
        }
        if (buslineIndex >= 0 && buslineIndex < buslineIdList.size()
                && buslineIdList.size() > 0) {
            boolean flag = busLineSearch
                    .searchBusLine((new BusLineSearchOption().city(city)
                            .uid(buslineIdList.get(buslineIndex))));
            if (flag) {
                Toast.makeText(BusLineSearchActivity.this, "检索成功~", 1000)
                        .show();
            } else {
                Toast.makeText(BusLineSearchActivity.this, "检索失败~", 1000)
                        .show();
            }
            buslineIndex++;
        }
    }

    /**
     * POI检索结果监听器
     */
    OnGetPoiSearchResultListener poiSearchResultListener = new OnGetPoiSearchResultListener() {
        @Override
        public void onGetPoiResult(PoiResult poiResult) {
            if (poiResult == null
                    || poiResult.error == SearchResult.ERRORNO.RESULT_NOT_FOUND) {// 没有找到检索结果
                Toast.makeText(BusLineSearchActivity.this, "未找到结果",
                        Toast.LENGTH_LONG).show();
                return;
            }

            if (poiResult.error == SearchResult.ERRORNO.NO_ERROR) {// 检索结果正常返回
                // 遍历所有poi，找到类型为公交线路的poi
                buslineIdList.clear();
                for (PoiInfo poi : poiResult.getAllPoi()) {
                    if (poi.type == PoiInfo.POITYPE.BUS_LINE
                            || poi.type == PoiInfo.POITYPE.SUBWAY_LINE) {
                        buslineIdList.add(poi.uid);
                    }
                }
                searchBusline();
            }
        }

        @Override
        public void onGetPoiDetailResult(PoiDetailResult arg0) {

        }
    };

    /**
     * 公交信息查询结果监听器
     */
    OnGetBusLineSearchResultListener busLineSearchResultListener = new OnGetBusLineSearchResultListener() {

        @Override
        public void onGetBusLineResult(BusLineResult busLineResult) {
            if (busLineResult.error != SearchResult.ERRORNO.NO_ERROR) {
                Toast.makeText(BusLineSearchActivity.this, "抱歉，未找到结果",
                        Toast.LENGTH_SHORT).show();
            } else {
                bdMap.clear();
                BusLineOverlay overlay = new MyBuslineOverlay(bdMap);// 用于显示一条公交详情结果的Overlay
                overlay.setData(busLineResult);
                overlay.addToMap();// 将overlay添加到地图上
                overlay.zoomToSpan();// 缩放地图，使所有overlay都在合适的视野范围内
                bdMap.setOnMarkerClickListener(overlay);
                // 公交线路名称
                Toast.makeText(BusLineSearchActivity.this,
                        busLineResult.getBusLineName(), Toast.LENGTH_SHORT)
                        .show();
            }
        }
    };

    class MyBuslineOverlay extends BusLineOverlay {

        public MyBuslineOverlay(BaiduMap arg0) {
            super(arg0);
        }

        /**
         * 站点点击事件
         */
        @Override
        public boolean onBusStationClick(int arg0) {
            MarkerOptions options = (MarkerOptions) getOverlayOptions().get(arg0);
            bdMap.animateMapStatus(MapStatusUpdateFactory.newLatLng(options.getPosition()));
            return true;
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        mapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mapView.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        poiSearch.destroy();// 释放检索对象资源
        busLineSearch.destroy();// 释放检索对象资源
        mapView.onDestroy();
    }

}
```   

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160419202326245)


### 路径规划实现  

```
package com.zj.mapall;

import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.overlayutil.DrivingRouteOverlay;
import com.baidu.mapapi.overlayutil.TransitRouteOverlay;
import com.baidu.mapapi.overlayutil.WalkingRouteOverlay;
import com.baidu.mapapi.search.core.SearchResult;
import com.baidu.mapapi.search.route.BikingRouteResult;
import com.baidu.mapapi.search.route.DrivingRoutePlanOption;
import com.baidu.mapapi.search.route.DrivingRoutePlanOption.DrivingPolicy;
import com.baidu.mapapi.search.route.DrivingRouteResult;
import com.baidu.mapapi.search.route.OnGetRoutePlanResultListener;
import com.baidu.mapapi.search.route.PlanNode;
import com.baidu.mapapi.search.route.RoutePlanSearch;
import com.baidu.mapapi.search.route.TransitRoutePlanOption;
import com.baidu.mapapi.search.route.TransitRoutePlanOption.TransitPolicy;
import com.baidu.mapapi.search.route.TransitRouteResult;
import com.baidu.mapapi.search.route.WalkingRoutePlanOption;
import com.baidu.mapapi.search.route.WalkingRouteResult;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.AdapterView.OnItemSelectedListener;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Spinner;
import android.widget.Toast;

/**
 * 路线规划
 * 
 * @author ys
 * 
 */
public class RoutePlanningActivity extends Activity implements OnClickListener {

    private MapView mapView;
    private BaiduMap bdMap;

    private EditText startEt;
    private EditText endEt;

    private String startPlace;// 开始地点
    private String endPlace;// 结束地点

    private Button driveBtn;// 驾车
    private Button walkBtn;// 步行
    private Button transitBtn;// 换成 （公交）
    private Button nextLineBtn;

    private Spinner drivingSpinner, transitSpinner;

    private RoutePlanSearch routePlanSearch;// 路径规划搜索接口

    private int index = -1;
    private int totalLine = 0;// 记录某种搜索出的方案数量
    private int drivintResultIndex = 0;// 驾车路线方案index
    private int transitResultIndex = 0;// 换乘路线方案index

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_route_planning);
        init();
    }

    /**
     * 
     */
    private void init() {
        mapView = (MapView) findViewById(R.id.mapview);
        mapView.showZoomControls(false);
        bdMap = mapView.getMap();

        startEt = (EditText) findViewById(R.id.start_et);
        endEt = (EditText) findViewById(R.id.end_et);
        driveBtn = (Button) findViewById(R.id.drive_btn);
        transitBtn = (Button) findViewById(R.id.transit_btn);
        walkBtn = (Button) findViewById(R.id.walk_btn);
        nextLineBtn = (Button) findViewById(R.id.nextline_btn);
        nextLineBtn.setEnabled(false);
        driveBtn.setOnClickListener(this);
        transitBtn.setOnClickListener(this);
        walkBtn.setOnClickListener(this);
        nextLineBtn.setOnClickListener(this);

        drivingSpinner = (Spinner) findViewById(R.id.driving_spinner);
        String[] drivingItems = getResources().getStringArray(
                R.array.driving_spinner);
        ArrayAdapter<String> drivingAdapter = new ArrayAdapter<>(this,
                android.R.layout.simple_spinner_item, drivingItems);
        drivingSpinner.setAdapter(drivingAdapter);
        drivingSpinner.setOnItemSelectedListener(new OnItemSelectedListener() {

            @Override
            public void onItemSelected(AdapterView<?> parent, View view,
                    int position, long id) {
                if (index == 0) {
                    drivintResultIndex = 0;
                    drivingSearch(drivintResultIndex);
                }
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {
            }
        });

        transitSpinner = (Spinner) findViewById(R.id.transit_spinner);
        String[] transitItems = getResources().getStringArray(
                R.array.transit_spinner);
        ArrayAdapter<String> transitAdapter = new ArrayAdapter<>(this,
                android.R.layout.simple_spinner_item, transitItems);
        transitSpinner.setAdapter(transitAdapter);
        transitSpinner.setOnItemSelectedListener(new OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view,
                    int position, long id) {
                if (index == 1) {
                    transitResultIndex = 0;
                    transitSearch(transitResultIndex);
                }
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {

            }
        });

        routePlanSearch = RoutePlanSearch.newInstance();
        routePlanSearch
                .setOnGetRoutePlanResultListener(routePlanResultListener);
    }

    /**
     * 路线规划结果回调
     */
    OnGetRoutePlanResultListener routePlanResultListener = new OnGetRoutePlanResultListener() {

        /**
         * 步行路线结果回调
         */
        @Override
        public void onGetWalkingRouteResult(
                WalkingRouteResult walkingRouteResult) {
            bdMap.clear();
            if (walkingRouteResult == null
                    || walkingRouteResult.error != SearchResult.ERRORNO.NO_ERROR) {
                Toast.makeText(RoutePlanningActivity.this, "抱歉，未找到结果",
                        Toast.LENGTH_SHORT).show();
            }
            if (walkingRouteResult.error == SearchResult.ERRORNO.AMBIGUOUS_ROURE_ADDR) {
                // TODO
                return;
            }
            if (walkingRouteResult.error == SearchResult.ERRORNO.NO_ERROR) {
                WalkingRouteOverlay walkingRouteOverlay = new WalkingRouteOverlay(
                        bdMap);
                walkingRouteOverlay.setData(walkingRouteResult.getRouteLines()
                        .get(drivintResultIndex));
                bdMap.setOnMarkerClickListener(walkingRouteOverlay);
                walkingRouteOverlay.addToMap();
                walkingRouteOverlay.zoomToSpan();
                totalLine = walkingRouteResult.getRouteLines().size();
                Toast.makeText(RoutePlanningActivity.this,
                        "共查询出" + totalLine + "条符合条件的线路", 1000).show();
                if (totalLine > 1) {
                    nextLineBtn.setEnabled(true);
                }
            }
        }

        /**
         * 换成路线结果回调
         */
        @Override
        public void onGetTransitRouteResult(
                TransitRouteResult transitRouteResult) {
            bdMap.clear();
            if (transitRouteResult == null
                    || transitRouteResult.error != SearchResult.ERRORNO.NO_ERROR) {
                Toast.makeText(RoutePlanningActivity.this, "抱歉，未找到结果",
                        Toast.LENGTH_SHORT).show();
            }
            if (transitRouteResult.error == SearchResult.ERRORNO.AMBIGUOUS_ROURE_ADDR) {
                // 起终点或途经点地址有岐义，通过以下接口获取建议查询信息
                // drivingRouteResult.getSuggestAddrInfo()
                return;
            }
            if (transitRouteResult.error == SearchResult.ERRORNO.NO_ERROR) {
                TransitRouteOverlay transitRouteOverlay = new TransitRouteOverlay(
                        bdMap);
                transitRouteOverlay.setData(transitRouteResult.getRouteLines()
                        .get(drivintResultIndex));// 设置一条驾车路线方案
                bdMap.setOnMarkerClickListener(transitRouteOverlay);
                transitRouteOverlay.addToMap();
                transitRouteOverlay.zoomToSpan();
                totalLine = transitRouteResult.getRouteLines().size();
                Toast.makeText(RoutePlanningActivity.this,
                        "共查询出" + totalLine + "条符合条件的线路", 1000).show();
                if (totalLine > 1) {
                    nextLineBtn.setEnabled(true);
                }
                // 通过getTaxiInfo()可以得到很多关于打车的信息
                Toast.makeText(
                        RoutePlanningActivity.this,
                        "该路线打车总路程"
                                + transitRouteResult.getTaxiInfo()
                                        .getDistance(), 1000).show();
            }
        }

        /**
         * 驾车路线结果回调 查询的结果可能包括多条驾车路线方案
         */
        @Override
        public void onGetDrivingRouteResult(
                DrivingRouteResult drivingRouteResult) {
            bdMap.clear();
            Log.i("test", "test1");
            if (drivingRouteResult == null
                    || drivingRouteResult.error != SearchResult.ERRORNO.NO_ERROR) {
                Toast.makeText(RoutePlanningActivity.this, "抱歉，未找到结果",
                        Toast.LENGTH_SHORT).show();
            }
            Log.i("test", "test2");
            if (drivingRouteResult.error == SearchResult.ERRORNO.AMBIGUOUS_ROURE_ADDR) {
                // 起终点或途经点地址有岐义，通过以下接口获取建议查询信息
                // drivingRouteResult.getSuggestAddrInfo()
                return;
            }
            if (drivingRouteResult.error == SearchResult.ERRORNO.NO_ERROR) {
                DrivingRouteOverlay drivingRouteOverlay = new DrivingRouteOverlay(
                        bdMap);
                Log.i("test", "test3");
                drivingRouteOverlay.setData(drivingRouteResult.getRouteLines()
                        .get(drivintResultIndex));// 设置一条驾车路线方案
                bdMap.setOnMarkerClickListener(drivingRouteOverlay);
                drivingRouteOverlay.addToMap();
                drivingRouteOverlay.zoomToSpan();
                totalLine = drivingRouteResult.getRouteLines().size();
                Toast.makeText(RoutePlanningActivity.this,
                        "共查询出" + totalLine + "条符合条件的线路", 1000).show();
                if (totalLine > 1) {
                    nextLineBtn.setEnabled(true);
                }
                
                //Log.i("test",drivingRouteResult.getTaxiInfo().getDistance()+"");
                // 通过getTaxiInfo()可以得到很多关于打车的信息
                //Toast.makeText(RoutePlanningActivity.this,"该路线打车总路程"+ drivingRouteResult.getTaxiInfo().getDistance(), 1000).show();
            }
        }

        @Override
        public void onGetBikingRouteResult(BikingRouteResult arg0) {
            // TODO Auto-generated method stub
            
            Log.i("test", "test5");
            
        }
    };

    /**
     * 驾车线路查询
     */
    private void drivingSearch(int index) {
        DrivingRoutePlanOption drivingOption = new DrivingRoutePlanOption();
        drivingOption.policy(DrivingPolicy.values()[drivingSpinner
                .getSelectedItemPosition()]);// 设置驾车路线策略
        drivingOption.from(PlanNode.withCityNameAndPlaceName("北京", startPlace));// 设置起点
        drivingOption.to(PlanNode.withCityNameAndPlaceName("北京", endPlace));// 设置终点
        routePlanSearch.drivingSearch(drivingOption);// 发起驾车路线规划
    }

    /**
     * 换乘路线查询
     */
    private void transitSearch(int index) {
        TransitRoutePlanOption transitOption = new TransitRoutePlanOption();
        transitOption.city("北京");// 设置换乘路线规划城市，起终点中的城市将会被忽略
        transitOption.from(PlanNode.withCityNameAndPlaceName("北京", startPlace));
        transitOption.to(PlanNode.withCityNameAndPlaceName("北京", endPlace));
        transitOption.policy(TransitPolicy.values()[transitSpinner
                .getSelectedItemPosition()]);// 设置换乘策略
        routePlanSearch.transitSearch(transitOption);
    }

    /**
     * 步行路线查询
     */
    private void walkSearch() {
        WalkingRoutePlanOption walkOption = new WalkingRoutePlanOption();
        walkOption.from(PlanNode.withCityNameAndPlaceName("北京", startPlace));
        walkOption.to(PlanNode.withCityNameAndPlaceName("北京", endPlace));
        routePlanSearch.walkingSearch(walkOption);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.drive_btn:// 驾车
            index = 0;
            drivintResultIndex = 0;
            startPlace = startEt.getText().toString();
            endPlace = endEt.getText().toString();
            driveBtn.setEnabled(false);
            transitBtn.setEnabled(true);
            walkBtn.setEnabled(true);
            nextLineBtn.setEnabled(false);
            drivingSearch(drivintResultIndex);
            break;
        case R.id.transit_btn:// 换乘
            index = 1;
            transitResultIndex = 0;
            startPlace = startEt.getText().toString();
            endPlace = endEt.getText().toString();
            transitBtn.setEnabled(false);
            driveBtn.setEnabled(true);
            walkBtn.setEnabled(true);
            nextLineBtn.setEnabled(false);
            transitSearch(transitResultIndex);
            break;
        case R.id.walk_btn:// 步行
            index = 2;
            startPlace = startEt.getText().toString();
            endPlace = endEt.getText().toString();
            walkBtn.setEnabled(false);
            driveBtn.setEnabled(true);
            transitBtn.setEnabled(true);
            nextLineBtn.setEnabled(false);
            walkSearch();
            break;
        case R.id.nextline_btn:// 下一条
            switch (index) {
            case 0:
                drivingSearch(++drivintResultIndex);
                break;
            case 1:
                transitSearch(transitResultIndex);
                break;
            case 2:

                break;
            }
            break;
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        mapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mapView.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        routePlanSearch.destroy();// 释放检索实例
        mapView.onDestroy();
    }

}
```

### 注意
在最新的jar包中，drivingRouteResult.getTaxiInfo().getDistance()，如果没有出租车信息时会报空指针异常

### 参考链接
[百度地图开发（五）之公交信息检索 + 路线规划 - crazy_jack - 博客频道 - CSDN.NET](http://blog.csdn.net/crazy1235/article/details/44069267)

### 效果如下

![这里写图片描述](http://img.blog.csdn.net/20160419202542660)




[whuhan2013/MapAll: 百度地图综合实例代码下载](https://github.com/whuhan2013/MapAll)

### 完成


























