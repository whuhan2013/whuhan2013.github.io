---
layout: post
title: Android之phoneGap入门
date: 2016-3-23
categories: blog
tags: [android]
description: 通告一下，我已不再每天写千字文，准备采用以下的方法进行练习，由于文章篇幅较长，链接较多，建议到简书或博客进行阅读。
---  

### 利用phoneGap可以利用HTML开发安卓应用，是web app的一种，可以有效的提高开发效率，降低开发成本 。

## 第一步：
### 开发环境配置以及基本操作请参考其它文档.
### 新增一个名为 phoneGap 的android项目,将主activity命名为:PhoneGapActivity.java
### 从下载好的 phonegap 找到 lib\android,(下载地址记不太清了,google callback-phonegap-0d1f305)
### 按照以下目录分别复制到android 项目 
### assets\www\phonegap-1.4.1.js
### res\xml\phonegap.xml
### res\xml\plugins.xml
### libs\phonegap-1.4.1.jar
 
### 以上路径除了www外,其它都是必须路径,不能更改名字,没有文件夹就创建一个


## 第二步：创建完成后复制以下代码到AndroidManifest.xml ,这些代码为程序提供权限,当然我们现在用不了这么多权限,但是加进去总没错.

```  
<supports-screens
 android:largeScreens="true"
 android:normalScreens="true"
 android:smallScreens="true"
 android:resizeable="true"
 android:anyDensity="true"
 />
 <uses-permission android:name="android.permission.CAMERA" />
 <uses-permission android:name="android.permission.VIBRATE" />
 <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
 <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
 <uses-permission android:name="android.permission.ACCESS_LOCATION_EXTRA_COMMANDS" />
 <uses-permission android:name="android.permission.READ_PHONE_STATE" />
 <uses-permission android:name="android.permission.INTERNET" />
 <uses-permission android:name="android.permission.RECEIVE_SMS" />
 <uses-permission android:name="android.permission.RECORD_AUDIO" />
 <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
 <uses-permission android:name="android.permission.READ_CONTACTS" />
 <uses-permission android:name="android.permission.WRITE_CONTACTS" />
 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```  

## 修改MainActivity
### 添加完成后,找到我们的主activity PhoneGapActivity.java 找到onCreate方法,
### 替换setContentView(R.layout.main);super.loadUrl("http://baidu.com");或者super.loadUrl("file:///android_asset/www/index.html");

## 第四步：写index.html文件
### 其中包括要调用的方法，和调用成功之后返回的方法，分为成功方法与失败方法
``` 
  <!DOCTYPE HTML>
  <html>
    <head>
      <meta http-equiv="Content-type" name="viewport" content="initial-scale=1.0, maximum-scale=1.0, user-scalable=no, width=device-width">
      <title></title>
  
           <script type="text/javascript" charset="utf-8" src="phonegap-1.4.1.js"> </script>
           <script type="text/javascript" charset="utf-8" src="test.js"> </script>
           <script type="text/javascript">
           function test1()
           {
              alert("HelloWorld");
              navigator.hmCommen.test(successCallback,errorCallback,{});
              
           }
           
           function testlogin(name,age)
           {
              //navigator.hmCommen.testLogin(successCallback,errorCallback,{});
              navigator.hmCommen.testLogin(successCallback,errorCallback,{"name":name,"age":age});
           }
           
           function successCallback()
           {
              alert("Success");
           }
           
           function errorCallback()
           {
              alert("failed");
           }
           function test2()
           {
              alert("hello world");
           }
           </script>
       </head>
       <body>
           <h1>Hello World</h1>
           <button type="button" onclick="test2()">Hello Worold</button>
           <button type="button" onclick="test1()">call me</button>
           <button type="button" onclick="testlogin('abc',12)">login test</button>
       </body>
   </html>
``` 
## 第五步：写js文件
```  
/*
 * PhoneGap is available under *either* the terms of the modified BSD license *or* the
 * MIT License (2008). See http://opensource.org/licenses/alphabetical for full text.
 *
 * Copyright (c) 2005-2010, Nitobi Software Inc.
 * Copyright (c) 2010-2011, IBM Corporation
 */

if (!PhoneGap.hasResource("hmCommen")) {
PhoneGap.addResource("hmCommen");

/**
 * This class contains information about the current network Connection.
 * @constructor
 */
var HmCommen = function() {
  
};



/**
 * Get connection info
 *
 * @param {Function} successCallback The function to call when the Connection data is available
 * @param {Function} errorCallback The function to call when there is an error getting the Connection data. (OPTIONAL)
 */
HmCommen.prototype.test = function(successCallback, errorCallback,options) {
    // Get info
    PhoneGap.exec(successCallback, errorCallback, "HM_service", "test", [options]);
};

HmCommen.prototype.testLogin = function(successCallback, errorCallback,options) {
    // Get info
    PhoneGap.exec(successCallback, errorCallback, "HM_service", "testLogin", [options]);
};


PhoneGap.addConstructor(function() {
    if (typeof navigator.hmCommen === "undefined") {
        navigator.hmCommen = new HmCommen();
    }
   
});
}
```  

## 第六步：配置XML文件
### 在plungs.xml中添加
```  
  <plugin name="HM_service" value="com.zj.phonegaptest.HMTest"/>  
```  

### 注意，第五步中对象必须与XML中配置的相同，value即为要调用的类，这样才知道要调用哪一个类

## 第七步：写实现类
```  
package com.zj.phonegaptest;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import android.widget.Toast;

import com.phonegap.api.*;


public class HMTest extends Plugin {

    @Override
    public PluginResult execute(String arg0, JSONArray arg1, String arg2) {
        // TODO Auto-generated method stub
        JSONObject arg=arg1.optJSONObject(0);
        if(arg0.equalsIgnoreCase("test"))
        {
            return doTest();
        }else if(arg0.equalsIgnoreCase("testLogin"))
        {
            try {
                return testLogin(arg);
            } catch (JSONException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        return null;
    }
    
    
    private String name=null;
    private int age=0;
    PluginResult pluginResult=null;

    private PluginResult testLogin(JSONObject arg) throws JSONException
    {
     name=(String)arg.get("name");
     age=arg.getInt("age");
     if("abc".equals(name))
    {
         pluginResult=new PluginResult(PluginResult.Status.OK);
    }else
    {
        pluginResult=new PluginResult(PluginResult.Status.OK);
    }
     
     return pluginResult;
    }
    
    private PluginResult doTest()
    {
        pluginResult=new PluginResult(PluginResult.Status.OK);
        return pluginResult;
    }

}2
```  

## 参考链接：
phoneGap 基于android 实例 一 - china-orange - ITeye技术网站
http://lvjj.iteye.com/blog/1484479
PhoneGap开发环境搭建 - 随机 - 博客园
http://www.cnblogs.com/Random/archive/2011/12/28/2305398.html

## 运行结果：
![完成](http://img.blog.csdn.net/20160323185004391)












