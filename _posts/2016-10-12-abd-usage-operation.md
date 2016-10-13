---
layout: post
title: adb常用操作
date: 2016-10-12
categories: blog
tags: [android]
description: adb常用
---


**adb运行automation测试**  

```
adb shell am instrument -w -r   -e debug false -e class com.rock.myapplication.RunnerTest com.rock.myapplication.test/android.support.test.runner.AndroidJUnitRunner
```

参考：[Android, Error=Unable to find instrumentation info for: ComponentInfo](http://stackoverflow.com/questions/36753486/android-error-unable-to-find-instrumentation-info-for-componentinfo)

**Android获取包名的方法**

```
aapt dump badging ~/temp/automattest/apk/app-internal-release-3.6.21.apk 
```

参考：[Android获取包名的方法](http://blog.sina.com.cn/s/blog_5b478f870102v5bo.html)


**如何用adb启动apk**

```
adb shell am start -n breakan.test/breakan.test.TestActivity
```

**adb截图并发送到电脑** 

```
adb shell /system/bin/screencap -p /sdcard/screenshot.png（保存到SDCard）
adb pull /sdcard/screenshot.png d:/screenshot.png（保存到电脑）
```













