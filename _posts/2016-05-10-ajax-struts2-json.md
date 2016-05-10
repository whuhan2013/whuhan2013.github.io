---
layout: post
title: struts2+ajax+json使用实例
date: 2016-5-10
categories: blog
tags: [javascript]
description: struts2+ajax+json使用实例
---   

### 本文主要包含一个struts2+ajax+json的使用实例  

### 步骤如下  

1.准备工作        
   ①ajax使用Jquery：jquery-1.4.2.min.js            
   ②struts2与json的依赖包：struts2-json-plugin-2.2.3.jar,json-lib                
  PS：版本可自己控制！~    
  
2.过程                
①引入json依赖包         
②编写action类            
③配置struts.xml           
④编写页面           
⑤测试               

### 参考链接  

[struts2 + ajax + json的结合使用，实例讲解 - tfy1332的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/tfy1332/article/details/9072921)

[Struts2 和 Ajax 交互 - - ITeye技术网站](http://hello-player.iteye.com/blog/998972)

[Java中：struts2+jQuery+ajax调用演示 - anyx的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/thinkscape/article/details/7467153)


### 实例 

### action类   

```
package com.zxt.action;

import java.util.ArrayList;
import java.util.List;

import com.opensymphony.xwork2.ActionSupport;

public class JsonAction extends ActionSupport {

  /**
    * 
    */
  private static final long serialVersionUID = 7443363719737618408L;
  /**
   * 姓名
   */
  private String name;
  /**
   * 身高
   */
  private String inch;
  /**
   * ajax返回结果，也可是其他类型的，这里以String类型为例
   */
  private List<PersonBean> result;

  @Override
  public String execute() throws Exception {
    // TODO Auto-generated method stub
    List<PersonBean> pList = new ArrayList<PersonBean>();
    if ("张三".equals(name)) {
      PersonBean p1 = new PersonBean();
      PersonBean p2 = new PersonBean();
      p1.setId(1);
      p1.setName("a");
      p2.setId(2);
      p2.setName("b");
      pList.add(p1);
      pList.add(p2);
      result = pList;
    } else {
      PersonBean p3 = new PersonBean();
      PersonBean p4 = new PersonBean();
      p3.setId(3);
      p3.setName("c");
      p4.setId(4);
      p4.setName("d");
      pList.add(p3);
      pList.add(p4);
      result = pList;
    }
    return SUCCESS;

  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getInch() {
    return inch;
  }

  public void setInch(String inch) {
    this.inch = inch;
  }

  /**
   * 
   * @Title: getResult
   * @Description:json调取结果
   * @param @return
   * @return String
   * @throws
   */
  public List<PersonBean> getResult() {
    return result;
  }
}
```

### struts.xml配置文件  

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
  "-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN"
  "http://struts.apache.org/dtds/struts-2.1.7.dtd">
<struts>
    <constant name="struts.devMode" value="true"/>
  <package name="ajax" extends="json-default">
      <action name="jsonAjax" class="com.zxt.action.JsonAction">
        <!-- 将返回类型设置为json -->
        <result type="json"></result>
      </action>
    </package>
</struts> 
```

### web.xml 

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0" 
  xmlns="http://java.sun.com/xml/ns/javaee" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
  http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
  <display-name></display-name> 
  <filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```

### index.jsp

```
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
  String path = request.getContextPath();
  String basePath = request.getScheme() + "://"
      + request.getServerName() + ":" + request.getServerPort()
      + path + "/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
<base href="<%=basePath%>">
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title>测试</title>
<script type="text/javascript" src="js/jquery-1.4.2.js"></script>
<script type="text/javascript">
  $(function() {
    $("#tj").click(function() {
      //提交的参数，name和inch是和struts action中对应的接收变量
  
  /*var data = {
        "list" : [ {
          "id" : 1,
          "content" : "测试信息1111"
        }, {
          "id" : 2,
          "content" : "测试信息2222"
        } ]
      };
      $.each(data.list, function(i, item) {
        alert(item.id);
        alert(item.content);
      });*/
      var params = {
        name : $("#xm").val(),
        inch : $("#sg").val()
      };
      $.ajax({
        type : "POST",
        url : "jsonAjax.action",
        data : params,
        dataType : "text", //ajax返回值设置为text（json格式也可用它返回，可打印出结果，也可设置成json）
        success : function(json) {
          var obj = $.parseJSON(json); //使用这个方法解析json
          var state_value = obj.result; //result是和action中定义的result变量的get方法对应的
          
          $.each(state_value, function(i, item) {
            alert(item.id);
            alert(item.name);
          });
          //alert(state_value);
        },
        error : function(json) {
          alert("json=" + json);
          return false;
        }
      });
    });
  });
</script>
</head>
<body>
  <span>姓名：</span>
  <input id="xm" type="text">
  <br />
  <span>身高：</span>
  <input id="sg" type="text">
  <br />
  <input type="button" value="提交" id="tj">
</body>
</html>
```

### 本例中的jsp解析了从服务器传来的list，解析list具体方法如下  

```
var data = {"list":[{"id":1,"content":"测试信息1111"},{"id":2,"content":"测试信息2222"}]}
   $.each(data.list, function(i, item) {
      alert(item.id);
      alert(item.content);
   });
```

### 完成