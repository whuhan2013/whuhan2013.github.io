---
layout: post
title: SSH框架整合
date: 2016-5-08
categories: blog
tags: [java]
description: SSH框架整合
---   

### ssh框架整合步骤如下  

提示：myeclipse环境、工程环境、tomcat环境的jdk保持一致       
1、新建一个工程，把工程的编码为utf-8              
2、把jsp的编码形式改成utf-8             
3、把jar包放入到lib下                   
4、建立三个src folder            
     src      存放源代码                
     config   存放配置文件                
        hibernate  存放hibernate的配置文件              
        spring     存放spring的配置文件        
        struts     存放struts的配置文件           
        struts.xml       
     test     存放单元测试         
5、在src下建立包          
       cn.itcast.s2sh.domain           
             持久化类和映射文件           
6、编写dao层和service层          
7、写spring的配置文件           
     1、写sessionFactory             
     2、测试           
     3、写dao和service           
     4、测试          
8、写action         
9、写spring的配置文件              
     把action注入到spring容器中        

```     
      <bean id="personAction"              class="cn.itcast.s2sh.struts2.action.sh.PersonAction" scope="prototype">
```   
   scope为"prototype"保证了action的多实例         
10、在web.xml          
      加入spring的监听器           
      加入struts2的过滤器          
11、请求          


### 详细代码 

### 持久化类与映射文件  

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
  <!-- 
    用来描述一个持久化类
    name  类的全名
    table 可以不写  默认值和类名一样 
    catalog  数据库的名称  一般不写
   -->
  <class name="cn.itcast.s2sh.domain.sh.Person">
    <!-- 
      标示属性  和数据库中的主键对应
      name  属性的名称
      column 列的名称
     -->
    <id name="pid" column="pid" length="200" type="java.lang.Long">
      <!-- 
        主键的产生器
          就该告诉hibernate容器用什么样的方式产生主键
       -->
      <generator class="increment"></generator>
    </id>
    <!-- 
      描述一般属性
     -->
    <property name="pname" column="pname" length="20" type="string">
    </property>
    
    <property name="psex" column="psex" length="10" type="java.lang.String"></property>
  </class>
</hibernate-mapping>
```


### dao  

```
package cn.itcast.s2sh.sh.dao.impl;

import java.io.Serializable;

import org.springframework.orm.hibernate3.support.HibernateDaoSupport;

import cn.itcast.s2sh.domain.sh.Person;
import cn.itcast.s2sh.sh.dao.PersonDao;

public class PersonDaoImpl extends HibernateDaoSupport implements PersonDao{

  @Override
  public void savePerson(Person person) {
    // TODO Auto-generated method stub
    this.getHibernateTemplate().save(person);
  }

  @Override
  public Person getPesonById(Serializable id) {
    // TODO Auto-generated method stub
    return (Person) this.getHibernateTemplate().load(Person.class, id);
  }
  
}
```
### service  

```
package cn.itcast.s2sh.sh.service.impl;

import java.io.Serializable;

import cn.itcast.s2sh.domain.sh.Person;
import cn.itcast.s2sh.sh.dao.PersonDao;
import cn.itcast.s2sh.sh.service.PersonService;

public class PersonServiceImpl implements PersonService{
  private PersonDao personDao;

  public PersonDao getPersonDao() {
    return personDao;
  }

  public void setPersonDao(PersonDao personDao) {
    this.personDao = personDao;
  }

  @Override
  public void savePerson(Person person) {
    // TODO Auto-generated method stub
    this.personDao.savePerson(person);
  }

  @Override
  public Person getPersonByID(Serializable id) {
    // TODO Auto-generated method stub
    return this.personDao.getPesonById(id);
  }
}
```


### spring配置文件  

### applicationcontext.xml  

```
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
  <import resource="applicationContext-db.xml"/>
  <import resource="applicationContext-person.xml"/>
</beans>
```

### applicationContext-db.xml  

```
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
  <!--  
  <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="configLocation">
      <value>classpath:hibernate/hibernate.cfg.xml</value>
    </property>
  </bean>
  -->
  <bean
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
      <value>classpath:jdbc.properties</value>
    </property>
  </bean>


  <bean id="dataSource" destroy-method="close"
    class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
  </bean>
  
  <bean id="sessionFactory"
    class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="dataSource">
      <ref bean="dataSource" />
    </property>
    <property name="mappingResources">
    <!-- list all the annotated PO classes -->
      <list>
        <value>
          cn/itcast/s2sh/domain/sh/Person.hbm.xml
        </value>
      </list>
    </property>
    <property name="hibernateProperties">
      <props>
        <prop key="hibernate.dialect">
          org.hibernate.dialect.MySQLDialect
        </prop>
        <prop key="hibernate.hbm2ddl.auto">
          update
        </prop>
      <prop key="hibernate.show_sql">
          true
        </prop>
        <prop key="hibernate.format_sql">
          true
        </prop>
      </props>
    </property>
  </bean>
  
  <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory">
      <ref bean="sessionFactory"/>
    </property>
  </bean>
  
  <tx:advice id="tx" transaction-manager="transactionManager">
    <tx:attributes>
      <tx:method name="save*" read-only="false"/>
      <tx:method name="update*" read-only="false"/>
      <tx:method name="delete*" read-only="false"/>
      <!-- 
        * 代表了除了上述的三种情况的以外的情况
       -->
      <tx:method name="*" read-only="true"/>
    </tx:attributes>
  </tx:advice>
    
  <aop:config>
    <aop:pointcut expression="execution(* cn.itcast.s2sh.sh.service.impl.*.*(..))" id="perform"/>
    <aop:advisor advice-ref="tx" pointcut-ref="perform"/>
  </aop:config>
</beans>
```


### applicationContext-person.xml 

```
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
  <bean id="personDao" class="cn.itcast.s2sh.sh.dao.impl.PersonDaoImpl">
    <property name="sessionFactory">
      <ref bean="sessionFactory"/>
    </property>
  </bean>
  
  <bean id="personService" class="cn.itcast.s2sh.sh.service.impl.PersonServiceImpl">
    <property name="personDao">
      <ref bean="personDao"/>
    </property>
  </bean>
  
  <bean id="personAction" class="cn.itcast.s2sh.struts2.action.sh.PersonAction" scope="prototype">
    <property name="personService">
      <ref bean="personService"/>
    </property>
  </bean>
</beans>
```

### struts.xml  

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
  "-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN"
  "http://struts.apache.org/dtds/struts-2.1.7.dtd">
<struts>
    <constant name="struts.devMode" value="true"/>
  <include file="struts2/struts-person.xml"></include>
  <!--  <constant name="struts.objectFactory" value="spring" />-->
</struts> 
```

### struts-person.xml  

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
  "-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN"
  "http://struts.apache.org/dtds/struts-2.1.7.dtd">
<struts>
   <package name="person" namespace="/" extends="struts-default">
      <action name="personAction_*" method="{1}" class="personAction">
        <result name="index">index.jsp</result>
      </action>
   </package>
</struts> 
```

### web.xml文件的编写  

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" 
  xmlns="http://java.sun.com/xml/ns/javaee" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
  <!-- 整合Spring -->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring/applicationContext.xml</param-value>
  </context-param>
  
   <filter>
    <filter-name>OpenSessionInViewFilter</filter-name>
    <filter-class>org.springframework.orm.hibernate3.support.OpenSessionInViewFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>OpenSessionInViewFilter</filter-name>
    <url-pattern>*.action</url-pattern>
  </filter-mapping>

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

### 三大框架整合原理

  1、三大框架的作用      

 - struts2是一个mvc框架           
 -  spring容器
         1、利用ioc和di做到了完全的面向接口编程           
         2、由于spring的声明式事务处理，使程序员不再关注事务            
         3、dao层和service层的类是单例的，但是action层是多例        
 -  hibernate      
         就是一个数据库的ormapping的框架            
         
  2、整合原理          
      1、当tomcat启动时，做的事情           
         1、因为在web.xml中， 

```
                 <listener>
                    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
                 </listener>
                 <context-param>
                      <param-name>contextConfigLocation</param-name>
                      <param-value>classpath:spring/applicationContext.xml</param-value>
                 </context-param>
                 <filter>
                      <filter-name>struts2</filter-name>
                      <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
                 </filter>
                 <filter-mapping>
                       <filter-name>struts2</filter-name>
                       <url-pattern>/*</url-pattern>
                 </filter-mapping>
```

 所以在启动的时候，执行的是

```

         ContextLoaderListener
         contextInitialized
      this.contextLoader = createContextLoader();
         加载spring的配置文件
    这里有一个固定的参数con的textConfigLocation
      可以指定classpath路径下的spring的配置文
       也可以任意位置指定配置文件  spring*.xml    WEB-INF/任意多个任意文件夹/spring-*.xml
  如果没有指定固定参数，则查找默认的加载路径：WEB-INF/applicationContext.xml
                          this.contextLoader.initWebApplicationContext(event.getServletContext());
 启动spring容器
 总结：当tomcat启动的时候，spring容器就启动了，这个时候service层和dao层所有的单例类就创建对象了
  struts2容器：
  加载了default.properties,struts-default.xml,struts-plugin.xml,struts.xml
```


   2、请求一个url时，发生的事情：
        1、在引入jar包时，导入了struts2-spring-plugin-2.1.8.1.jar包，该jar中有一个文件struts-plugin.xml

```
                <bean type="com.opensymphony.xwork2.ObjectFactory" name="spring" 
                      class="org.apache.struts2.spring.StrutsSpringObjectFactory" />
                <constant name="struts.objectFactory" value="spring" />
```

 2、由于上面的配置改变了action的生成方式，action由StrutsSpringObjectFactory生成，经过查找是由SpringObjectFactory中的buidBean方法
            生成的

```
               try {
         o = appContext.getBean(beanName);
          } catch (NoSuchBeanDefinitionException e) {
              lass beanClazz = getClassInstance(beanName);
        o = buildBean(beanClazz, extraContext);
        }
```
  
3、由上面的代码可以看出，先从spring容器中查找相应的action,如果没有找到，再根据反射机制创建action,
             beanName就是struts配置文件class属性的值，所以class属性的值和spring中ID的值保持一致




###  openSessionView模式  


```
在web.xml中

      <filter>
  <filter-name>OpenSessionInViewFilter</filter-name>
  <filter-class>org.springframework.orm.hibernate3.support.OpenSessionInViewFilter</filter-class>
 </filter>
 <filter-mapping>
  <filter-name>OpenSessionInViewFilter</filter-name>
  <url-pattern>*.action</url-pattern>
 </filter-mapping>

<filter>
  <filter-name>struts2</filter-name>
  <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
 </filter>
 <filter-mapping>
  <filter-name>struts2</filter-name>
  <url-pattern>/*</url-pattern>
 </filter-mapping>
```

  OpenSessionInView在第一个位置，struts2的过滤器在第二个位置

1、加入了OpenSessionInView模式解决了懒加载的问题
2、因为延迟了session的关闭时间，所以在session一级缓存中的数据会长时间停留在内存中，
    增加了内存的开销


### 错误

在整合SSH框架的时候出现了很多错误 

### java.lang.OutOfMemoryError: PermGen space

内存不够，可能是tomcat,或者是myeclipse内存不够的原因  

### 参考链接  

[Tomcat中JVM内存溢出及合理配置 - ye1992的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/ye1992/article/details/9344807)

[[转]MyEclipse内存不足问题 - - ITeye技术网站](http://lhp--2006.iteye.com/blog/1139064)

[myeclipse修改内存大小不足_百度经验](http://jingyan.baidu.com/article/f7ff0bfc73f54f2e26bb138c.html?qq-pf-to=pcqq.group)

### 错误  

![这里写图片描述](http://img.blog.csdn.net/20160508132857650)

这个错误很奇怪，困扰很久，原来的那种写法不知道为什么就是不能成功，而且在tomcat8中刷新一下就可以了，但在tomcat7中不行，而且如果tomcat中有了一个新的写法的程序时，原来写法的程序也可以用了，tomcat程序之间可以互相影响吗？？？找到一个最接近的解释，不知道对不对。

### 参考链接

[上网站时输入密码后提示could not open Hibernate Session for transaction；nested exception is_百度知道](http://zhidao.baidu.com/link?url=McyjA7OoxeQsdl8eJC994YkI_eTmS4fy19LsP09z7_arzWSGckWfiM3FDoCpOSBaErzFXqaNpIMHy9JhV19xJ_)

### 注解方式实现整合SSH框架  

### spring配置文件修改为  

```
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
    http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">
  <!--  
  <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="configLocation">
      <value>classpath:hibernate/hibernate.cfg.xml</value>
    </property>
  </bean>
  -->
  <bean
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
      <value>classpath:jdbc.properties</value>
    </property>
  </bean>


  <bean id="dataSource" destroy-method="close"
    class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
  </bean>
  
  <bean id="sessionFactory"
    class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="dataSource">
      <ref bean="dataSource" />
    </property>
    <property name="mappingResources">
    <!-- list all the annotated PO classes -->
      <list>
        <value>
          cn/itcast/s2sh/domain/sh/Person.hbm.xml
        </value>
      </list>
    </property>
    <property name="hibernateProperties">
      <props>
        <prop key="hibernate.dialect">
          org.hibernate.dialect.MySQLDialect
        </prop>
        <prop key="hibernate.hbm2ddl.auto">
          update
        </prop>
      <prop key="hibernate.show_sql">
          true
        </prop>
        <prop key="hibernate.format_sql">
          true
        </prop>
      </props>
    </property>
  </bean>
  
  <bean id="hibernateTemplate" class="org.springframework.orm.hibernate3.HibernateTemplate">
    <property name="sessionFactory">
      <ref bean="sessionFactory"/>
    </property>
  </bean>
  
  <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
    <property name="sessionFactory">
      <ref bean="sessionFactory"/>
    </property>
  </bean>
  
  <context:component-scan base-package="cn.itcast.s2sh"></context:component-scan>
  
  <tx:annotation-driven transaction-manager="transactionManager"/>
  
  
</beans>
```

### dao  

```
package cn.itcast.s2sh.sh.dao.impl;

import java.io.Serializable;

import javax.annotation.Resource;

import org.springframework.orm.hibernate3.HibernateTemplate;
import org.springframework.orm.hibernate3.support.HibernateDaoSupport;
import org.springframework.stereotype.Repository;

import cn.itcast.s2sh.domain.sh.Person;
import cn.itcast.s2sh.sh.dao.PersonDao;

@Repository("personDao")
public class PersonDaoImpl implements PersonDao{
  @Resource(name="hibernateTemplate")
  private HibernateTemplate hibernateTemplate;
  @Override
  public void savePerson(Person person) {
    // TODO Auto-generated method stub
    this.hibernateTemplate.save(person);
  }

  @Override
  public Person getPesonById(Serializable id) {
    // TODO Auto-generated method stub
    return  (Person) this.hibernateTemplate.load(Person.class, id);
  }
  
}
```

### service 

```
package cn.itcast.s2sh.sh.service.impl;

import java.io.Serializable;

import javax.annotation.Resource;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import cn.itcast.s2sh.domain.sh.Person;
import cn.itcast.s2sh.sh.dao.PersonDao;
import cn.itcast.s2sh.sh.service.PersonService;

@Service("personService")
public class PersonServiceImpl implements PersonService{
  @Resource(name="personDao")
  private PersonDao personDao;

  @Override
  @Transactional(readOnly=false)
  public void savePerson(Person person) {
    // TODO Auto-generated method stub
    this.personDao.savePerson(person);
  }

  @Override
  public Person getPersonByID(Serializable id) {
    // TODO Auto-generated method stub
    Person person = this.personDao.getPesonById(id);
    return person;
  }
}
```

### action 

```
package cn.itcast.s2sh.struts2.action.sh;

import javax.annotation.Resource;

import org.apache.struts2.ServletActionContext;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Controller;

import cn.itcast.s2sh.domain.sh.Person;
import cn.itcast.s2sh.sh.service.PersonService;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;
import com.opensymphony.xwork2.config.entities.ActionConfig;
import com.opensymphony.xwork2.config.impl.ActionConfigMatcher;

@Controller("personAction")
@Scope("prototype")
public class PersonAction extends ActionSupport{
  @Resource(name="personService")
  private PersonService personService;
  
  public String savePerson(){
    Person person = new Person();
    person.setPname("afds");
    this.personService.savePerson(person);
    return null;
  }
  
  public String showPerson(){
    System.out.println("annoation aaaaaaaaaaaaaaaaa");
    Person person = this.personService.getPersonByID(2L);
    ServletActionContext.getRequest().setAttribute("person", person);
    return "index";
  }
}
```

[错误配置的代码](https://github.com/whuhan2013/ssh/tree/master/s2sh_sh)             
[正确配置的代码](https://github.com/whuhan2013/ssh/tree/master/s2sh)     
[注解配置的代码](https://github.com/whuhan2013/ssh/tree/master/s2sh_annoation)        

### 完成