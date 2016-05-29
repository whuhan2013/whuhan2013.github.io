---
layout: post
title: WebService入门
date: 2016-5-29
categories: blog
tags: [网络编程]
description: WebService入门
---   

### webservice 的概念，解决什么问题？

webservice 就是一个应用程序，它提供一种通过web 方式访问的api.
解决两个系统或者（应用程序）之间的远程调用.....       
调用是跨语言，跨平台...            
webservice 最基本的组成部分就是客户端，服务端...        

**webservice 中的一些概念**                     
服务端：(作为服务端，怎么将自己的应用程序发布成一个webservice，让别人调用)                      
xml （webservice的客户端与服务端进行交互的时候传递的数据格式）     
webservice description language（web 服务描述语言.. api）xml,简称wsdl                         
soap（简单对象访问协议） webservice 的客户端与服务端进行交互的时候走的协议                  
(soap 分两个版本（soap 1.1 与soap1.2）),现在的本是soap1.1,因为java jdk 只支持soap1.1版本的协议发布                
***** soap 协议=在http 的基础之上传送xml 格式的数据..   


### 发布服务：


```
package cn.itcast.server;

import javax.xml.ws.Endpoint;

public class PublishServer {

  /**
   * @param args
   */
  public static void main(String[] args) {
    //java jdk 提供一个自带的类可以将java 应用程序发布成webservice 
    /**
     * 1,提供服务对外的访问地址
     * 2,提供服务的类的对象...
     */
    
    Endpoint.publish("http://10.129.69.114:9999/helloService", new HelloService());
  }

}
```


### 被发布的类

```
package com.zj.server;

import javax.jws.WebService;

import com.zj.bean.User;

@WebService
public class HelloService {

  public void doubleKill()
  {
    System.out.println("zj");
  }
  
  public User getUserById(int id){
    User user=new User();
    user.setId(id);
    user.setMomo("123445");
    user.setUsername("zj");
    user.setWeibo("zj@sina.com.cn");
    user.setWeixin("l89999");
    return user;
  }
}

```

**注意：**1,endpoint是java jdk 提供的类，用来发布webservice，所以你的jdk 版本必须在1.6.0_21之上..
      2,被发布的类当中必须包含一个有效（方法必须为publish的非静态的，非final的方法）的方法
      3,被发布的类上面必须有注解...


### 客户端：

(作为客户端，怎么调用别人发布的webservice)          

调用服务：我们可以通过java jdk 自带的一个命令 wsimport 根据服务端说明书（wsdl）生成本地的java 代码

我们直接操作这些java 代码，就可以调用webservice  

- wsimport -d . +服务说明书（wsdl）的地址 生成本地的class 文件
- wsimport -s . +服务说明书（wsdl）的地址 生成本地的class 文件与java文件
- wsimport -s . -p(包名)+服务说明书（wsdl）的地址 生成本地的class 文件与java文件

**第一种方式调用：通过wsimport**

工具1：webservice explorer （通过图形化界面的方式调用webservice）

工具2：tcp/ip Monitor  可以拦截webservice客户端与webservice 服务端进行交互的整个过程以及数据传输的格式

**代码**

```
package com.zj.clent;

import com.zj.server.HelloService;
import com.zj.server.HelloServiceService;

public class InvokeHelloService {
  
  public static void main(String[] args) {
    HelloServiceService helloServiceService=new HelloServiceService();
    
    HelloService helloService=helloServiceService.getHelloServicePort();
    
    helloService.doubleKill();
  }

}
```

**获得返回值**

```
package com.zj.clent;

import com.baidu.google.soso.so.HelloService;
import com.baidu.google.soso.so.HelloServiceService;
import com.baidu.google.soso.so.User;

public class WsimportInvoke {

  public static void main(String[] args) {
    
    HelloServiceService helloServiceService=new HelloServiceService();
    
    HelloService helloService=helloServiceService.getHelloServicePort();
    
     User user=helloService.getUserById(1);
    System.out.println(user.getMomo());
  }
}
```




**第二种方式调用:通过java jdk 自带的一个类URLConnect(可以发送一个http 请求)**      

原理：我们可以通过URLConnect 这个对象，发送一个http 请求，往webservice 服务端 传送xml 格式的数据，
模拟soap 协议 ,因为soap协议就是在http 的基础上传送xml格式的数据..

```
package com.zj.clent;

import java.io.IOException;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;

public class URLConnectionInvoke {
  
  public static void main(String[] args) throws IOException {
    
    URL url=new URL("http://10.129.69.114:9999/helloService");
     HttpURLConnection connection=(HttpURLConnection) url.openConnection();
     
     connection.setDoInput(true);   //要往服务器接收
     connection.setDoOutput(true);     //要往服务器传送数据，必须设置为ture
     
     
     connection.setRequestProperty("Content-Type", "text/xml;charset=utf-8");
     
     connection.setRequestMethod("POST");
     
     OutputStream outputStream=connection.getOutputStream();
     //outputStream.write(b);
     
     
  }
}

```





**第三种方式调用：通过客户端编程的方式调用 webservice **

我们需要通过java jdk 自带的类 Service ，
同时，我们需要依赖一个接口，这个接口我们可以通过wsimport 生成的本地代码当中获取

1. 复制接口到我们本地的包中  
2. 使用,关键类QName – 被称为完全限定名即：Qualified Name的缩写。QName 的值包含名称空间 URI、本地部分和前缀。

```
package com.zj.clent;

import java.net.MalformedURLException;
import java.net.URL;

import javax.xml.namespace.QName;
import javax.xml.ws.Service;

import com.baidu.google.soso.so.HelloServiceService;

public class ServiceInvoke {
  
  public static void main(String[] args) throws MalformedURLException {
    URL url=new URL("http://10.129.69.114:9999/helloService?wsdl");
    
    //1.命名空间2.服务名称
    QName qName=new QName("http://server.zj.com/","HelloServiceService");
    Service service=Service.create(url,qName);
     
    //获取接口类型 
    HelloService hs=service.getPort(new QName("http://server.zj.com/", "HelloServicePort"),HelloService.class);
    
    hs.doubleKill();
  
  }

}
```


/**
  互联网上有很多免费的服务,http://www.webxml.com.cn 可以在这个网站上面找到。       
**/       
         
**1,调用互联网上手机号码归属地查询的服务 **                             
使用第一种：wsimport 生成本地代码调用                
使用第二种：使用urlConnect 调用天气预报..              

**第四种调用方式：通过ajax 去调用webservice**   

  xmlhttpRequest 对象时浏览器自带的一个对象，可以通过此对象发送一个http 请求，传送xml 格式的数据到服务端
（模拟soap 协议...）                
  不能访问：跨域(a 站点的js 访问b 站点的请求...)      

由于使用ajax – js调用web服务完成不同于使用java代码调用。所以，必须要对SOAP文件非常的了解。           
一般使用ajax调用，应该是在已经获知了以下信息以后才去调用：
获知请求（request）的soap文本。           
获知响应(response)的soap文本。                   

![这里写图片描述](http://img.blog.csdn.net/20160529172156314)

### webservice 加深:

  通过webservice 的客户端与服务端的几种调用方式，通过tcp ip/monitor 监控webservice 请求的过程
  
拦截请求的数据，对数据进行分析...             
  webservice 的客户端与服务端进行交互的时候，     
  第一次通过get 请求 wsdl 的服务说明书               
  第二次通过post 的方式 请求 webservice 服务...            
  
### 理解wsdl 服务的说明书：

我们可以通过修改注解来修改wsdl 服务说明书的描述。
如果修改了说明书，则会影响wsimport 生成的本地代码....

```
@WebService
(
    targetNamespace="www.baidu.com",  
    serviceName="HelloServicePortType",
    portName="ServicePortType"
)
```

修改wsdl的内容，如命名空间，服务名等       
@WebMethod(exclude=true) 方法不再对外公开                    
@WebMethod(operationName="getUserByName") 修改方法名称       

### 修改wsdl

```
public
  @WebResult(name="date")
  String getDate(
      @WebParam(name="date")
      String date){
    
    
    
    DateFormat dateFormat=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    
    return dateFormat.format(new Date());
  }
```


### WebService和Web服务器有什么区别呢？

我们可以把WebService看作是Web服务器上应用；反过来说，Web服务器是WebService运行时所必需的容器。这就是它们的区别和联系。

使用JDK1.6发布的简单Web服务，其内部其实是使用Socket实现。可以查看：SUN公司未对外公布的API类com.sun.xml.internal.ws.transport.http.server. ServerMgr获知，请使用反编译工具。

**WebService的特点**
WebService通过HTTP POST方式接受客户的请求
WebService与客户端之间一般使用SOAP协议传输XML数据.
它本身就是为了跨平台或跨语言而设计的