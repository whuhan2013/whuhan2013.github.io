---
layout: post
title: Ionic环境搭建   
date: 2016-7-1
categories: blog
tags: [Ionic]
description: Ionic环境搭建
---


**stepts**

```
npm install -g ionic@beta        
```

Make sure you have NodeJS installed. Download the installer here or use your favorite package manager. It’s best to get the 5x version of node along with the 3x version of npm. This offers the best in stability and speed for building.

Once that’s done, create your first Ionic app:

```
 ionic start cutePuppyPics --v2
```

To run your app, cd into the directory that was created and then run the ionic serve command:

```
$ cd cutePuppyPics
$ ionic serve
```

You can play with it right in the browser!   

![](http://ionicframework.com/img/docs/tutorial-screen.png)          




**Building to a Device**  


After you have Ionic installed, you can build your app to a physical device. If you don’t have a physical device on hand, you can still build to a device emulator. Check out the iOS simulator docs if you are on a Mac, or the Genymotion docs if you are looking to emulate an Android device. You will also need Cordova to run your app on a native device. To install Cordova, run:

```
$ sudo npm install -g cordova
```


**Building for Android**  

To build for Android, you’ll need to add the Android platform module to Cordova:

```
$ ionic platform add android
```

Next, you’ll need to install the Android SDK. The Android SDK allows you to build compile to a target device running Android. Although the Android SDK comes with a stock emulator, Genymotion is recommended, since it’s much faster. Once installed, start an Android image and run:

```
$ ionic run android
```


### Problems

- As a common cause for Chinese programmers,the existence of GFW will make it hard for us to dowlad gradles

**solutions:**

1. dowload specificed version gradle from Internet.   
2. paste it in somewhere likes myApp\platforms\android\gradle\gradle-2.2.1-all.zip  
3. change distributionUrl in myApp/platforms/android/cordova/lib/builders/GradleBuilder.js 

```
from 
var distributionUrl = 'distributionUrl=http\\://services.gradle.org/distributions/gradle-2.2.1-all.zip';
to
var distributionUrl = 'distributionUrl=../gradle-2.2.1-all.zip';
```

**references**  

[ionic build android error when download gradle - Stack Overflow](http://stackoverflow.com/questions/29874564/ionic-build-android-error-when-download-gradle)


- ionic serve opened a blank page  

**solutions:**

change your browse,firefox or chorme will be fine.  


- EveryTime I run ionic run android,the code will go back to the original and don't change.

**solutions:** 

there are three directory have page htmls    

1. app/
2. www/
3. android/

please edit code in app directory and it will work well.







