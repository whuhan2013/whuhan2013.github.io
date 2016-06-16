---
layout: post
title: FastPolice项目总结
date: 2016-6-16
categories: blog
tags: [android]
description: FastPolice项目总结
---


This is the final homework for spatial information Mobile Service Lesson.It generally inclusived these models. 

**welcome page** 

I used a openlibary to do it ,when you first launch your application,the prompt page will be seen but other time you will see the welcome page.

**References**  

[AppIntro](https://github.com/PaoloRotolo/AppIntro)

![这里写图片描述](http://img.blog.csdn.net/20160616122958232)


**Main Interface**


I used viewpage and tablayout to achive tab effect,and achived Immersive status bar. 


![这里写图片描述](http://img.blog.csdn.net/20160616123711991)


**3D Menu** 


This 3D Menu is pretty cool to me,it can change skin for your application too,and the slide of the menu may interrupt with the viewpager ,so you should dispatch touch event carefully.    This module also used

-  [Icon Font](http://www.jianshu.com/p/dd01072998c5)
- [multipletheme](https://github.com/dersoncheng/MultipleTheme)
- [residelayout](https://github.com/dongjunkun/ResideLayout)
- [material dialog](https://github.com/afollestad/material-dialogs) - [swipetoloadlayout](https://github.com/Aspsine/SwipeToLoadLayout). 

**References** 

[一款优雅的干货集中营Android客户端，实现了沉浸式状态栏，无缝换肤，带3D感觉的侧滑菜单](http://www.jianshu.com/p/3a78ea86b571)


<figure class="half">
    <img src="http://img.blog.csdn.net/20160616124648761" alt="screenshot" title="screenshot" width="250" height="436" >   
    <img src="http://img.blog.csdn.net/20160616124703043" alt="screenshot" title="screenshot" width="250" height="436" >
</figure>


<figure class="half"><img src="http://dreamofbook.qiniudn.com/Zero.png"><img src="http://dreamofbook.qiniudn.com/Zero.png"></figure>

<table><tr>
<td><img src="http://dreamofbook.qiniudn.com/Zero.png" border=0></td>
<td><img src="http://dreamofbook.qiniudn.com/Zero.png" border=0></td>
</tr></table>


**Police Fragment** 

I used a sliding folding recyllerview to achieve this effect.if you want to know more can read [RecyclerView实现滑动折叠效果](http://whuhan2013.github.io/blog/2016/06/02/recycleview-slide-fold/)

I also did a little wave at the botom of this fragmet,it is just a sine and a cosine function that make paint look like waves,if you would like to know more can read [自定义view实现水波纹效果](http://whuhan2013.github.io/blog/2016/05/21/android-water-ripple/)


and if you clicke the recyllerview item you will enter a detail pages about police and it used a turn over card animation,you can learn more form [实现翻转卡片的动画效果 - 良有以也](http://whuhan2013.github.io/blog/2016/06/04/card-turnover-effect/)


![这里写图片描述](http://img.blog.csdn.net/20160616125921413)



![这里写图片描述](http://img.blog.csdn.net/20160616125930773)



**girl fragment** 

This fragment is pretty little and beautiful,I use MVP design pattern to construction project.This fragment used okhttp and retrofit to get date from webserver api and use rxjava and dbflow also,if you want to learn more you can click following links.

- [retrofit](http://whuhan2013.github.io/blog/2016/06/10/retrofit-learn-start/)
- [retorit and rxjava](http://whuhan2013.github.io/blog/2016/06/11/retrofit-and-rxjava/)
- [MVP desing pattern](http://whuhan2013.github.io/blog/2016/06/09/android-mvp-pattern/)
- [dbflow](http://whuhan2013.github.io/blog/2016/06/12/android-dbflow-learn/)


![这里写图片描述](http://img.blog.csdn.net/20160616130630187)



**MangeFragment**

In this Fragmet you can distribute cases to polices and the case had been deald would be yeallow and others would be blue,In this fragmnet we used SwipeRefreshLayout to achieve pull to refresh and sawtooth effect.

- [SwipeRefreshLayout ](http://whuhan2013.github.io/blog/2016/06/08/android-swipelayout-learn/)
- [sawtooth effect](http://whuhan2013.github.io/blog/2016/05/21/android-edge-sawtooth/)

![这里写图片描述](http://img.blog.csdn.net/20160616131539493)

![这里写图片描述](http://img.blog.csdn.net/20160616131603337)


**Search Effect** 


I used a custom view to achieve a search effect that look like radar what can search police who is cloe to the case.  

[Radar efect](http://whuhan2013.github.io/blog/2016/05/12/android-viewpager-radar/)

![这里写图片描述](http://img.blog.csdn.net/20160616131838079).


**source code**

[Fastpolice](https://github.com/whuhan2013/fastpolice)

**open libary used** 

```
 compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
    compile 'com.github.paolorotolo:appintro:3.4.0'
    compile 'com.jakewharton:butterknife:7.0.1'
    compile 'com.mikepenz:iconics-core:2.5.5@aar'
    compile 'com.mikepenz:material-design-iconic-typeface:2.2.0.1@aar'
    compile 'com.mikepenz:fontawesome-typeface:4.5.0.1@aar'
    compile 'com.mikepenz:foundation-icons-typeface:3.0.0.1@aar'
    compile 'com.android.support:support-v4:23.4.0'
    compile 'jp.wasabeef:glide-transformations:1.3.1'
    compile('com.github.afollestad.material-dialogs:core:0.8.5.4@aar') {
        transitive = true
    }
    compile('com.github.afollestad.material-dialogs:commons:0.8.5.4@aar') {
        transitive = true
    }
    compile 'com.android.support:design:23.2.1'
    compile 'com.android.support:recyclerview-v7:23.2.1'
    compile 'com.android.support:cardview-v7:23.2.1'
    compile 'com.github.coyarzun89:fabtransitionactivity:0.2.0'
    apt "com.github.Raizlabs.DBFlow:dbflow-processor:${dbflow_version}"
    compile "com.github.Raizlabs.DBFlow:dbflow-core:${dbflow_version}"
    compile "com.github.Raizlabs.DBFlow:dbflow:${dbflow_version}"
    compile "com.github.Raizlabs.DBFlow:dbflow-sqlcipher:${dbflow_version}"
    compile 'jp.wasabeef:recyclerview-animators:2.2.1'
    compile 'com.squareup.retrofit2:retrofit:2.0.0'
    compile 'com.google.code.gson:gson:2.6.2'
    compile 'com.squareup.retrofit2:converter-gson:2.0.0'
    compile 'com.squareup.okhttp3:okhttp:3.2.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0'
    compile 'com.commit451:PhotoView:1.2.5'
    compile 'com.squareup.picasso:picasso:2.5.2'
    compile 'io.reactivex:rxandroid:1.1.0'
    compile 'io.reactivex:rxjava:1.1.0'
    compile files('libs/baidumapapi_base_v3_7_3.jar')
    compile files('libs/baidumapapi_util_v3_7_3.jar')
    compile files('libs/baidumapapi_search_v3_7_3.jar')
    compile files('libs/baidumapapi_cloud_v3_7_3.jar')
    compile files('libs/baidumapapi_radar_v3_7_3.jar')
    compile files('libs/baidumapapi_map_v3_7_3.jar')
    compile files('libs/locSDK_6.13.jar')
    compile files('libs/xUtils-2.4.7.jar')
    compile files('libs/org.apache.http.legacy.jar')
```


**Thanks**

- [CoXier/Girl](https://github.com/CoXier/Girl)
-  [dongjunkun/GanK](https://github.com/dongjunkun/GanK)


**LICENSE**
***

Source code released under the V3 GPL agreement,make sure you understand this aggrement. 


```
Copyright 2014 drakeet

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

 http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

