---
layout: post
title: 模拟Struts2实现
date: 2016-4-24
categories: blog
tags: [java]
description: 模拟Struts2实现
---   

### 本文主要是一个模拟的Struts2的简单实现  

### 真正的MVC架构

![这里写图片描述](http://img.blog.csdn.net/20160424163431279)

### 实现主要思路

定义一个过滤器，接收传递过去的Action，根据处理的结果重定向或者转发。

### 首先定义index.jsp

```
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title></title>
    
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
	<!--
	<link rel="stylesheet" type="text/css" href="styles.css">
	-->

  </head>
  
  <body>
    <form action="${pageContext.request.contextPath}/addCustomer.action" method="post">
    	姓名:<input type="text" name="name"/><br/>
    	年龄:<input type="text" name="age"/><br/>
    	<input type="submit" value="保存"/>
    </form>
  </body>
</html>
```  

### 在web.xml中配置过滤器

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>MyStruct2</display-name>
  
  <filter>
		<filter-name>CenterFilter</filter-name>
		<filter-class>cn.itcast.framework.core.CenterFilter</filter-class>
<!--		<init-param>-->
<!--			<param-name>aciontPostFix</param-name>-->
<!--			<param-value>action,do</param-value>-->
<!--		</init-param>-->
	</filter>
	<filter-mapping>
		<filter-name>CenterFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```   


### 核心配置文件  

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- 核心配置文件 -->
<framework>
	<!-- 
	action:
		name:必须有，且唯一。如同之前请求的什么operation
		class：必须有，要执行哪个JavaBean的类名
		method:可以没有。没有的话就是默认值execute。JavaBean中对应的操作方法。该方法的特点是：public String addCustomer(){}
	 -->
	<action name="addCustomer" class="cn.itcast.domain.Customer" method="addCustomer">
		<!-- 
		result:代表着一种结果。
			type:可以没有，默认是dispatcher。转向目标的类型。dispatcher代表着请求转发
			name:必须有。对应的是JavaBean中addCustomer的返回值
			主体内容：必须有。目标资源的URI
		-->
		<result type="redirect" name="success">/success.jsp</result>
		<result type="dispatcher" name="error">/error.jsp</result>
	</action>
	<action name="updateCustomer" class="cn.itcast.domain.Customer" method="updateCustomer">
		<result type="dispatcher" name="success">/success.jsp</result>
	</action>
</framework>
```   


### 核心过滤器实现   

1. 初始化配置文件,读取XML配置文件：把配置文件中的信息封装到对象中.再放到actions中    
2. 取配置参数，如果你请求的地址以action\do\空结尾的话，才真正过滤。  
3. 解析用户请求的URI，截取后缀名，看看是否需要我们的框架进行处理
4. 如果需要框架处理，解析uri中的动作名称，获得相应方法处理
5. 根据结果中的type决定是转发还是重定向，重定向的目标就是结果中的targetUri


{% highlight java %}
package cn.itcast.framework.core;

import java.io.IOException;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.beanutils.BeanUtils;
import org.dom4j.Document;
import org.dom4j.Element;

import cn.itcast.framework.util.Dom4JUtil;

public class CenterFilter implements Filter {
	//存取配置文件信息。key：对应action中的name value：Action对象
	private Map<String, Action> actions = new HashMap<String, Action>();
	private FilterConfig filterConfig;
	
	public void init(FilterConfig filterConfig) throws ServletException {
		initCfg();//初始化配置文件
		this.filterConfig = filterConfig;
		
	}
	//初始化配置文件
	private void initCfg() {
		//读取XML配置文件：把配置文件中的信息封装到对象中.再放到actions中
		
		Document document = Dom4JUtil.getDocument();
		Element root = document.getRootElement();
		//得到所有的action元素，创建Action对象，封装信息
		List<Element> actionElements = root.elements("action");
		if(actionElements!=null&&actionElements.size()>0){
			for(Element actionElement:actionElements){
				//封装action信息开始
				Action action = new Action();
				action.setName(actionElement.attributeValue("name"));
				action.setClassName(actionElement.attributeValue("class"));
				String methodXmlAttrValue = actionElement.attributeValue("method");
				if(methodXmlAttrValue!=null)
					action.setMethod(methodXmlAttrValue);
				//封装action信息结束
				
				
				//得到每个action元素中的result元素，创建Result对象，封装信息
				//封装属于当前action的结果
				List<Element> resultElements = actionElement.elements("result");
				if(resultElements!=null&&resultElements.size()>0){
					for(Element resultElement:resultElements){
						Result result = new Result();
						result.setName(resultElement.attributeValue("name"));
						result.setTargetUri(resultElement.getText().trim());
						String typeXmlValue = resultElement.attributeValue("type");
						if(typeXmlValue!=null){
							result.setType(ResultType.valueOf(typeXmlValue));
						}
						action.getResults().add(result);
					}
				}
				//封装属于当前action的结果
				
				
				
				//把Action对象都放到Map中
				actions.put(action.getName(), action);
			}
		}
	}
	public void doFilter(ServletRequest req, ServletResponse resp,
			FilterChain chain) throws IOException, ServletException {
		try {
			HttpServletRequest request = (HttpServletRequest)req;
			HttpServletResponse response = (HttpServletResponse)resp;
			//真正的控制器部分
			
			//取一个配置参数
			String aciontPostFixs [] = {"action","","do"};//你请求的地址以action\do\空结尾的话，才真正过滤。默认值
			String aciontPostFix = filterConfig.getInitParameter("aciontPostFix");
			if(aciontPostFix!=null){
				aciontPostFixs = aciontPostFix.split("\\,");
			}
			
			//解析用户请求的URI
			String uri = request.getRequestURI();//   /strutsDay01MyFramework/addCustomer.action
			
			//截取后缀名，看看是否需要我们的框架进行处理
			//切后缀名
			String extendFileName = uri.substring(uri.lastIndexOf(".")+1);// /strutsDay01MyFramework/addCustomer.action   action
																			// /strutsDay01MyFramework/addCustomer.do   do
																			// /strutsDay01MyFramework/addCustomer   ""
			boolean needProcess = false;
			for(String s:aciontPostFixs){
				if(extendFileName.equals(s)){
					needProcess = true;
					break;
				}
			}
			
			if(needProcess){
				//需要框架处理
				//解析uri中的动作名称
				String requestActionName = uri.substring(uri.lastIndexOf("/")+1, uri.lastIndexOf("."));
				System.out.println("您的请求动作名是："+requestActionName);
				//查找actions中对应的Action对象
				if(actions.containsKey(requestActionName)){
					Action action = actions.get(requestActionName);
					//得到类名称的字节码
					Class clazz = Class.forName(action.getClassName());
					//封装数据到JavaBean中,利用BeanUtils框架
					Object bean = clazz.newInstance();
					BeanUtils.populate(bean, request.getParameterMap());
					//实例化，调用其中指定的方法名称
					Method m = clazz.getMethod(action.getMethod(), null);
					//根据方法的返回值，遍历结果
					String resultValue = (String)m.invoke(bean, null);
					
					List<Result> results = action.getResults();
					if(results!=null&&results.size()>0){
						for(Result result:results){
							
							if(resultValue.equals(result.getName())){
								//根据结果中的type决定是转发还是重定向
								//重定向的目标就是结果中的targetUri
								if("dispatcher".equals(result.getType().toString())){
									//转发
									request.getRequestDispatcher(result.getTargetUri()).forward(request, response);
								}
								if("redirect".equals(result.getType().toString())){
									//重定向
									response.sendRedirect(request.getContextPath()+result.getTargetUri());
								}
							}
						}
					}
				}else{
					throw new RuntimeException("The action "+requestActionName+" is not founded in your config files!");
				}
				
			}else{
				chain.doFilter(request, response);
			}
		} catch (Exception e) {
			throw new RuntimeException(e);
		}
		
	}

	public void destroy() {

	}

}
{% endhighlight %} 

### javabean如下

{% highlight java %}
package cn.itcast.domain;

import java.io.Serializable;

public class Customer implements Serializable {
	private String name;
	private int age;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public String addCustomer(){
		if("wzhting".equals(name)){
			return "success";
		}else{
			return "error";
		}
	}
}
{% endhighlight %}



### 完成











