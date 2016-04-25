---
layout: post
title: Struts2基础知识
date: 2016-4-25
categories: blog
tags: [java]
description: Struts2基础知识
---   

### 本文主要包括以下内容  

1. struts2常用常量的定义与意义
2. struts2处理流程
3. 拆分struts
4. 动态方法调用,使用通配符
5. 接收请求参数
6. 中文编码问题
7. 自定义类型转化器
8. 访问或添加request/session/application
9. 常用servlet对象的获取

### struts2常用常量的定义与意义  


![这里写图片描述](http://img.blog.csdn.net/20160425213329970)

![这里写图片描述](http://img.blog.csdn.net/20160425213355596)


### struts2处理流程

![这里写图片描述](http://img.blog.csdn.net/20160425213504081)


每一次请求都会创建一个新的action,所以struts2的action是线程安全的 

### 拆分struts

![这里写图片描述](http://img.blog.csdn.net/20160425213717141)

### 动态方法调用 

![这里写图片描述](http://img.blog.csdn.net/20160425213829102)

(不建议使用)动态方法调用：http://localhost:8080/struts2day02/customer/addCustomer!updateCustomer（应该执行addCustomer，使用!updateCustomer,在请求addCustomer就执行了updateCustomer）
关闭动态调用的功能：

```
struts.enable.DynamicMethodInvocation = false  


```

### 使用通配符 

```
<package name="orders" namespace="/orders" extends="mypackage">
	<action name="orders_*" class="cn.itcast.action.OrdersAction" method="{1}">
			<result type="dispatcher" name="success">/orders/success.jsp</result>
	</action>
	</package>
```  

### 接收请求参数

### 采用基本类型接收请求参数(get/post) 

在Action类中定义与请求参数同名的属性，struts2便能自动接收请求参数并赋予给同名属性。
请求路径http://localhost:8080/test/view.action?id=100

```
public class ProductAction{
private Integer id;
public void setId(Integer id){//struts2通过反射技术调用与请求参数同名的属性的settter方法来获取请求参数值
this.id = id;
}
public Integer getId(){return id;}
}
```

### 采用复合类型接收请求参数

请求路径：http://localhost:8080/test/view.action?product.id=78

```
public class ProductAction{
private Product product;
public void setProduct(Product product){this.product = product;}
public Product getProduct(){return product;}
}
```

struts2首先通过反射技术调用Product的默认构造函数创建product对象，然后再通过反射技术调用product中与请求参数同名的属性的setter方法来获取请求参数值 


### 中文乱码问题

![这里写图片描述](http://img.blog.csdn.net/20160425214317377)   

### 过滤器实现  

```
package com.zj.filter;

import java.io.IOException;
import java.util.Map;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

public class EncodeFilter implements Filter {
	private FilterConfig config=null;
	private ServletContext context=null;
	private String encode=null;

	@Override
	public void destroy() {
		// TODO 自动生成的方法存根
		

	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1,
			FilterChain arg2) throws IOException, ServletException {
		// TODO 自动生成的方法存根
		//响应乱码解决
		arg1.setCharacterEncoding(encode);
		arg1.setContentType("text/html;charset="+encode);
		//请求乱码解决
		
		arg2.doFilter(new MyHttpServletRequest((HttpServletRequest) arg0), arg1);
		

	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		// TODO 自动生成的方法存根
		this.config=arg0;
		this.context=arg0.getServletContext();
		this.encode=context.getInitParameter("encode");

	}
	
	private class MyHttpServletRequest extends HttpServletRequestWrapper
	{
		private HttpServletRequest request=null;
        private boolean isNotEncode=true;
		public MyHttpServletRequest(HttpServletRequest request) {
			super(request);
			// TODO 自动生成的构造函数存根
			this.request=request;
		}
		
		@Override
		public String getParameter(String name) {
			// TODO 自动生成的方法存根
			return getParameterValues(name)==null?null:getParameterValues(name)[0];
		}
		
		@Override
		public Map<String, String[]> getParameterMap() {
			// TODO 自动生成的方法存根
			try
			{
			if(request.getMethod().equalsIgnoreCase("post"))
			{
				request.setCharacterEncoding(encode);
				return request.getParameterMap();
			}else if(request.getMethod().equalsIgnoreCase("GET"))
			{
				Map<String,String[]> map=request.getParameterMap();
				if(isNotEncode)
				{
				for(Map.Entry<String, String[]> entry:map.entrySet())
				{
					String []vs=entry.getValue();
					for(int i=0;i<vs.length;i++)
					{
						vs[i]=new String(vs[i].getBytes("iso8859-1"),encode);
					}
				}
				isNotEncode=false;
				}
				return map;
			}else
			{
			    return request.getParameterMap();
			}
			//return super.getParameterMap();
			}catch(Exception e)
			{
				e.printStackTrace();
				throw new RuntimeException();
			}
		}
		
		@Override
		public String[] getParameterValues(String name) {
			// TODO 自动生成的方法存根
			return getParameterMap().get(name);
		}
	}

}
```

### web.xml中注册  

```
<!-- 过滤器配置开始 -->
		<filter>
			<description>全站乱码过滤器</description>
			<filter-name>EncodeFilter</filter-name>
			<filter-class>com.zj.filter.EncodeFilter</filter-class>
		</filter>
		<filter-mapping>
			<filter-name>EncodeFilter</filter-name>
			<url-pattern>/*</url-pattern>
		</filter-mapping>
```

### 解决乱码的另一方法 

```
<!-- 全站参数配置 -->
	<context-param>
		<description>全站编码配置</description>
		<param-name>encode</param-name>
		<param-value>utf-8</param-value>
	</context-param>
```



### 自定义类型转化器,解决输入Date类型的问题  


1、编写一个类，继承     
com.opensymphony.xwork2.conversion.impl.DefaultTypeConverter

2、覆盖掉其中的convertValue方法

```
public Object convertValue(Map<String, Object> context, Object value,Class toType)
			context:OGNL表达式的上下文
			value:实际的值。用户输入的都是字符串，但他是一个String数组。
			toType：目标类型
	public class DateConvertor extends DefaultTypeConverter {
		/*
		 context:ognl表达式的上下文
		 value：用户输入的值（ 保存数据时）或者模型中的属性。用户输入的值是String数组
		 toType:目标类型
		 */
		@Override
		public Object convertValue(Map<String, Object> context, Object value,
				Class toType) {
			DateFormat df = new SimpleDateFormat("yyyy/MM/dd");
			if(toType==Date.class){
				//2013/05/31----->java.util.Date 保存数据时
				String strValue = ((String[])value)[0];
				try {
					return df.parse(strValue);
				} catch (ParseException e) {
					throw new RuntimeException(e);
				}
			}else{
				//java.util.Date----->2013/05/31 获取数据时
				Date dValue = (Date)value;
				return df.format(dValue);
			}
		}
	}
```

3、注册类型转换器     
3.1局部类型转换器：只对当前的Action有效     

		具体做法：在动作类相同的包中，建立一个名称是“动作类名-conversion.properties”的配置文件，    
			文件中增加以下内容：要验证的字段=验证器的类全名         
					birthday=cn.itcast.convertor.DateConvertor      

3.2全局类型转换器：对所有的Action都有效   

		具体做法：在WEB-INF/classes目录下，建立一个名称为"xwork-conversion.properties"的配置文件，  
			文件中增加以下内容：目标类型全名=验证器的类全名         
					java.util.Date=cn.itcast.convertor.DateConvertor      
		
		
		
### 注意：如果转换失败，Struts2框架会寻找name=input的结果页面  

### 访问或添加request/session/application  

```
/**
 * 
 */
package cn.itcast.action;

import java.io.Serializable;
import java.util.Map;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

/**
 * @author wzhting
 *
 */
//域范围
public class ScopeAction extends ActionSupport implements Serializable {
	//向各大域范围存放点数据
	@Override
	public String execute() throws Exception {
		ActionContext ac = ActionContext.getContext();
		Map<String,Object> applicationMap = ac.getApplication();//这个就是ServletContext对象中维护的那个Map
		applicationMap.put("p", "application_p");// ServletContext.setAttribute(key,object);
		
		Map<String,Object> sessionMap = ac.getSession();//这个就是HttpSession对象中维护的那个Map
		sessionMap.put("p", "session_p");// HttpSession.setAttribute(key,object);
		
		ac.put("p", "request_p");//相当于ServletRequest.setAttribute(key,obj);  
		return super.execute();
	}
	
}
``` 

### 在jsp中可以得到  

applicationScope生存周期是整个应用  
sessionScope生存周期是整个会话   
requestScope生存周期是一次请求  


```
 <body>
    应用范围：${applicationScope.p}<br/>  
    会话范围：${sessionScope.p}<br/>
    请求范围：${requestScope.p}<br/>
  </body>
```


 ### 常用servlet对象的获取

 ### 方式一：通过ServletActionContext直接获取  

```
 //方式一
	public String execute1(){
		ServletContext sc = ServletActionContext.getServletContext();
		System.out.println(sc);
		ServletRequest request = ServletActionContext.getRequest();
		System.out.println(request);
		return SUCCESS;
	}
```  
 

 ### 方式二：实现指定接口，由struts框架运行时注入 

```
 public class WebObjectAction extends ActionSupport implements Serializable,ServletContextAware,ServletRequestAware,ServletResponseAware {
	private ServletContext context;
	private HttpServletRequest request;
	private HttpServletResponse response;
	
	//方式二
	public String execute2(){
		System.out.println(context);
		System.out.println(request);
		System.out.println(response);
		return SUCCESS;
	}
	public void setServletContext(ServletContext context) {//如果动作类实现了ServletContextAware接口，就会自动调用该方法
		this.context = context;
	}
	public void setServletRequest(HttpServletRequest request) {
		this.request = request;
	}
	public void setServletResponse(HttpServletResponse response) {
		this.response = response;
	}

	
	
}
```  


### 完成


