---
layout: post
title: JAVA求职专项练习(一)
date: 2016-7-25
categories: blog
tags: [求职]
description: 
---

1、**Thread. sleep()是否会抛出checked exception?**               
checked exception：指的是编译时异常，该类异常需要本函数必须处理的，用try和catch处理，或者用throws抛出异常，然后交给调用者去处理异常。                  
runtime exception：指的是运行时异常，该类异常不必须本函数必须处理，当然也可以处理。           
Thread.sleep()抛出的InterruptException属于checked exception；IllegalArgumentException属于Runtime exception;      

2、 **关于struts框架，下面那些说法是正确的？**               
A，structs可以进行文件上传          
B，structs基于MVC模式，MVC是模型，视图，控制器，是一种设计模式              
C，structs框架让流程结构更清晰             
D，structs需要很多 action类，会增加类文件数目                

**工作机制：**        
Struts的工作流程:                    
在web应用启动时就会加载初始化ActionServlet,ActionServlet从 
struts-config.xml文件中读取配置信息,把它们存放到各种配置对象 
当ActionServlet接收到一个客户请求时,将执行如下流程. 

(1)检索和用户请求匹配的ActionMapping实例,如果不存在,就返回请求路径无效信息;           
(2)如果ActionForm实例不存在,就创建一个ActionForm对象,把客户提交的表单数据保存到ActionForm对象中;        
(3)根据配置信息决定是否需要表单验证.如果需要验证,就调用ActionForm的validate()方法;       
(4)如果ActionForm的validate()方法返回或返回一个不包含ActionMessage的ActuibErrors对象, 就表示表单验证成功;       
(5)ActionServlet根据ActionMapping所包含的映射信息决定将请求转发给哪个Action,如果相应的Action实例不存在,就先创建这个实例,然后调用Action的execute()方法;                
(6)Action的execute()方法返回一个ActionForward对象,ActionServlet在把客户请求转发给 ActionForward对象指向的JSP组件;  
(7)ActionForward对象指向JSP组件生成动态网页,返回给客户;          

为什么要用：            
JSP、Servlet、JavaBean技术的出现给我们构建强大的企业应用系统提供了可能。但用这些技术构建的系统非常的繁乱，所以在此之上，我们需要一个规则、一个把这些技术组织起来的规则，这就是框架，Struts便应运而生。            
基于Struts开发的应用由3类组件构成：控制器组件、模型组件、视图组件                 

3、 **下面有关List接口、Set接口和Map接口的描述，错误的是？**              
List接口中的对象按一定顺序排列，允许重复        
Set接口中的对象没有顺序，但是不允许重复              
Map接口中的对象是key、value的映射关系，key不允许重复                  
Map接口和Collection接口是同一等级的，所以并不是继承自collection             

4、 **有如下4条语句：()**          
Integer i01=59;          
int i02=59;                              
Integer i03=Integer.valueOf(59);         
Integer i04=new Integer(59);            

Integer i01 = 59. 直接赋值数字，java会自动装箱，自动调用Integer.valueOf(59).    
Integer i03 = Integer.valueOf(59).  Integer.valueOf(int i)会返回一个Integer对象，当i在-128~127之间时，会返回缓存中已创建的Integer对象。               
Integer i04 = new Integer(59) 返回一个新的对象。              
所以这道题中，59在-128~127之间，所以前三条语句返回的是同一个对象（在缓存区已创建的对象），而i04使用new新创建了一个新的对象，所以i04与前面三个对象都不一样。           

5、 **HttpSession session = request.getSession(false) 与HttpSession session = request.getSession(true)的区别？**   
HttpSession session = request.getSession(boolean create)                
返回当前请求的会话。如果当前请求不属于任何会话，而且create参数为true，则创建一个会话，否则返回null。此后所有来自同一个的请求都属于这个会话，通过它的getSession返回的是当前会话。   

- equest.getSession() 等价于 request.getSession(true)             
这两个方法的作用是相同的，查找请求中是否有关联的session id，如果有则返回这个号码所对应的session对象，如果没有则生成一个新的session对象。所以说，通过此方法是一定可以获得一个session对象。
- request.getSession(false) 查找请求中是否有关联的session id，如果有则返回这个号码所对应的session对象，如果没有则返回一个null。


