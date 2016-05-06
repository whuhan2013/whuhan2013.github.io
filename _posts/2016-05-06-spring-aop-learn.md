---
layout: post
title: Spring之AOP
date: 2016-5-06
categories: blog
tags: [java]
description: Spring之AOP
---   

在软件业，AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。


### aop中的基本概念  

   1、切面          
        事务、日志、安全性框架、权限等都是切面       
   2、通知            
      切面中的方法就是通知            
   3、目标类               
   4、切入点            
         只有符合切入点，才能让通知和目标方法结合在一起           
   5、织入：             
         形成代理对象的方法的过程             
          
  
好处：
   事务、日志、安全性框架、权限、目标方法之间完全是松耦合的

### 切入点表达式  

![这里写图片描述](http://img.blog.csdn.net/20160506084841445)

```
execution（public * *（..））  所有的公共方法

execution（* set*（..））  以set开头的任意方法

execution（* com.xyz.service.AccountService.*（..）） com.xyz.service.AccountService类中的所有的方法

execution（* com.xyz.service.*.*（..））  com.xyz.service包中的所有的类的所有的方法

execution（* com.xyz.service..*.*（..）） com.xyz.service包及子包中所有的类的所有的方法

execution(* cn.itcast.spring.sh..*.*(String,?,Integer))  cn.itcast.spring.sh包及子包中所有的类的有三个参数
                                                            第一个参数为String,第二个参数为任意类型，
                                                            第三个参数为Integer类型的方法
```

### springAOP的具体加载步骤：

   1、当spring容器启动的时候，加载了spring的配置文件    
   2、为配置文件中所有的bean创建对象          
   3、spring容器会解析aop:config的配置              
       1、解析切入点表达式，用切入点表达式和纳入spring容器中的bean做匹配          
            如果匹配成功，则会为该bean创建代理对象，代理对象的方法=目标方法+通知           
            如果匹配不成功，不会创建代理对象         
   4、在客户端利用context.getBean获取对象时，如果该对象有代理对象则返回代理对象，如果无代理对象，则返回目标对象           

说明：如果目标类没有实现接口，则spring容器会采用cglib的方式产生代理对象，如果实现了接口，会采用jdk的方式     


### 通知

- 前置通知         
      1、在目标方法执行之前执行          
      2、无论目标方法是否抛出异常，都执行，因为在执行前置通知的时候，目标方法还没有执行，还没有遇到异常          
      
- 后置通知
      1、在目标方法执行之后执行
      2、当目标方法遇到异常，后置通知将不再执行        
      3、后置通知可以接受目标方法的返回值，但是必须注意：      
               后置通知的参数的名称和配置文件中returning="var"的值是一致的         
                    
- 最终通知：             
      1、在目标方法执行之后执行     
      2、无论目标方法是否抛出异常，都执行，因为相当于finally       
      
- 异常通知         
      1、接受目标方法抛出的异常信息          
      2、步骤          
           在异常通知方法中有一个参数Throwable  ex          
           在配置文件中      
           
```
              <aop:after-throwing method="throwingMethod" pointcut-ref="perform" throwing="ex"/>
```     

 - 环绕通知      
       1、如果不在环绕通知中调用ProceedingJoinPoint的proceed，目标方法不会执行       
       2、环绕通知可以控制目标方法的执行         

### 实例   


### spring配置文件  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:aop="http://www.springframework.org/schema/aop" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
    <!-- 
      导入目标类，导入切面
     -->       
     <bean id="personDao" class="cn.itcast.spring.aop.sh.PersonDaoImpl"></bean>
     
     <bean id="myTransaction" class="cn.itcast.spring.aop.sh.MyTransaction"></bean>
     
     <!-- 
      aop的配置
      -->
      <aop:config>
        <!-- 
           切入点表达式
         -->
        <aop:pointcut expression="execution(* cn.itcast.spring.aop.sh.PersonDaoImpl.*(..))" id="perform"/>
        <aop:aspect ref="myTransaction">
      
          <aop:before method="beginTransaction" pointcut-ref="perform"/>
          <aop:after-returning method="commit" pointcut-ref="perform" returning="var"/>
      
           <aop:after method="finallyMethod" pointcut-ref="perform"/>
          <aop:after-throwing method="throwingMethod" pointcut-ref="perform" throwing="ex"/>
          <aop:around method="aroundMethod" pointcut-ref="perform"/>
          
        </aop:aspect>
      </aop:config>
</beans>

```   

### 目标类  

```
package cn.itcast.spring.aop.sh;



public interface PersonDao {
  public void savePerson(Person person);
}
```  

### 切面   

```
package cn.itcast.spring.aop.sh;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.hibernate.Transaction;



public class MyTransaction extends HibernateUtils{
  private Transaction transaction;
  /**
   * 通过该参数可以获取目标方法的一些信息
   * @param joinpoint
   */
  public void beginTransaction(JoinPoint joinpoint){
    System.out.println("bbb");
    this.transaction = sessionFactory.getCurrentSession().beginTransaction();
  }
  
  public void commit(Object var){
    System.out.println(var);
    System.out.println("abc");
    this.transaction.commit();
  }
  
  public void finallyMethod(){
    System.out.println("finally method");
  }
  
  /**
   * 异常通知
   * @param ex
   */
  public void throwingMethod(Throwable ex){
    System.out.println(ex.getMessage());
  }
  
  public void aroundMethod(ProceedingJoinPoint joinPoint) throws Throwable{
    System.out.println(joinPoint.getSignature().getName());
    joinPoint.proceed();//调用目标方法
    System.out.println("aaaasfdasfd");
  }
}
```  

### 使用  

```
package cn.itcast.spring.aop.sh;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;



public class PersonTest {
  @Test
  public void test(){
    ApplicationContext context = new ClassPathXmlApplicationContext("cn/itcast/spring/aop/sh/applicationContext.xml");
    PersonDaoImpl personDao = (PersonDaoImpl)context.getBean("personDao");
    Person person = new Person();
    person.setPname("aaas");
    personDao.savePerson(person);
  }
}
``` 

### 多个切面的spring配置文件  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:aop="http://www.springframework.org/schema/aop" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
   <bean id="salaryManager" class="cn.itcast.spring.aop.sh.aspects.SalaryManagerImpl"></bean>

  <bean id="logger" class="cn.itcast.spring.aop.sh.aspects.Logger"></bean>
  
  <bean id="security" class="cn.itcast.spring.aop.sh.aspects.Security"></bean>
  
  <bean id="privilege" class="cn.itcast.spring.aop.sh.aspects.Privilege">
    <property name="access" value="afds"></property>
  </bean>
  
  <aop:config>
    <aop:pointcut expression="execution(* cn.itcast.spring.aop.sh.aspects.SalaryManagerImpl.*(..))" id="perform"/>
    <aop:aspect ref="logger">
      <aop:before method="logging" pointcut-ref="perform"/>
    </aop:aspect>
    <aop:aspect ref="security">
      <aop:before method="security" pointcut-ref="perform"/>
    </aop:aspect>
    <aop:aspect ref="privilege">
      <aop:around method="isAccess" pointcut-ref="perform"/>
    </aop:aspect>
  </aop:config>
</beans>

```

### 使用注解方式实现AOP 

### spring配置文件  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd">
   <!-- 
      把目标类和切面纳入到spring容器中管理，启动类扫描的机制
    -->
    <context:component-scan base-package="cn.itcast.spring.aop.annotation.sh"></context:component-scan>
    <!-- 
      启动aop的注解解析器
     -->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

### 目标类 

```
package cn.itcast.spring.aop.annotation.sh;

import org.springframework.stereotype.Repository;


@Repository("personDao")
public class PersonDaoImpl extends HibernateUtils{
  public String savePerson(Person person) {
    // TODO Auto-generated method stub
    //int a = 1/0;
    sessionFactory.getCurrentSession().save(person);
    System.out.println("aaaa");
    return "aaaa";
  }
}
```

### 切面  

```
package cn.itcast.spring.aop.annotation.sh;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.hibernate.Transaction;
import org.springframework.stereotype.Component;


/**
 * @Aspect
 * ==
 * <aop:config> 
 *   <aop:pointcut
 *    expression=
 *     "execution(* cn.itcast.spring.aop.annotation.sh.PersonDaoImpl.*(..))"
 *     id="aa()"/>
 * </aop:config>
 * @author Think
 *
 */
@Component("myTransaction")
@Aspect
public class MyTransaction extends HibernateUtils{
  private Transaction transaction;
  
  @Pointcut("execution(* cn.itcast.spring.aop.annotation.sh.PersonDaoImpl.*(..))")
  private void aa(){}//方法签名  返回值必须是void  方法的修饰符最好是private
  
  @Before("aa()")
  public void beginTransaction(JoinPoint joinpoint){
    this.transaction = sessionFactory.getCurrentSession().beginTransaction();
  }
  
  @AfterReturning(value="aa()",returning="val")
  public void commit(Object val){
    this.transaction.commit();
  }
  
}
```

### 其中  

```
@Pointcut("execution(* cn.itcast.spring.aop.annotation.sh.PersonDaoImpl.*(..))")
  private void aa(){}//方法签名  返回值必须是void  方法的修饰符最好是private

相当于    


 <aop:config> 
     <aop:pointcut
      expression=
      "execution(* cn.itcast.spring.aop.annotation.sh.PersonDaoImpl.*(..))"
      id="aa()"/>
  </aop:config>
 
```

### 完成