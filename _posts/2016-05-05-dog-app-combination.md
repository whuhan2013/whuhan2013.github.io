---
layout: post
title: 狗脸识别APP整合
date: 2016-5-05
categories: blog
tags: [android]
description: 狗脸识别APP整合
---   

### 本文主要包括以下内容  

1. android studio中导入so文件  
2. 通过URI获得Bitmap  

### android studio中导入so文件  

在main文件夹下建立jniLibs目录，并将so文件拷贝进去即可。

## 注意
声明的native方法与so文件中定义的方法的包名必须相同

### 通过URI获得Bitmap    

```
private Bitmap getBitmapFromUri(Uri uri)
 {
  try
  {
   // 读取uri所在的图片
   Bitmap bitmap = MediaStore.Images.Media.getBitmap(this.getContentResolver(), uri);
   return bitmap;
  }
  catch (Exception e)
  {
   Log.e("[Android]", e.getMessage());
   Log.e("[Android]", "目录为：" + uri);
   e.printStackTrace();
   return null;
  }
 }
```

### 参考链接  
[(1)Android 通过Uri获取Bitmap对象_又吹风了_新浪博客](http://blog.sina.com.cn/s/blog_5de73d0b0100zfm8.html)

### 源代码  

[whuhan2013/AndroidApp: This is the summary of Android App](https://github.com/whuhan2013/AndroidApp/tree/master/DogApp)

### 整合效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160505142943483)