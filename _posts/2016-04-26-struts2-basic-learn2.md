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
2. 自定义拦截器
3. 用户输入验证
4. 国际化

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

### 用户输入验证  

### 1、编程方式：


		
```
动作类中的所有方法进行验证：
			步骤：
			a、动作类继承ActionSupport
			b、覆盖调用public void validate()方法
			c、在validate方法中，编写不符合要求的代码判断，并调用父类的addFieldError(String fieldName,String errorMessage)
				如果fieldError（存放错误信息的Map）有任何的元素，就是验证不通过，动作方法不会执行。
				Struts2框架会返回到name=input的result
			d、在name=input指定的页面上使用struts2的标签显示错误信息。<s:fielderror/>
		
		动作类中指定的方法进行验证：
			编写步骤与上面相同
			
			验证方法书写有要求：
				public void validateXxx()   Xxx代表的是要验证的动作方法名，其中要把动作方法名的首字母变为大写。
```  


### 2、基于XML配置文件的方式：  

```
动作类中的所有方法进行验证：
			在动作类的包中，建立一个名称为：动作简单类名-validation.xml ，比如要验证的动作类名是UserAction   UserAction-validation.xml
			内容如下：
			<?xml version="1.0" encoding="UTF-8"?>
			<!DOCTYPE validators PUBLIC
					"-//OpenSymphony Group//XWork Validator 1.0.3//EN"
					"http://www.opensymphony.com/xwork/xwork-validator-1.0.3.dtd">
			<validators>
				<field name="username">
					<!-- 内置验证器都是定义好的，在xwork-core.jar com.opensymphony.xwork2.validator.validators包中的default.xml文件中 -->
					<field-validator type="requiredstring"><!-- 不能为null或者""字符串，默认会去掉前后的空格 -->
						<message>用户名不能为空</message>
					</field-validator>
				</field>
			</validators>
		动作类中指定的方法进行验证：
			配置文件的名称书写有一定要求。
					动作类名-动作名（配置文件中的动作名）-validation.xml
					UserAction-user_add-validation.xml
``` 

### 3、自定义基于XML的验证器

```
a、编写一个类，继承FieldValidatorSupport类。
		b、在public void validate(Object object)编写你的验证逻辑
				不符合要求的就向fieldErrors中放消息
		c、一定注册你的验证器才能使用
				在WEB-INF/classes目录下建立一个名称为validators.xml的配置文件，内容如下：
				<validators>
					<validator name="strongpassword" class="cn.itcast.validators.StrongPasswordValidator"/>
				</validators>
		d、日后就可以像使用Struts2提供的16个验证器方式去使用了。
```   

### 实例  

```
package cn.itcast.validators;

import com.opensymphony.xwork2.validator.ValidationException;
import com.opensymphony.xwork2.validator.validators.FieldValidatorSupport;

public class StrongPasswordValidator extends FieldValidatorSupport {

	public void validate(Object object) throws ValidationException {
		String fieldName = getFieldName();//取得字段名
		String filedValue = (String)getFieldValue(fieldName, object);//取得用户输入的当前字段的值
		if(!isPasswordStrong(filedValue)){
			addFieldError(fieldName, object);
		}
	}
	private static final String GROUP1 = "abcdefghijklmnopqrstuvwxyz";
	private static final String GROUP2 = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
	private static final String GROUP3 = "0123456789";
	protected boolean isPasswordStrong(String password) {
		boolean ok1 = false;
		boolean ok2 = false;
		boolean ok3 = false;
		int length = password.length();
		for(int i=0;i<length;i++){
			if(ok1&&ok2&&ok3)
				break;
			String character = password.substring(i,i+1);
			if(GROUP1.contains(character)){
				ok1 = true;
				continue;
			}
			if(GROUP2.contains(character)){
				ok2 = true;
				continue;
			}
			if(GROUP3.contains(character)){
				ok3 = true;
				continue;
			}
		}
		return ok1&&ok2&&ok3;
	}
}
```


### Struts2对于i18n的支持

```
全局资源文件/包范围资源文件/动作类的资源文件
	全局资源文件：放到WEB-INF/classes目录下
	包范围资源文件：服务于Java类中的包下的动作类的。  
	
			取名：package_语言_国家.properties
	
	动作类的资源文件:放到与动作类相同的包中
			取名：动作类名_语言_国家.properties
```
			
jsp中如何读取国际化的消息

```
<body>
    <s:i18n name="itcast">
    	<s:text name="welcome">
    		<s:param>yr</s:param>
    		<s:param>study</s:param>
    	</s:text>
    </s:i18n>
    <s:i18n name="cn/itcast/action/package">
    	<s:text name="welcome">
    		<s:param>wxy</s:param>
    		<s:param>find boy friend</s:param>
    	</s:text>
    </s:i18n>
  </body>
```   


动作类中如何读取国际化的消息  

```
public class I18nAction extends ActionSupport implements Serializable {
	public String execute(){
		//取出资源文件中的welcome的值
//		String value = getText("welcome");
		String value = getText("welcome", new String[]{"朱巧玲","学习"});
		//封装到请求范围中
		ActionContext.getContext().put("message", value);
		return SUCCESS;
	}
}
```

### 完成