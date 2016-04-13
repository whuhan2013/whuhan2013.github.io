---
layout: post
title: Android实现监测网络状态
date: 2016-4-13
categories: blog
tags: [android]
description: Android实现监测网络状态
---

### 本文主要用到了安卓监测网络状态变化功能，实现了WIFI,3G,无网络状态切换时发出通知的功能。

### 主要知识点     
1. service
2.  broadcast
3.  接口回调实现  

### service的基本知识 

service可分为   

- 按运行地点分类
   * 本地服务 
  * 远程服务 

- 按按运行类型分类：
     * 前台服务
     * 后台服务

- 按使用方式分类：
    * startService 启动的服务
    * bindService 启动的服务
    * startService 同时也 bindService 启动的服务

### service生命周期 

![这里写图片描述](http://img.blog.csdn.net/20160413195749904)

### 详情请见参考链接：
[Android 中的 Service 全面总结 - - 博客频道 - CSDN.NET](http://blog.csdn.net/woaieillen/article/details/7371871)

[Android开发之如何保证Service不被杀掉（broadcast+system/app） - 其实并不难,是你太悲观 - 博客频道 - CSDN.NET](http://blog.csdn.net/mad1989/article/details/22492519)

### 安卓监测网络状态变化 

### service部分 

```
package com.zj.servicewifi;


import java.util.Date;
import java.util.Timer;
import java.util.TimerTask;

import android.app.Service;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.Binder;
import android.os.IBinder;
import android.provider.SyncStateContract.Constants;
import android.util.Log;

public class WIFIService extends Service{
    int IntentId;
    int NOINTENT=0;
    int WIFI=1;
    int GRS=2;
     
    
    // 实时监听网络状态改变  
    private BroadcastReceiver mReceiver = new BroadcastReceiver()  
    {  
        @Override  
        public void onReceive(Context context, Intent intent)  
        {  
            String action = intent.getAction();  
            if (action.equals(ConnectivityManager.CONNECTIVITY_ACTION))  
            {  
                Timer timer = new Timer();  
                timer.schedule(new QunXTask(getApplicationContext()), new Date());  
            }  
        }  
    };  
  
    public interface GetConnectState  
    {  
        public void GetState(int isConnected); // 网络状态改变之后，通过此接口的实例通知当前网络的状态，此接口在Activity中注入实例对象  
    }  
  
    private GetConnectState onGetConnectState;  
  
    public void setOnGetConnectState(GetConnectState onGetConnectState)  
    {  
        this.onGetConnectState = onGetConnectState;  
    }  
  
    private Binder binder = new MyBinder();  
    private boolean isContected = true;  
  
    @Override  
    public IBinder onBind(Intent intent)  
    {  
        return binder;  
    }  
  
    @Override  
    public void onCreate()  
    {// 注册广播  
        IntentFilter mFilter = new IntentFilter();  
        mFilter.addAction(ConnectivityManager.CONNECTIVITY_ACTION); // 添加接收网络连接状态改变的Action  
        registerReceiver(mReceiver, mFilter);  
    }  
  
    class QunXTask extends TimerTask  
    {  
        private Context context;  
  
        public QunXTask(Context context)  
        {  
            this.context = context;  
        }  
  
        @Override  
        public void run()  
        {  
            if (is3GConnected(context)&&isWifiConnected(context)==false)  
            {  
                System.out.println("hereere*************");
                IntentId= 2;
            } 
            else if(isWifiConnected(context))
            {
                IntentId=WIFI;
            }
            else  
            {  
               IntentId=NOINTENT;
            }  
            
            
            if (onGetConnectState != null)  
            {  
                onGetConnectState.GetState(IntentId); // 通知网络状态改变  
                Log.i("mylog", "通知网络状态改变:" + IntentId);  
            }  
        }  
  
        /* 
         * 判断是3G否有网络连接 
         */  
        private boolean is3GConnected(Context context)  
        {  
            if (context != null)  
            {  
                ConnectivityManager mConnectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);  
                NetworkInfo mNetworkInfo = mConnectivityManager.getActiveNetworkInfo();
                if (mNetworkInfo != null)  
                {  
                    return mNetworkInfo.isAvailable();  
                }  
            }  
            return false;  
        }  
  
        /* 
         * 判断是否有wifi连接 
         */  
        private boolean isWifiConnected(Context context)  
        {  
            if (context != null)  
            {  
                ConnectivityManager mConnectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);  
                NetworkInfo mWiFiNetworkInfo = mConnectivityManager.getNetworkInfo(ConnectivityManager.TYPE_WIFI);  
                if (mWiFiNetworkInfo != null)  
                {  
                    return mWiFiNetworkInfo.isAvailable();  
                }  
            }  
            return false;  
        }  
    }  
  
    public class MyBinder extends Binder  
    {  
        public WIFIService getService()  
        {  
            return WIFIService.this;  
        }  
    }  
  
    @Override  
    public void onDestroy()  
    {  
        super.onDestroy();  
        unregisterReceiver(mReceiver); // 删除广播  
    }  

}
```   

### 注意，其中用到广播接收者，广播接收者有两种注册方式，在代码中注册与在XML文件中注册，本例中在代码中注册了，如果再在XML中注册，会报错 

### activity中代码 

```
package com.zj.servicewifi;

import com.zj.servicewifi.WIFIService.GetConnectState;

import android.app.Activity;
import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.Handler;
import android.os.IBinder;
import android.os.Message;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Toast;

public class MainActivity extends Activity {
    
    protected String TAG = "mylog";  
    WIFIService receiveMsgService;  
    int IntentID=0;
    ServiceConnection sc;
   boolean state;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        
        sc=new ServiceConnection() {
            
            @Override
            public void onServiceDisconnected(ComponentName name) {
                // TODO Auto-generated method stub
                
            }
            
            @Override
            public void onServiceConnected(ComponentName name,  IBinder service) {
                // TODO Auto-generated method stub
                
                            receiveMsgService = ((WIFIService.MyBinder) service)  
                                    .getService();  
                            receiveMsgService.setOnGetConnectState(new GetConnectState() { // 添加接口实例获取连接状态  
                                        @Override  
                                        public void GetState(int id) {  
                                            if (IntentID != id) { // 如果当前连接状态与广播服务返回的状态不同才进行通知显示  
                                                IntentID = id;  
                                                if (IntentID==0) {// 已连接  
                                                    handler.sendEmptyMessage(0);  
                                                } else if(IntentID==1){// 未连接  
                                                    handler.sendEmptyMessage(1);  
                                                }  else if(IntentID==2)
                                                {
                                                    handler.sendEmptyMessage(2);
                                                }
                                            }  
                                        }  
                                    });
                 

            }
        };
    }
    
    public void bind(View view)
    {
        startService(new Intent(MainActivity.this,WIFIService.class));
        bindService(new Intent(MainActivity.this, WIFIService.class), sc, getApplicationContext().BIND_AUTO_CREATE);
        state = true;

    }
    public void unbind(View view)
    {
        if(state ){
            
            unbindService(sc);
            state = false;
        }

    }
    public void start(View view)
    {
        
    }
    public void stop(View view)
    {
        
    }
    @Override  
    protected void onDestroy()  
    {  
        // TODO Auto-generated method stub  
        super.onDestroy();  
        if(state){
            unbindService(sc);
            state = false;
        } 
    }  
    
      Handler handler = new Handler() {  
            public void handleMessage(Message msg) {  
      
                switch (msg.what) {  
                case 0:
                    Toast.makeText(MainActivity.this, "网络未经连接" ,Toast.LENGTH_LONG).show(); 
                    break;
                case 1:// 已连接  
                    Toast.makeText(MainActivity.this, "WIFI已经连接" ,Toast.LENGTH_LONG).show();  
                    break;  
                case 2:// 未连接  
                    Toast.makeText(MainActivity.this, "3G已连接" ,Toast.LENGTH_LONG).show();  
                    break;  
                default:  
                    break;  
                }  
                ;  
            };  
      
        };  
    
    
}
``` 

### 本例中最重要的一点就是在service中定义了一个接口，在activity中实例化，则在service中调用方法的时候，会调用在activity中实例化的方法，不知道这是什么设计模式，只觉得很神奇。  

### 参考链接：
[android 通过Service和Receiver来监听网络状态 - - ITeye技术网站](http://zdpeng.iteye.com/blog/1981584)

[Android判断设备网络连接状态，并判断连接方式 - lzan13的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/lzan13/article/details/8964119)

### 完成


























