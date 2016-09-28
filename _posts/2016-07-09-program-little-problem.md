---
layout: post
title: 开发中的小问题总结
date: 2016-7-9
categories: blog
tags: [错误&问题]
description: 开发中的小问题总结 
---


### 本文主要包括以下内容  

1. Android6.0移除了httpClient包 
2. 获取e.printstacktrace()的值

### Android6.0移除了httpClient包  

android 6.0中移除了httpclient中的包，导致有一些httpclient相关的包找不到，通常是在网络处理是算错。

**解决方法** 

导入sdk\platforms\android-23\optional路径下的org.apache.http.legacy.jar


#### 获取e.printstacktrace()的值

```
public static void main(String[] args) {  
        try {  
            String aa = "";  
            System.out.println(aa.substring(3));  
  
        } catch (Exception e) {  
            e.printStackTrace();  
            StringWriter sw = new StringWriter();  
            e.printStackTrace(new PrintWriter(sw, true));  
            String str = sw.toString();  
            System.out.println("==========");  
  
            System.out.println(str);  
        }  
    }  
```

