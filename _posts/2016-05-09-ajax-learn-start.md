---
layout: post
title: AJAX入门
date: 2016-5-09
categories: blog
tags: [javascript]
description: AJAX入门
---   

### 编写步骤                     
    1、测试与服务器的通信            
      a、创建XmlHttpRequest对象，固定写法：  

```
          function createXmlHttpRequest(){
             var xmlHttp;
             try{    //Firefox, Opera 8.0+, Safari
                 xmlHttp=new XMLHttpRequest();
            }catch (e){
                 try{    //Internet Explorer
                    xmlHttp=new ActiveXObject("Msxml2.XMLHTTP");
                }catch (e){
                    try{
                        xmlHttp=new ActiveXObject("Microsoft.XMLHTTP");
                    }catch (e){}  
                 }
            }
             return xmlHttp;
           }
           
var xhr = createXmlHttpRequest();
```
           
b、注册状态变化的事件处理：    

```
        xhr.onreadystatechange=function(){
          if(xhr.readyState==4){
            //真正的处理
            if(xhr.status==200||xhr.status==304){
              //服务器正确返回
              var data = xhr.responseText;//服务器返回的数据
              //把返回的数据写到div中
              document.getElementById("d1").innerHTML=data;
            }
          }
        }
```

c、初始化xhr对象

```
xhr.open("GET","/ajaxday02/servlet/ServletDemo1?time="+new Date().getTime());
```

d、向服务器发送数据              
        xhr.send(null);            
        

### XmlHttpRequest详解（JavaScript对象）   

```
    常用属性：
      readyState：代表着XmlHttpRequest对象的当前状态
        0 (未初始化) 对象已建立，但是尚未初始化（尚未调用open方法） 
        1 (初始化) 对象已建立，尚未调用send方法 
        2 (发送数据) send方法已调用，但是当前的状态及http头未知 
        3 (数据传送中) 已接收部分数据，因为响应及http头不全，
        4 (完成) 数据接收完毕,此时可以通过通过responseBody和responseText获取完整的回应数据 
      只有为4，客户端操作相应的处理
      -------------------------------------------------
      status：代表服务器的HTTP相应码。200是成功。304服务器端内容没有改变。
      -------------------------------------------------
      responseText：服务器返回文本数据。
      
      onreadystatechange：当XmlHttpRequest对象的readyState发生变化时，都会触发该事件。
```
      
### 常用方法：

```
      open(method,url,isAsync):初始化XmlHttpRequest对象。
        method：请求方式。一般使用get或者post
        url：请求的服务器地址。可以使用相对路径或者绝对路径。
            特别注意：如果该地址没有变化，浏览器一般不会再次发出请求的。解决办法，加上一个时间戳。
              /ajaxday02/servlet/ServletDemo1?time="+new Date().getTime()
        isAsync：是否是异步请求。默认是true。
      send(requestData):向服务器发送请求数据。没有传递null。
        数据时用在POST请求方式的。数据形式：username=admin&password=123
        
    通过XmlHttpRequest向服务器发送POST请求：
      //设置请求消息头，告知服务器，发送的正文数据的类型。
        xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded");//固定写法
        //发送数据
        xhr.send("username=admin&password=123");
```

4.4服务端返回的数据

- HTML数据
- responseText：他是XmlHttpRequest对象的一个属性。服务器返回的数据会封装到此属性中。
- XML数据,responseXML:返回的是xml对象的DOM对象。

### 实例  

### get方式与服务器交互  

```
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>AJAX进行get方式的请求测试</title>
</head>
<body>
  <input type="button" id="b1" name="b1" value="测试与服务器的通信"/>
  <div id="d1">
        
  </div>
  <script type="text/javascript">
    
    window.onload=function(){//当页面被全部加在完毕后再执行
      //给b1按钮注册事件
      document.getElementById("b1").onclick=function(){
        //AJAX代码
        
        //得到XmlHttpRequest对象
        var xhr = createXmlHttpRequest();
        //xhr的readyState改变都会触发onreadystatechange事件
          
          /*
           * readyState的取值：
           *  0 (未初始化) 对象已建立，但是尚未初始化（尚未调用open方法） 
           *  1 (初始化) 对象已建立，尚未调用send方法 
           *  2 (发送数据) send方法已调用，但是当前的状态及http头未知 
           *  3 (数据传送中) 已接收部分数据，因为响应及http头不全， 
           *  4 (完成) 数据接收完毕,此时可以通过通过responseBody和responseText获取完整的回应数据 

           */
        xhr.onreadystatechange=function(){
          if(xhr.readyState==4){
            //真正的处理
            if(xhr.status==200||xhr.status==304){
              //服务器正确返回
              var data = xhr.responseText;//服务器返回的数据
              //把返回的数据写到div中
              document.getElementById("d1").innerHTML=data;
            }
          }
        }
        //建立与服务器的连接
          /*
           * oXMLHttpRequest.open(bstrMethod, bstrUrl, varAsync, bstrUser, bstrPassword);
           * bstrMethod:请求方式。一般使用get或post
           * bstrUrl:请求的资源地址：可以绝对或相对路径
           * varAsync:是否是异步请求。默认是true。
           */
        xhr.open("GET","http://localhost:8080/Test/servlet/ServletDemo1?time="+new Date().getTime());
        //发送数据
          //oXMLHttpRequest.send(varBody);  varBody:请求参数
        xhr.send(null);

        //接收服务器返回的数据
          
        
      }
    }
    function createXmlHttpRequest(){
       var xmlHttp;
       try{    //Firefox, Opera 8.0+, Safari
               xmlHttp=new XMLHttpRequest();
        }catch (e){
               try{    //Internet Explorer
                      xmlHttp=new ActiveXObject("Msxml2.XMLHTTP");
                }catch (e){
                      try{
                              xmlHttp=new ActiveXObject("Microsoft.XMLHTTP");
                      }catch (e){}  
               }
        }
       return xmlHttp;
     }

  </script>
</body>
</html>
```

### 服务器端代码  

```
package cn.itcast.controller;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/servlet/ServletDemo1")
public class ServletDemo1 extends HttpServlet {
  private static final long serialVersionUID = 1L;
       
   
  public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    response.setContentType("text/html;charset=UTF-8");
    PrintWriter out = response.getWriter();
    out.write("hello ajax");
    System.out.println("ServletDemo1执行了");
  }

  public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    doGet(request,response);
  }

}
```

### post方式与服务器交互  

```
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>AJAX进行post方式的请求测试</title>
</head>
<body>
  <input type="button" id="b1" name="b1" value="测试与服务器的通信"/>
  <div id="d1">
        
  </div>
  <script type="text/javascript">
    
    window.onload=function(){
      document.getElementById("b1").onclick = function(){
        //获取xmlhttpRequest对象
        var xhr = createXmlHttpRequest();
        //注册状态变化的回调函数
        xhr.onreadystatechange = function(){
          if (xhr.readyState == 4) {
            if (xhr.status == 200 || xhr.status == 304) {
            //什么都不做
            }
          }
        }
        //初始化xmlhttpRequest对象，即open
        xhr.open("POST", "/Test/servlet/ServletDemo2?time=" + new Date().getTime());
        //设置请求消息头，告知服务器，发送的正文数据的类型
        //不设置这句无法接收数据   
        xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
        //发送数据
        xhr.send("username=admin&password=123");
      }
    }
    function createXmlHttpRequest(){
       var xmlHttp;
       try{    //Firefox, Opera 8.0+, Safari
               xmlHttp=new XMLHttpRequest();
        }catch (e){
               try{    //Internet Explorer
                      xmlHttp=new ActiveXObject("Msxml2.XMLHTTP");
                }catch (e){
                      try{
                              xmlHttp=new ActiveXObject("Microsoft.XMLHTTP");
                      }catch (e){}  
               }
        }
       return xmlHttp;
     }

  </script>
</body>
</html>
```

### 服务器代码  

```
package cn.itcast.controller;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Servlet implementation class ServletDemo2
 */
@WebServlet("/servlet/ServletDemo2")
public class ServletDemo2 extends HttpServlet {
  private static final long serialVersionUID = 1L;
       
   
  protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // TODO Auto-generated method stub
    response.setContentType("text/html;charset=UTF-8");
    PrintWriter out = response.getWriter();
    out.write("hello abc");
    System.out.println("ServletDemo2执行了");
  }

  
  protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // TODO Auto-generated method stub
    System.out.println("ServletDemo2 doPost running");
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println(username+":"+password);
  }

}
```           
### 问题  

eclipse启动tomcat, http://localhost:8080无法访问 

### 参考链接 

[eclipse启动tomcat, http://localhost:8080无法访问 - 海邻网络IT技术网 - 博客频道 - CSDN.NET](http://blog.csdn.net/ji_ju/article/details/8545588)


AJAX问题之XMLHttpRequest status = 0


好像是因为在本地打开与在服务器打开的HTML的路径是不一样的        
### 参考链接

[AJAX问题之XMLHttpRequest status = 0 - iaiti的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/iaiti/article/details/42192659)

### 完成