---
layout: post
title: ShareSDK集成微信、QQ、微博分享
date: 2016-5-27
categories: blog
tags: [android]
description: ShareSDK集成微信、QQ、微博分享
---   

**1、前言**          

为什么要使用第三方的作为集成分享的工具呢？而不去用官方的呢？有什么区别么？
一个字"快"，如果你使用官方的得一个个集成他们的SDK，相信这是一个痛苦的过程。

**2、准备需要分享的各个平台的key**      

这个需要自己去各个开放平台注册应用得到appkey，
才可以分享到该平台（QQ、微信、微博开放平台），          
不然人家也不会让你无故分享到他们的平台          

**3、申请ShareSDK的appkey**  

http://bbs.mob.com/forum.php?mod=viewthread&tid=8212&extra=page%3D1

**4、下载SDK** 

http://www.mob.com/#/downloadDetail/ShareSDK/android

**5、开始集成** 

官方集成地址http://www.cnblogs.com/smyhvae/p/4585340.html

**添加应用信息：**

先在app这个module（即我们这个项目的module）下新建一个assets文件夹（即第三方资产目录），操作如下：


![](http://upload-images.jianshu.io/upload_images/2098971-25f13addb29aeebb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2098971-1ddaf050844f83bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后，我们将上图中的ShareSDK.xml文件复制到assets目录下。

### (1)然后配置AndroidManifest.xml添加权限

```
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
```


### (2)在application结点下注册activity

```

      <activity
            android:name="com.mob.tools.MobUIShell"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"
            android:configChanges="keyboardHidden|orientation|screenSize"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="stateHidden|adjustResize" >
            <intent-filter>
                <data android:scheme="tencent1104646053" />
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```

主要用于QQ分享

如果项目集成了微信，还需要添加以下WXEntryActivity，不然的话，mob后台无法做微信的分享统计：（**注意：一定要在工程的包下新建wxapi目录再放置WXEntryActivity，包名、包名、包名wxapi**）


```
 * 官网地站:http://www.mob.com
 * 技术支持QQ: 4006852216
 * 官方微信:ShareSDK   （如果发布新版本的话，我们将会第一时间通过微信将版本更新内容推送给您。如果使用过程中有任何问题，也可以通过微信与我们取得联系，我们将会在24小时内给予回复）
 *
 * Copyright (c) 2013年 mob.com. All rights reserved.
 */
package com.smyhvae.sharedemo.wxapi;
import android.content.Intent;
import android.widget.Toast;
import cn.sharesdk.wechat.utils.WXAppExtendObject;
import cn.sharesdk.wechat.utils.WXMediaMessage;
import cn.sharesdk.wechat.utils.WechatHandlerActivity;
/** 微信客户端回调activity示例 */
public class WXEntryActivity extends WechatHandlerActivity {
    /**
     * 处理微信发出的向第三方应用请求app message
     * <p>
     * 在微信客户端中的聊天页面有“添加工具”，可以将本应用的图标添加到其中
     * 此后点击图标，下面的代码会被执行。Demo仅仅只是打开自己而已，但你可
     * 做点其他的事情，包括根本不打开任何页面
     */
    public void onGetMessageFromWXReq(WXMediaMessage msg) {
        Intent iLaunchMyself = getPackageManager().getLaunchIntentForPackage(getPackageName());
        startActivity(iLaunchMyself);
    }
    /**
     * 处理微信向第三方应用发起的消息
     * <p>
     * 此处用来接收从微信发送过来的消息，比方说本demo在wechatpage里面分享
     * 应用时可以不分享应用文件，而分享一段应用的自定义信息。接受方的微信
     * 客户端会通过这个方法，将这个信息发送回接收方手机上的本demo中，当作
     * 回调。
     * <p>
     * 本Demo只是将信息展示出来，但你可做点其他的事情，而不仅仅只是Toast
     */
    public void onShowMessageFromWXReq(WXMediaMessage msg) {
        if (msg != null && msg.mediaObject != null
                && (msg.mediaObject instanceof WXAppExtendObject)) {
            WXAppExtendObject obj = (WXAppExtendObject) msg.mediaObject;
            Toast.makeText(this, obj.extInfo, Toast.LENGTH_SHORT).show();
        }
    }
}
```

同时，在清单文件中进行声明：

```
        <activity
            android:name=".wxapi.WXEntryActivity"
            android:configChanges="keyboardHidden|orientation|screenSize"
            android:exported="true"
            android:screenOrientation="portrait"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"/>
```


### (3)添加代码

集成ShareSDK至少需要在两个位置添加代码，包括：

（一）在项目的入口Activity，在其onCreate方法中插入下面的代码进行初始化：（这个方法越早调用越好）

ShareSDK.initSDK(this);

如果不在所有的ShareSDK的操作之前调用这行代码，就会抛出空指针异常。

（二）在项目出口Activity的onDestroy方法中第一行插入下面的代码：

ShareSDK.stopSDK(this);

上方这行代码会结束ShareSDK的统计功能并释放资源。如果这行代码没有被调用，那么“应用启动次数”将会不准确，因为应用可能从来没有被关闭过（注：这一行代码我还是没用到，不知道会造成什么实质性的后果）。

开始编写分享代码
分享到微信好友：

```
Wechat.ShareParams sp = new Wechat.ShareParams();
sp.setTitle(resp.getTitle());
sp.setText(resp.getDescription());
sp.setShareType(Platform.SHARE_WEBPAGE);
sp.setUrl(resp.getLink());
sp.setImageUrl(resp.getIcon());
Platform wechat = ShareSDK.getPlatform(Wechat.NAME);
wechat.setPlatformActionListener(new PlatformActionListener() {  
  @Override 
   public void onComplete(Platform platform, int i, HashMap<String, Object> hashMap) {  
      popupWindow.dismiss();  
      statisticsShareCount();    
    ToastUtils.show(context, "分享成功");  
  }   
 @Override   
 public void onError(Platform platform, int i, Throwable throwable) {     
   ToastUtils.show(context, "分享失败");  
  }   
 @Override 
   public void onCancel(Platform platform, int i) {  
      ToastUtils.show(context, "分享取消");  
  }
});
// 执行图文分享wechat.share(sp);
```


### 参考链接

[在Android Studio中使用shareSDK进行社会化分享（图文教程） - 生命壹号 - 博客园](http://www.cnblogs.com/smyhvae/p/4585340.html)


[Android 使用ShareSDK集成微信、QQ、微博分享 - 简书](http://www.jianshu.com/p/486940baf91f)





