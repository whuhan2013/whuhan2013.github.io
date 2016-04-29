---
layout: post
title: Struts2之Crud综合实例
date: 2016-4-29
categories: blog
tags: [java]
description: Struts2之Crud综合实例
---   

### 本文是Struts2的综合实例，主要包含以下功能  

1. 添加，删除，修改，查询用户
2. 上传，下载图片  
3. 拦截器实现登陆功能 
4. 验证器检查输入 


### 下载图片功能以前没有实现过，步骤如下    

- 在类中增加两个属性  

```
//文件下载
    private InputStream inputStream;
    private String imageFileName;
```  

- 下载方法实现  

```
public String download(){
        path = ServletActionContext.getRequest().getParameter("path");
        filename = ServletActionContext.getRequest().getParameter("filename");
        
        String storePath = ServletActionContext.getServletContext().getRealPath("/files");
        
        try {
            inputStream = new FileInputStream(storePath+"\\"+path+"\\"+filename);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        
        return SUCCESS;
    }
```  

### 配置struts.xml  

```
<action name="download" class="cn.itcast.domain.User" method="download">
            <interceptor-ref name="mydefaultstack"></interceptor-ref>
            <result type="stream" name="success">
                <param name="contentType">application/octet-stream</param>
                <param name="inputStream">inputStream</param><!-- 输入是对应的动作类中的那个字段 -->
                <param name="contentDisposition">attachment;filename=${filename}</param><!-- 要下载的文件名 -->
            </result>
            <result name="login">/login.jsp</result>
        </action>
```  

### JSP页面实现  

```
<c:url value="/user/download" var="url">
            <c:param name="path" value="${user.path}"></c:param>
            <c:param name="filename" value="${user.filename}"></c:param>
        </c:url>
        <a href="${url}">下载</a>
```


### 登陆验证功能  

### 定义拦截器类   

```
package cn.itcast.interceptor;

import javax.servlet.http.HttpSession;

import org.apache.struts2.ServletActionContext;

import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.interceptor.Interceptor;

public class PermissionInterceptor implements Interceptor {

    public void destroy() {

    }

    public void init() {

    }

    public String intercept(ActionInvocation invocation) throws Exception {
        HttpSession session = ServletActionContext.getRequest().getSession();
        Object obj = session.getAttribute("user");
        if(obj==null){
            return "login";
        }else{
            return invocation.invoke();
        }
    }

}
```  


### struts.xml中的配置  

```
<package name="mydefault" extends="struts-default">
        <interceptors>
            <interceptor name="permissionInterceptor" class="cn.itcast.interceptor.PermissionInterceptor"></interceptor>
            <interceptor-stack name="mydefaultstack">
                <interceptor-ref name="defaultStack"></interceptor-ref>
                <interceptor-ref name="permissionInterceptor"></interceptor-ref>
            </interceptor-stack>
        </interceptors>
    </package>
```  

### 在其他Action中使用拦截 

```
<interceptor-ref name="mydefaultstack"></interceptor-ref>
```  

### 条件查询实现  

### JSP页面  

```
<td>
                    <s:form action="user_queryCondition" namespace="/user">
                        <s:textfield name="username" label="用户名"></s:textfield>
                        <s:select list="#{'0':'女','1':'男'}" label="性别" name="sex" headerKey="" headerValue="请选择"></s:select>
                        <s:select label="学历" name="education" list="#{'研究生':'研究生','本科':'本科','专科':'专科','高中':'高中'}" headerKey="" headerValue="请选择"></s:select>
                        <s:submit value="查询"></s:submit>
                    </s:form>
                </td>
```  

### javabean方法实现  

```
public String queryCondition(){
        List<User> users =s.findUserByCondition(this);
        ActionContext.getContext().put("users", users);// #users
        return SUCCESS;
    }
```


### service层方法实现   

```
public List<User> findUserByCondition(User user) {
        boolean ok1 = true;
        boolean ok2 = true;
        boolean ok3 = true;
        StringBuffer sb = new StringBuffer("where 1=1 ");
        
        
        if(user.getUsername()!=null&&!user.getUsername().equals("")){
            ok1 = false;
            sb.append(" and username like '%"+user.getUsername()+"%' ");
        }
        if(user.getSex()!=null&&!user.getSex().equals("")){
            ok2 = false;
            sb.append(" and sex='"+user.getSex()+"'");
        }
            
        if(user.getEducation()!=null&&!user.getEducation().equals("")){
            ok3 = false;
            sb.append(" and education='"+user.getEducation()+"'");
        }
        
        boolean conditionOk = ok1&&ok2&&ok3;//如果为false，说明至少有一个查询条件
        if(conditionOk){
//          System.out.println("没有查询条件");
//          return null;
            return dao.findUsersByCondition(null);
        }else{
//          System.out.println("有查询条件");
//          System.out.println(sb.toString());
//          return null;
            return dao.findUsersByCondition(sb.toString());
        }
    }
```

### 其他增删改查详细实例参见github源码   

[ssh/Struts2Crud at master · whuhan2013/ssh](https://github.com/whuhan2013/ssh/tree/master/Struts2Crud)

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160429192055660)