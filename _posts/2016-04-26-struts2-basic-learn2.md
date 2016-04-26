---
layout: post
title: Struts2基础知识(二)
date: 2016-4-26
categories: blog
tags: [java]
description: Struts2基础知识(二)
---   

### 本文主要包括以下内容   

1. 文件上传，多文件上传 

### 文件上传   

![这里写图片描述](http://img.blog.csdn.net/20160426123455413)

![这里写图片描述](http://img.blog.csdn.net/20160426123510734)


- 将头设置为enctype="multipart/form-data" 

```
 <body>
    <form action="${pageContext.request.contextPath}/upload/upload1.action" method="post" enctype="multipart/form-data">
    	文件：<input type="file" name="image"/><br/>
    	<input type="submit" value="上传"/>
    </form>
  </body>
```  


- 写接收处理的方法，有两种，一种是自己实现IO流，一种是使用FileUtils     

```
package cn.itcast.action;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.Serializable;

import javax.servlet.ServletContext;

import org.apache.commons.io.FileUtils;
import org.apache.struts2.ServletActionContext;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class UploadAction1 extends ActionSupport implements Serializable {
	
	private File image;//对应的就是表单中文件上传的那个输入域的名称，Struts2框架会封装成File类型的
	private String imageFileName;//   上传输入域FileName  文件名
	private String imageContentType;// 上传文件的MIME类型
	
	public File getImage() {
		return image;
	}



	public void setImage(File image) {
		this.image = image;
	}



	public String getImageFileName() {
		return imageFileName;
	}



	public void setImageFileName(String imageFileName) {
		this.imageFileName = imageFileName;
	}



	public String getImageContentType() {
		return imageContentType;
	}



	public void setImageContentType(String imageContentType) {
		this.imageContentType = imageContentType;
	}



	public String execute(){
		System.out.println(imageContentType);
		try {
			//处理实际的上传代码
			//找到存储文件的真实路径
		System.out.println(imageFileName);
			ServletContext sc = ServletActionContext.getServletContext();
			String storePath = sc.getRealPath("/files");
			//构建输入输出流
			/*OutputStream out = new FileOutputStream(storePath+"\\"+imageFileName);
			InputStream in = new FileInputStream(image);
			byte b[] = new byte[1024];
			int len = -1;
			while((len=in.read(b))!=-1){
				out.write(b, 0, len);
			}
			out.close();
		in.close();*/
			
		FileUtils.copyFile(image, new File(storePath,imageFileName));
			
			ActionContext.getContext().put("message", "上传成功！");
			return SUCCESS;
		} catch (Exception e) {
			e.printStackTrace();
			return ERROR;
		}
	}
}
```   

### 多文件上传实现  

```
package cn.itcast.action;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.Serializable;

import javax.servlet.ServletContext;

import org.apache.commons.io.FileUtils;
import org.apache.struts2.ServletActionContext;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class UploadAction2 extends ActionSupport implements Serializable {
	
	private File[] images;//对应的就是表单中文件上传的那个输入域的名称，Struts2框架会封装成File类型的
	private String[] imagesFileName;//   上传输入域FileName  文件名
	private String[] imagesContentType;// 上传文件的MIME类型
	
	

	public File[] getImages() {
		return images;
	}



	public void setImages(File[] images) {
		this.images = images;
	}



	public String[] getImagesFileName() {
		return imagesFileName;
	}



	public void setImagesFileName(String[] imagesFileName) {
		this.imagesFileName = imagesFileName;
	}



	public String[] getImagesContentType() {
		return imagesContentType;
	}



	public void setImagesContentType(String[] imagesContentType) {
		this.imagesContentType = imagesContentType;
	}



	public String execute(){
		try {
			
			if(images!=null&&images.length>0){
				ServletContext sc = ServletActionContext.getServletContext();
				String storePath = sc.getRealPath("/files");
				for(int i=0;i<images.length;i++)
					FileUtils.copyFile(images[i], new File(storePath,imagesFileName[i]));
			}
			ActionContext.getContext().put("message", "上传成功！");
			return SUCCESS;
		} catch (Exception e) {
			e.printStackTrace();
			return ERROR;
		}
	}
}
```  


### 自定义拦截器  


![这里写图片描述](http://img.blog.csdn.net/20160426141634792)

![这里写图片描述](http://img.blog.csdn.net/20160426141646844)



1、编写一个类，实现         com.opensymphony.xwork2.interceptor.Interceptor

2、主要实现public String intercept(ActionInvocation invocation) throws Exception{}方法   

```
		该方法的返回值就相当于动作的返回值
		如果调用了String result = invocation.invoke();得到了动作类的返回的值。
	public String intercept(ActionInvocation invocation) throws Exception {
		//判断用户是否登录
		HttpSession session = ServletActionContext.getRequest().getSession();
		Object obj = session.getAttribute("user");
		if(obj==null){
			return "login";
		}else{
			return invocation.invoke();//调用动作方法
		}
	}
```  


3、拦截器定义好后，一定要在配置文件中进行注册：

```
		<interceptors> 只是定义拦截器，并没有起作用 
			<interceptor name="permissionInterceptor" class="cn.itcast.interceptor.PermissionInterceptor"></interceptor>
		</interceptors>
```

4、配置文件中的动作，要通过

```
		<interceptor-ref name="permissionInterceptor"></interceptor-ref>使用该拦截器
```

注意：一旦动作中使用了自定义的拦截器，那么默认的就不起作用了。一般应该采用如下的做法：

```
		<interceptor-ref name="defaultStack"></interceptor-ref>
		<interceptor-ref name="permissionInterceptor"></interceptor-ref>
```

		
多个动作类都要使用的话，可以通过package来进行组合。

```
<package name="mypackage" extends="struts-default">
		<interceptors> <!--  只是定义拦截器，并没有起作用 -->
			<interceptor name="permissionInterceptor" class="cn.itcast.interceptor.PermissionInterceptor"></interceptor>
			<interceptor-stack name="mydefaultstack">
				<interceptor-ref name="defaultStack"></interceptor-ref>
				<interceptor-ref name="permissionInterceptor"></interceptor-ref>
			</interceptor-stack>
		</interceptors>
		<!-- 配置全局错误结果 :范围只是本包-->
		<global-results>
			<result type="dispatcher" name="error">/customer/error.jsp</result>
		</global-results>
		
	</package>

<package name="interceptor" extends="mypackage">
		<action name="visitIndex" class="cn.itcast.action.VisitAction" method="execute">
			<interceptor-ref name="mydefaultstack"></interceptor-ref>
			
			<result name="success">/index.jsp</result>
			<result name="login">/login.jsp</result>
		</action>
	</package>
```