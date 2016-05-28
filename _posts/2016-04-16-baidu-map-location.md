---
layout: post
title: 百度地图开发之定位
date: 2016-4-16
categories: blog
tags: [地图开发]
description: 百度地图开发之定位
---

### 本文主要讲述利用百度地图API实现定位功能

### 第一步:下载SDK与申请KEY   
参考链接：[Android 百度地图 SDK v3.0.0 （一） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/37729091)  

### 注意  

百度地图SDK与百度定位SDK已经分开了，如果在工程中同时导入这两个包，会冲突，解决方法是下载包含这两个功能的SDK

参考链接： [百度地图SDK和百度导航SDK合并冲突问题 - Singleton1900的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/singleton1900/article/details/42555113)

### 代码实现

### 将地图中心点移到指定点实现

```
//设定中心点坐标 
LatLng cenpt =  new LatLng(30.663791,104.07281);  
//定义地图状态 
MapStatus mMapStatus = new MapStatus.Builder() 
.target(cenpt) 
.zoom(12) 
.build(); 
//定义MapStatusUpdate对象，以便描述地图状态将要发生的变化 

MapStatusUpdate mMapStatusUpdate = MapStatusUpdateFactory.newMapStatus(mMapStatus); 
//改变地图状态 
mBaiduMap.setMapStatus(mMapStatusUpdate);
```   

### 参考链接：   
[百度地图一打开就自动将中心点移动到定位位置 - 开源中国社区](http://www.oschina.net/question/1389206_158886?fromerr=8bA6r3Xf)

### 要实现定位功能，得到经纬度

### 第一步：配置Manifest文件 

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.zj.mapdemo"
    android:versionCode="1"
    android:versionName="1.0" >
    
        <uses-permission android:name="com.android.launcher.permission.READ_SETTINGS" />
     <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" >
    </uses-permission>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" >
    </uses-permission>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" >
    </uses-permission>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" >
    </uses-permission>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" >
    </uses-permission>
    <uses-permission android:name="android.permission.READ_PHONE_STATE" >
    </uses-permission>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" >
    </uses-permission>
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" >
    </uses-permission>
    <uses-permission android:name="android.permission.READ_LOGS" >
    </uses-permission>
    <uses-permission android:name="android.permission.VIBRATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.WRITE_SETTINGS" />

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="21" />

    <application
        android:name="com.zj.mapdemo.LocationApplication"
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <service
            android:name="com.baidu.location.f"
            android:enabled="true"
            android:process=":remote" >
            <intent-filter>
                <action android:name="com.baidu.location.service_v2.2" >
                </action>
            </intent-filter>
        </service>      
        <meta-data
            android:name="com.baidu.lbsapi.API_KEY"
            android:value="B7h1TyUxC0nNGm4wsalDWHPnsCI9tNke" />
        
        
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```   


### 写Application文件,实例化LocationService

```
package com.zj.mapdemo;



import com.baidu.mapapi.SDKInitializer;
import com.zj.mapdemo.LocationService;
import com.zj.mapdemo.WriteLog;
import android.app.Application;
import android.app.Service;
import android.os.Vibrator;

/**
 * 主Application，所有百度定位SDK的接口说明请参考线上文档：http://developer.baidu.com/map/loc_refer/index.html
 *
 * 百度定位SDK官方网站：http://developer.baidu.com/map/index.php?title=android-locsdk
 * 
 * 直接拷贝com.baidu.location.service包到自己的工程下，简单配置即可获取定位结果，也可以根据demo内容自行封装
 */
public class LocationApplication extends Application {
    public LocationService locationService;
    public Vibrator mVibrator;
    @Override
    public void onCreate() {
        super.onCreate();
        /***
         * 初始化定位sdk，建议在Application中创建
         */
        locationService = new LocationService(getApplicationContext());
        mVibrator =(Vibrator)getApplicationContext().getSystemService(Service.VIBRATOR_SERVICE);
        WriteLog.getInstance().init(); // 初始化日志
        SDKInitializer.initialize(getApplicationContext());  
       
    }
}
```  
  
### locationService实现 

```
package com.zj.mapdemo;

import com.baidu.location.BDLocationListener;
import com.baidu.location.LocationClient;
import com.baidu.location.LocationClientOption;
import com.baidu.location.LocationClientOption.LocationMode;
import android.content.Context;

/**
 * 
 * @author baidu
 *
 */
public class LocationService {
    private LocationClient client = null;
    private LocationClientOption mOption,DIYoption;
    private Object  objLock = new Object();

    /***
     * 
     * @param locationContext
     */
    public LocationService(Context locationContext){
        synchronized (objLock) {
            if(client == null){
                client = new LocationClient(locationContext);
                client.setLocOption(getDefaultLocationClientOption());
            }
        }
    }
    
    /***
     * 
     * @param listener
     * @return
     */
    
    public boolean registerListener(BDLocationListener listener){
        boolean isSuccess = false;
        if(listener != null){
            client.registerLocationListener(listener);
            isSuccess = true;
        }
        return  isSuccess;
    }
    
    public void unregisterListener(BDLocationListener listener){
        if(listener != null){
            client.unRegisterLocationListener(listener);
        }
    }
    
    /***
     * 
     * @param option
     * @return isSuccessSetOption
     */
    public boolean setLocationOption(LocationClientOption option){
        boolean isSuccess = false;
        if(option != null){
            if(client.isStarted())
                client.stop();
            DIYoption = option;
            client.setLocOption(option);
        }
        return isSuccess;
    }
    
    public LocationClientOption getOption(){
        return DIYoption;
    }
    /***
     * 
     * @return DefaultLocationClientOption
     */
    public LocationClientOption getDefaultLocationClientOption(){
        if(mOption == null){
            mOption = new LocationClientOption();
            mOption.setLocationMode(LocationMode.Hight_Accuracy);//可选，默认高精度，设置定位模式，高精度，低功耗，仅设备
            mOption.setCoorType("bd09ll");//可选，默认gcj02，设置返回的定位结果坐标系，如果配合百度地图使用，建议设置为bd09ll;
            mOption.setScanSpan(3000);//可选，默认0，即仅定位一次，设置发起定位请求的间隔需要大于等于1000ms才是有效的
            mOption.setIsNeedAddress(true);//可选，设置是否需要地址信息，默认不需要
            mOption.setIsNeedLocationDescribe(true);//可选，设置是否需要地址描述
            mOption.setNeedDeviceDirect(false);//可选，设置是否需要设备方向结果
            mOption.setLocationNotify(false);//可选，默认false，设置是否当gps有效时按照1S1次频率输出GPS结果
            mOption.setIgnoreKillProcess(true);//可选，默认true，定位SDK内部是一个SERVICE，并放到了独立进程，设置是否在stop的时候杀死这个进程，默认不杀死   
            mOption.setIsNeedLocationDescribe(true);//可选，默认false，设置是否需要位置语义化结果，可以在BDLocation.getLocationDescribe里得到，结果类似于“在北京天安门附近”
            mOption.setIsNeedLocationPoiList(true);//可选，默认false，设置是否需要POI结果，可以在BDLocation.getPoiList里得到
            mOption.SetIgnoreCacheException(false);//可选，默认false，设置是否收集CRASH信息，默认收集
        }
        return mOption;
    }
    
    public void start(){
        synchronized (objLock) {
            if(client != null && !client.isStarted()){
                client.start();
            }
        }
    }
    public void stop(){
        synchronized (objLock) {
            if(client != null && client.isStarted()){
                client.stop();
            }
        }
    }
    
}
```

### MainActivity实现  

```
package com.zj.mapdemo;


import com.baidu.location.BDLocation;
import com.baidu.location.BDLocationListener;
import com.baidu.location.Poi;
import com.baidu.mapapi.SDKInitializer;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.MapStatus;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.model.LatLng;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.text.method.ScrollingMovementMethod;
import android.util.Log;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.TextView;

/***
 * 单点定位示例，用来展示基本的定位结果，配置在LocationService.java中
 * 默认配置也可以在LocationService中修改
 * 默认配置的内容自于开发者论坛中对开发者长期提出的疑问内容
 * 
 * @author baidu
 *
 */
public class MainActivity extends Activity {
    private LocationService locationService;
    private TextView LocationResult;
    private Button startLocation;
    double p1,p2;
    
    MapView mapView;
    BaiduMap mBaiduMap;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        // TODO Auto-generated method stub
        super.onCreate(savedInstanceState);
        // -----------demo view config ------------
        SDKInitializer.initialize(getApplicationContext());
        setContentView(R.layout.location);
        LocationResult = (TextView) findViewById(R.id.textView1);
        LocationResult.setMovementMethod(ScrollingMovementMethod.getInstance());
        startLocation = (Button) findViewById(R.id.addfence);
        
        mapView=(MapView) findViewById(R.id.id_bmapView2);
        mBaiduMap=mapView.getMap();
        
        
        startLocation.setOnClickListener(new OnClickListener() {
            
            @Override
            public void onClick(View v) {
                // TODO Auto-generated method stub
                                move(); 
            }
        });

    }
    
    
    public void change()
    {
        //设定中心点坐标 
                LatLng cenpt =  new LatLng(30.663791,110.07281);  
                
                //定义地图状态 
                MapStatus mMapStatus = new MapStatus.Builder() 
                .target(cenpt) 
                .zoom(12) 
                .build(); 
                //定义MapStatusUpdate对象，以便描述地图状态将要发生的变化 

                MapStatusUpdate mMapStatusUpdate = MapStatusUpdateFactory.newMapStatus(mMapStatus); 
                //改变地图状态 
                mBaiduMap.setMapStatus(mMapStatusUpdate);
    }

    /**
     * 显示请求字符串
     * 
     * @param str
     */
    public void logMsg(String str) {
        try {
            if (LocationResult != null)
                LocationResult.setText(str);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public void move()
    {
        // -----------location config ------------
                locationService = ((LocationApplication) getApplication()).locationService; 
                //获取locationservice实例，建议应用中只初始化1个location实例，然后使用，可以参考其他示例的activity，都是通过此种方式获取locationservice实例的
                locationService.registerListener(mListener);
                //注册监听
                int type = getIntent().getIntExtra("from", 0);
                if (type == 0) {
                    locationService.setLocationOption(locationService.getDefaultLocationClientOption());
                } else if (type == 1) {
                    locationService.setLocationOption(locationService.getOption());
                }
            
                            locationService.start();// 定位SDK
                                                    // start之后会默认发起一次定位请求，开发者无须判断isstart并主动调用request
                            startLocation.setText(getString(R.string.stoplocation));
                            
                            Log.i("location",p1+"");
                            
                            if(p1>0)
                            {
                                Log.i("location",p2+"");
                            //设定中心点坐标 
                            LatLng cenpt =  new LatLng(30,30);  
                            
                            //定义地图状态 
                            MapStatus mMapStatus = new MapStatus.Builder() 
                            .target(cenpt) 
                            .zoom(12) 
                            .build(); 
                            //定义MapStatusUpdate对象，以便描述地图状态将要发生的变化 

                            MapStatusUpdate mMapStatusUpdate = MapStatusUpdateFactory.newMapStatus(mMapStatus); 
                            //改变地图状态 
                            mBaiduMap.setMapStatus(mMapStatusUpdate);
                            }
                            
                        
    }
    
    Handler handler=new Handler()
    {
        public void handleMessage(android.os.Message msg) {
            Log.i("location",p2+"-------------------+");
            //设定中心点坐标 
            LatLng cenpt =  new LatLng(p1,p2);  
            
            //定义地图状态 
            MapStatus mMapStatus = new MapStatus.Builder() 
            .target(cenpt) 
            .zoom(12) 
            .build(); 
            //定义MapStatusUpdate对象，以便描述地图状态将要发生的变化 

            MapStatusUpdate mMapStatusUpdate = MapStatusUpdateFactory.newMapStatus(mMapStatus); 
            //改变地图状态 
            mBaiduMap.setMapStatus(mMapStatusUpdate);
            }
        };
    

    
    /***
     * Stop location service
     */
    @Override
    protected void onStop() {
        // TODO Auto-generated method stub
        locationService.unregisterListener(mListener); //注销掉监听
        locationService.stop(); //停止定位服务
        super.onStop();
    }

    /*@Override
    protected void onStart() {
        // TODO Auto-generated method stub
        super.onStart();
        // -----------location config ------------
        locationService = ((LocationApplication) getApplication()).locationService; 
        //获取locationservice实例，建议应用中只初始化1个location实例，然后使用，可以参考其他示例的activity，都是通过此种方式获取locationservice实例的
        locationService.registerListener(mListener);
        //注册监听
        int type = getIntent().getIntExtra("from", 0);
        if (type == 0) {
            locationService.setLocationOption(locationService.getDefaultLocationClientOption());
        } else if (type == 1) {
            locationService.setLocationOption(locationService.getOption());
        }
        startLocation.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                if (startLocation.getText().toString().equals(getString(R.string.startlocation))) {
                    locationService.start();// 定位SDK
                                            // start之后会默认发起一次定位请求，开发者无须判断isstart并主动调用request
                    startLocation.setText(getString(R.string.stoplocation));
                    
                    //设定中心点坐标 
                    LatLng cenpt =  new LatLng(30,30);  
                    
                    //定义地图状态 
                    MapStatus mMapStatus = new MapStatus.Builder() 
                    .target(cenpt) 
                    .zoom(12) 
                    .build(); 
                    //定义MapStatusUpdate对象，以便描述地图状态将要发生的变化 

                    MapStatusUpdate mMapStatusUpdate = MapStatusUpdateFactory.newMapStatus(mMapStatus); 
                    //改变地图状态 
                    mBaiduMap.setMapStatus(mMapStatusUpdate);
                    
                } else {
                    locationService.stop();
                    startLocation.setText(getString(R.string.startlocation));
                }
            }
        });
    }*/

    
    /*****
     * @see copy funtion to you project
     * 定位结果回调，重写onReceiveLocation方法，可以直接拷贝如下代码到自己工程中修改
     *
     */
    private BDLocationListener mListener = new BDLocationListener() {

        @Override
        public void onReceiveLocation(BDLocation location) {
            // TODO Auto-generated method stub
            if (null != location && location.getLocType() != BDLocation.TypeServerError) {
                StringBuffer sb = new StringBuffer(256);
                sb.append("time : ");
                /**
                 * 时间也可以使用systemClock.elapsedRealtime()方法 获取的是自从开机以来，每次回调的时间；
                 * location.getTime() 是指服务端出本次结果的时间，如果位置不发生变化，则时间不变
                 */
                sb.append(location.getTime());
                sb.append("\nerror code : ");
                sb.append(location.getLocType());
                sb.append("\nlatitude : ");
                sb.append(location.getLatitude());
                
                p1=location.getLatitude();
                Log.i("location", location.getLatitude()+"p1======="+p1);
                sb.append("\nlontitude : ");
                sb.append(location.getLongitude());
                
                Log.i("location", location.getLongitude()+"");
                p2=location.getLongitude();
                
                handler.sendEmptyMessage(0);
                sb.append("\nradius : ");
                sb.append(location.getRadius());
                
                sb.append("\nCountryCode : ");
                sb.append(location.getCountryCode());
                sb.append("\nCountry : ");
                sb.append(location.getCountry());
                sb.append("\ncitycode : ");
                sb.append(location.getCityCode());
                sb.append("\ncity : ");
                sb.append(location.getCity());
                sb.append("\nDistrict : ");
                sb.append(location.getDistrict());
                sb.append("\nStreet : ");
                sb.append(location.getStreet());
                sb.append("\naddr : ");
                sb.append(location.getAddrStr());
                sb.append("\nDescribe: ");
                sb.append(location.getLocationDescribe());
                sb.append("\nDirection(not all devices have value): ");
                sb.append(location.getDirection());
                sb.append("\nPoi: ");
                if (location.getPoiList() != null && !location.getPoiList().isEmpty()) {
                    for (int i = 0; i < location.getPoiList().size(); i++) {
                        Poi poi = (Poi) location.getPoiList().get(i);
                        sb.append(poi.getName() + ";");
                    }
                }
                if (location.getLocType() == BDLocation.TypeGpsLocation) {// GPS定位结果
                    sb.append("\nspeed : ");
                    sb.append(location.getSpeed());// 单位：km/h
                    sb.append("\nsatellite : ");
                    sb.append(location.getSatelliteNumber());
                    sb.append("\nheight : ");
                    sb.append(location.getAltitude());// 单位：米
                    sb.append("\ndescribe : ");
                    sb.append("gps定位成功");
                } else if (location.getLocType() == BDLocation.TypeNetWorkLocation) {// 网络定位结果
                    // 运营商信息
                    sb.append("\noperationers : ");
                    sb.append(location.getOperators());
                    sb.append("\ndescribe : ");
                    sb.append("网络定位成功");
                } else if (location.getLocType() == BDLocation.TypeOffLineLocation) {// 离线定位结果
                    sb.append("\ndescribe : ");
                    sb.append("离线定位成功，离线定位结果也是有效的");
                } else if (location.getLocType() == BDLocation.TypeServerError) {
                    sb.append("\ndescribe : ");
                    sb.append("服务端网络定位失败，可以反馈IMEI号和大体定位时间到loc-bugs@baidu.com，会有人追查原因");
                } else if (location.getLocType() == BDLocation.TypeNetWorkException) {
                    sb.append("\ndescribe : ");
                    sb.append("网络不同导致定位失败，请检查网络是否通畅");
                } else if (location.getLocType() == BDLocation.TypeCriteriaException) {
                    sb.append("\ndescribe : ");
                    sb.append("无法获取有效定位依据导致定位失败，一般是由于手机的原因，处于飞行模式下一般会造成这种结果，可以试着重启手机");
                }
                logMsg(sb.toString());
            }
        }

    };
}
```  


### 主要思路是先利用百度提供的例子得到当前经纬度，然后handler发送消息，接收消息后当前地图中心平移到当前位置  

### 注意，百度地图版本较多，以官方例子为准，不然会出很多BUG

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160416213354361)


### 更新，为当前位置加上图标

```
// 构建Marker图标
            BitmapDescriptor bitmap = null;
            
                bitmap = BitmapDescriptorFactory.fromResource(R.drawable.navi_map_gps_locked); // 非推算结果
        
            // 构建MarkerOption，用于在地图上添加Marker
            OverlayOptions option = new MarkerOptions().position(cenpt).icon(bitmap);
            // 在地图上添加Marker，并显示
            mBaiduMap.addOverlay(option);
            mBaiduMap.setMapStatus(MapStatusUpdateFactory.newLatLng(cenpt));
```





























