---
layout: post
title: java之代理设计模式
date: 2016-5-04
categories: blog
tags: [java]
description: java之代理设计模式
---   

   
代理模式是常用的java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。       
按照代理的创建时期，代理类可以分为两种。          
静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。            
动态代理：在程序运行时，运用反射机制动态创建而成。         
 
![这里写图片描述](http://img.blog.csdn.net/20160504190821103)    


### 静态代理 

1. 定义一个接口  

```
package cn.itcast.proxy.sh;

public interface PersonDao {
  public void savePerson();
  public void updatePerson();
  public void deletePerson();
}
```

2、 目标类   

```
package cn.itcast.proxy.sh;

public class PersonDaoImpl implements PersonDao{
  public void deletePerson() {
    System.out.println("delete person");
  }
  public void savePerson() {
    System.out.println("save person");
  }
  public void updatePerson() {
    System.out.println("update person");
  }
}
``` 

3、代理类,包含了目标类和一些额外的操作

```
package cn.itcast.proxy.sh;

public class PersonDaoProxy implements PersonDao{
  private PersonDao personDao;
  private Transaction transaction;
  public PersonDaoProxy(PersonDao personDao,Transaction transactions){
    this.personDao = personDao;
    this.transaction = transactions;
  }
  public void deletePerson() {
    this.transaction.beginTransaction();
    this.personDao.deletePerson();
    this.transaction.commit();
  }
  public void savePerson() {
    this.transaction.beginTransaction();
    this.personDao.savePerson();
    this.transaction.commit();
  }
  public void updatePerson() {
    this.transaction.beginTransaction();
    this.personDao.updatePerson();
    this.transaction.commit();
  }
}
```

4、使用  

```
package cn.itcast.proxy.sh;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ProxyTest {
  @Test
  public void test(){
    ApplicationContext context = new ClassPathXmlApplicationContext("cn/itcast/proxy/sh/applicationContext-proxy.xml");
    PersonDao personDao = (PersonDao)context.getBean("personDao2");
    personDao.deletePerson();
  }
}
```

### 静态代理的缺点  

静态代理模式的缺点：
      1、如果一个系统中有100Dao，则创建100个代理对象        
      2、如果一个dao中有很多方法需要事务，则代理对象的方法中重复代码还是很多                        
      3、由第一点和第二点可以得出：proxy的重用性不强          

### 动态代理  

 1、产生的代理对象和目标对象实现了共同的接口             
       jdk动态代理               
   2、代理对象是目标对象的子类                      
       hibernate: Person person = session.load(Person.class,1L);      javassisit                     
       spring:cglib动态代理                      

jdk的动态代理：                  
   1、因为是用jdk的API做到的                             
   2、代理对象是动态产生的                
cglib产生的代理类是目标类的子类                           

注意事项：                       
    1、拦截器中invoke方法体的内容就是代理对象方法体的内容             
    2、当客户端执行代理对象.方法时，进入到了拦截器的invoke方法体       
    3、拦截器中invoke方法的method参数是在调用的时候赋值的             


### jdk动态代理  

### 定义一个接口   

```
package cn.itcast.jdkproxy.sh;

public interface PersonDao {
  public void savePerson();
  public void updatePerson();
  public void deletePerson();
}
```  

### 目标类  

```
package cn.itcast.jdkproxy.sh;

public class PersonDaoImpl implements PersonDao{
  public void deletePerson() {
    System.out.println("delete person");
  }
  public void savePerson() {
    System.out.println("save person");
  }
  public void updatePerson() {
    System.out.println("update person");
  }
}
```

### 拦截器  

```
package cn.itcast.jdkproxy.sh;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * 拦截器
 * @author Think
 *   1、引入目标类
 *   2、引入事务
 *   3、通过构造函数给目标类和事务赋值
 *   4、填充invoke方法
 *
 */
public class PersonInterceptor implements InvocationHandler{
  
  private Object target;//目标类
  private Transaction transaction;//引入事务
  
  public PersonInterceptor(Object target,Transaction transaction){
    this.target = target;
    this.transaction = transaction;
  }
  

  @Override
  public Object invoke(Object proxy, Method method, Object[] args)
      throws Throwable {
    // TODO Auto-generated method stub
    this.transaction.beginTransaction();
    method.invoke(this.target, args);//调用目标类的方法
    this.transaction.commit();
    return null;
  }

}
``` 

### 使用  

```
package cn.itcast.jdkproxy.sh;

import java.lang.reflect.Proxy;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ProxyTest {
  @Test
  public void test(){
    Object target = new PersonDaoImpl();
    Transaction transaction = new Transaction();
    PersonInterceptor interceptor = new PersonInterceptor(target, transaction);
    /**
     * 1、目标类的类加载器
     * 2、目标类实现的所有的接口
     * 3、拦截器
     */
    PersonDao personDao = (PersonDao)Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), interceptor);
    personDao.deletePerson();
  }
}
``` 


### cglib动态代理  

JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。 

### 目标类  

```
package net.battier.dao.impl;  
   
 /** 
  * 这个是没有实现接口的实现类 
  *  
  * @author student 
  *  
  */  
 public class BookFacadeImpl1 {  
     public void addBook() {  
         System.out.println("增加图书的普通方法...");  
     }  
 }  
```

### 拦截类  

```
package net.battier.proxy;  
   
import java.lang.reflect.Method;  
  
import net.sf.cglib.proxy.Enhancer;  
import net.sf.cglib.proxy.MethodInterceptor;  
import net.sf.cglib.proxy.MethodProxy;  
  
/** 
 * 使用cglib动态代理 
 *  
 * @author student 
 *  
  */  
 public class BookFacadeCglib implements MethodInterceptor {  
     private Object target;  
   
    /** 
     * 创建代理对象 
      *  
      * @param target 
      * @return 
      */  
     public Object getInstance(Object target) {  
        this.target = target;  
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(this.target.getClass());  
        // 回调方法  
        enhancer.setCallback(this);  
        // 创建代理对象  
        return enhancer.create();  
    }  
  
    @Override  
    // 回调方法  
    public Object intercept(Object obj, Method method, Object[] args,  
            MethodProxy proxy) throws Throwable {  
        System.out.println("事物开始");  
        proxy.invokeSuper(obj, args);  
        System.out.println("事物结束");  
        return null;  
  
  
    }  
  
}  
```

### 使用  

```
package net.battier.test;  
  
import net.battier.dao.impl.BookFacadeImpl1;  
import net.battier.proxy.BookFacadeCglib;  
  
public class TestCglib {  
      
    public static void main(String[] args) {  
        BookFacadeCglib cglib=new BookFacadeCglib();  
        BookFacadeImpl1 bookCglib=(BookFacadeImpl1)cglib.getInstance(new BookFacadeImpl1());  
        bookCglib.addBook();  
    }  
}  
```

### 参考链接
[java动态代理（JDK和cglib） - C'est la vie - 博客园](http://www.cnblogs.com/jqyp/archive/2010/08/20/1805041.html)

