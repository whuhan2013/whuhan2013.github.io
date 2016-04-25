---
layout: post
title: Struts2入门
date: 2016-4-25
categories: blog
tags: [java]
description: Struts2入门
---   

### 搭建Struts2的开发环境  

1. 找到所需的jar包：发行包的lib目录中（不同版本需要的最小jar包是不同的，参见不同版本的文档。本文使用的是2.1.7）


        struts2-core.jar  核心jar包

		xwork-2.jar  xwork核心jar包
		
		ognl.jar  ognl表达式
		
		freemarker.jar  FreeMarker模板
		
		commons-logging.jar  日志
		
		commons-fileupload.jar  文件上传，2.1.6之后必须加入此包
		
		commons-io.jar  文件上传依赖的包  

2. 建立一个名称为struts.xml的配置文件，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN"
	"http://struts.apache.org/dtds/struts-2.1.7.dtd">
<struts>

</struts>
```

3. 配置核心控制器，就是一个过滤器   

```
		<filter>
			<filter-name>struts2</filter-name>
			<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
		</filter>
		<filter-mapping>
			<filter-name>struts2</filter-name>
			<url-pattern>/*</url-pattern>
		</filter-mapping>
```


### 如果TOmcat启动成功，没有报错，证明环境搭建成功！  


### 开发第一个Struts2案例

1、编写struts.xml配置文件

```
		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE struts PUBLIC
			"-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN"
			"http://struts.apache.org/dtds/struts-2.1.7.dtd">
		<struts><!--这是Struts2配置文件的根元素-->
			<package name="itcast" namespace="/test" extends="struts-default">
			<!--
			pageckage:方便管理动作元素
				name：必须有。包的名称，配置文件中必须保证唯一。
				namespace：该包的名称空间，一般是以"/"开头
				extends:集成的父包的名称。struts-default名称的包是struts2框架已经命名好的一个包。（在struts2-core.jar中有一个struts-default.xml中）
				abstract：是否是抽象包。没有任何action元素的包就是抽象包（java类）
				
			-->
				<action name="helloworld" class="cn.itcast.action.HelloWorldAction" method="sayHello">
				<!--
				action:代表一个请求动作
					name：同包中必须唯一。动作的名称
					class:负责处理的JavaBean的类全名
					method：JavaBean中的对应处理方法。（动作方法：特点是，public String 方法名(){}）
				-->
					<result name="success">/1.jsp</result>
					<!--
					result:结果类型
						name:动作方法返回的字符串
						主体内容：View的具体地址。
					-->
				</action>
			</package>
		</struts>
```   


2、根据配置文件，创建需要的javabean和对应的动作方法，	在动作方法中完成你的逻辑调用。


```	
	package cn.itcast.action;
	
	public class HelloWorldAction implements Serializable {
		private String message;

		public String getMessage() {
			return message;
		}

		public void setMessage(String message) {
			this.message = message;
		}
		public String sayHello(){
			message = "helloworld by struts2";
			return "success";
		}
	}
```

3、编写View，显示结果  

```
	 ${message}
```

	 
4、访问helloworld动作的方式：
http://localhost:8080/struts2day01/test/helloworld   应用名称/包的名称空间/动作的名称     

默认情况下：访问动作名helloworld，可以直接helloworld，或者helloworld.action
		
### 搜索顺序

```
http://localhost:8080/struts2day01/test/a/b/c/helloworld
			/test/a/b/c:名称空间
			helloworld：动作名称
			
	搜索顺序：名称空间
			/test/a/b/c  没有helloworld
			/test/a/b	 没有helloworld
			/test/a      没有helloworld
			/test        有了，调用执行
```  

### Struts2配置文件的详解  


1、struts.xml配置文件编写是没有提示的问题？      
	方法一：上网即可         
	方法二：      
		1、拷贝http://struts.apache.org/dtds/struts-2.1.7.dtd地址       
		2、Eclipse的window、preferences，搜索XML Catelog         
		3、点击add按钮                     
			Location：dtd文件的路径                
			Key Type:URI                  
			Key:http://struts.apache.org/dtds/struts-2.1.7.dtd        

2、Struts配置文件中的各种默认值。       

action:                        
	class:默认值是com.opensymphony.xwork2.ActionSupport        
			常量： 	SUCCESS   success       
					   NONE	none              
					   ERROR	error                 
				    	INPUT	input                        
					  LOGIN	login                          
	method：默认值是public String execute(){}                

	实际开发中：自己编写的动作类一般情况下继承com.opensymphony.xwork2.ActionSupport  
	
result：    
	type：转到目的地的方式。默认值是转发,名称是dispatcher        
		（注：type的取值是定义好的，不是瞎写的。在struts-default.xml中的package中有定义）
		
```
		<result-type name="chain" class="com.opensymphony.xwork2.ActionChainResult"/>
		<result-type name="dispatcher" class="org.apache.struts2.dispatcher.ServletDispatcherResult" default="true"/>
		<result-type name="freemarker" class="org.apache.struts2.views.freemarker.FreemarkerResult"/>
		<result-type name="httpheader" class="org.apache.struts2.dispatcher.HttpHeaderResult"/>
		<result-type name="redirect" class="org.apache.struts2.dispatcher.ServletRedirectResult"/>
		<result-type name="redirectAction" class="org.apache.struts2.dispatcher.ServletActionRedirectResult"/>
		<result-type name="stream" class="org.apache.struts2.dispatcher.StreamResult"/>
		<result-type name="velocity" class="org.apache.struts2.dispatcher.VelocityResult"/>
		<result-type name="xslt" class="org.apache.struts2.views.xslt.XSLTResult"/>
		<result-type name="plainText" class="org.apache.struts2.dispatcher.PlainTextResult" />
```  

		
1. dispatcher：普通的转发到某个页面
2. chain：普通的抓发到某个动作名称
3. redirect：重定向到一个页面
4. redirectAction:重定向到一个动作名称
5. plainText：以纯文本的形式输出JSP内容


### result元素的写法：

方式一：

```
			<result type="chain" name="success">a2</result>
```

方式二：

```
			<result type="chain" name="success">
				<param name="actionName">a2</param><!--name对应的chain的处理器中的setActionName()方法-->
			</result>
```

			
### 注意：如果要转向的是在另外一个名称空间的动作，那么只能使用方式二  

```
			<package name="p1" namespace="/namespace1" extends="struts-default">
				<action name="a2">
					<result type="dispatcher" name="success">/3.jsp</result>
				</action>
			</package>
			<package name="p2" namespace="/namespace2" extends="struts-default">
				<action name="a1">
					<result type="chain" name="success">
						<param name="namespace">/namespace1</param>
						<param name="actionName">a2</param>
					</result>
				</action>
			</package>
```  



### 各种result使用示例  


```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN"
	"http://struts.apache.org/dtds/struts-2.1.7.dtd">
<struts>
    <constant name="struts.devMode" value="true"></constant>
    <constant name="struts.action.extension" value="action,,do"></constant>
	<package name="itcast" namespace="/test" extends="struts-default">
		<action name="helloworld" class="cn.itcast.action.HelloWorldAction" method="sayHello">
			<result name="success">/1.jsp</result>
		</action>
		<action name="visit2jsp">
			<result type="redirect" name="success">/2.jsp</result>
		</action>
	</package>
	
	<package name="p1" namespace="/namespace1" extends="struts-default">
	    <!--  <action name="a1">
			<result type="chain" name="success">a2</result>
		</action>-->
		<action name="a2">
			<result type="dispatcher" name="success">/3.jsp</result>
		</action>
	</package>
	
	<package name="p2" namespace="/namespace2" extends="struts-default">
		<action name="a1">
			<result type="chain" name="success">
				<param name="namespace">/namespace1</param>
				<param name="actionName">a2</param>
			</result>
		</action>
	</package>
	
	<package name="p3" namespace="/ns3" extends="struts-default">
		<action name="a3">
			<result type="redirectAction" name="success">
				<param name="namespace">/ns4</param>
				<param name="actionName">a4</param>
			</result>
		</action>
	</package>
	
	<package name="p4" namespace="/ns4" extends="struts-default">
		<action name="a4">
			<result type="dispatcher" name="success">/4.jsp</result>
		</action>
	</package>
	
	<package name="p5" namespace="/ns5" extends="struts-default">
		<action name="a5">
			<result type="plainText" name="success">/5.jsp</result>
		</action>
	</package>
	
	<!-- 动态的给Action类的属性赋值 -->
	<package name="p6" namespace="/ns6" extends="struts-default">
		<action name="a6" class="cn.itcast.action.HelloWorldAction">
			<param name="message">you are big shit!</param>
			<result name="success">/6.jsp</result>
		</action>
	</package>
	<!-- 获取Action的值 -->
	<package name="p7" namespace="/ns7" extends="struts-default">
		<action name="a7" class="cn.itcast.action.HelloWorldAction" method="sayHello">
			<result type="redirect" name="success">/7.jsp?msg=${message}</result>
		</action>
	</package>
	
	
</struts>
```   

其中第6个动态的给Action的属性赋值与第7个获得action的值比较值得注意

### 6.jsp 

```
<body>
    6666:${message}
  </body>
```   

### 7.jsp

```
<body>
    7777:${param.msg}
  </body>
```  

### 开发中配置文件的更改，在访问时让框架自动重新加载：

struts.devMode = false（default.properties）      
利用strutx.xml中的constant元素来覆盖掉default.properties默认行为

```
<struts>
	<constant name="struts.devMode" value="true"></constant>
</struts>
```

### 控制action的后缀名 

```
<constant name="struts.action.extension" value="action,,do"></constant>
```   


### 完成


