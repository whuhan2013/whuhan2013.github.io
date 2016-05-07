---
layout: post
title: Spring之数据库操作
date: 2016-5-07
categories: blog
tags: [java]
description: Spring之数据库操作
---   

### 本文主要包括以下内容  

1. spring+jdbc数据库操作   
2. spring+jdbc声明事务处理   
3. spring+hibernate声明事务处理   

### spring+jdbc数据库操作  

### 方法   

1、让自己写的一个dao类继承JdbcDaoSupport             
2、让自己写的一个dao类继承JdbcTemplate          
3、让自己写的一个dao类里有一个属性为JdbcTemplate        

### 总结   

   1、引入dataSource的方式：               
        1、在dataSource的设置中直接写值              
        2、引入properties文件                
   2、在dao的写法中有很多种，最终只需要把dataSource注入jdbcTemplate中


### 继承JdbcDaoSupport的方法  

```
package cn.itcast.spring.jdbc;

import java.util.List;

import org.springframework.jdbc.core.support.JdbcDaoSupport;

public class PersonDao extends JdbcDaoSupport{
  public void update(){
    this.getJdbcTemplate().execute("update person set pname='a' where pid=3");
  }
  
  public void query(){
    List<Person> persons = this.getJdbcTemplate().query("select * from person", new PersonRowMapper());
    for(Person person:persons){
      System.out.println(person.getPname());
    }
  }
}

```

### 继承JdbcTemplate的方法  

```
package cn.itcast.spring.jdbc;

import javax.sql.DataSource;

import org.springframework.jdbc.core.JdbcTemplate;

public class PersonDao2 extends JdbcTemplate{
  public PersonDao2(DataSource dataSource){
    super(dataSource);
  }
  public void update(){
    this.execute("update person set pname='aa' where pid=2");
  }
}
```

### 属性为JdbcTemplate的方法  

```
package cn.itcast.spring.jdbc;

import javax.sql.DataSource;

import org.springframework.jdbc.core.JdbcTemplate;

public class PersonDao3{
  private JdbcTemplate jdbcTemplate;
  public JdbcTemplate getJdbcTemplate() {
    return jdbcTemplate;
  }
  public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
    this.jdbcTemplate = jdbcTemplate;
  }
  public void update(){
    this.jdbcTemplate.execute("update person set pname='aaa' where pid=2");
  }
}
```

### 其中查询时的PersonRowMapper为  

```
package cn.itcast.spring.jdbc;

import java.sql.ResultSet;
import java.sql.SQLException;

import org.springframework.jdbc.core.RowMapper;

public class PersonRowMapper implements RowMapper{

  @Override
  public Object mapRow(ResultSet rs, int rowNum) throws SQLException {
    // TODO Auto-generated method stub
    Person person = new Person();
    person.setPid(rs.getLong("pid"));
    person.setPname(rs.getString("pname"));
    person.setPsex(rs.getString("psex"));
    return person;
  }
  /**
   * crud做一下
   */
  
  

}
```

### 注入dataSource的方法，在配置文件中配置   

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
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
  
  <bean id="personDao" class="cn.itcast.spring.jdbc.PersonDao">
    <property name="dataSource">
      <ref bean="dataSource"/>
    </property>
  </bean>
  
  <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource">
      <ref bean="dataSource"/>
    </property>
  </bean>
  
  <bean id="personDao2" class="cn.itcast.spring.jdbc.PersonDao2">
    <constructor-arg index="0" ref="dataSource"></constructor-arg>
  </bean>
  
  <bean id="personDao3" class="cn.itcast.spring.jdbc.PersonDao3">
    <property name="jdbcTemplate">
      <ref bean="jdbcTemplate"/>
    </property>
  </bean>
</beans>
```  

### jdbc.properties如下，路径在src文件夹下

```
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc\:mysql\://localhost\:3306/hibernate
jdbc.username=root
jdbc.password=root
```


### 测试  

```
package cn.itcast.spring.jdbc;

import javax.sql.DataSource;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class PersonTest {
  @Test
  public void test(){
    ApplicationContext context = new ClassPathXmlApplicationContext("cn/itcast/spring/jdbc/applicationContext.xml");
    PersonDao personDao = (PersonDao)context.getBean("personDao");
    personDao.update();
  }
  
  @Test
  public void testDataSource()
  {
    ApplicationContext context=new ClassPathXmlApplicationContext("cn/itcast/spring/jdbc/applicationContext.xml");
    DataSource datasource=(DataSource) context.getBean("dataSource");
    System.out.println(datasource);
  }
}
``` 


### spring+jdbc声明事务处理  


spring声明式事务处理的步骤：     
   1、搭建环境              
   2、把dao层和service层的接口和类写完               
   3、在spring的配置文件中，先导入dataSource               
   4、测试                   
   5、导入dao和service层的bean             
   6、测试                    
   7、进行AOP的配置                 
       1、引入事务管理器                  
       2、进行aop的配置                    
   8、测试service层的类看是否是代理对象            

### dao

```
package cn.itcast.spring.jdbc.transaction.sh.dao;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.support.JdbcDaoSupport;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.support.AbstractPlatformTransactionManager;

import cn.itcast.spring.jdbc.transaction.bean.Person;

public class PersonDaoImpl extends JdbcDaoSupport implements PersonDao{

  @Override
  public List<Person> getPerson() {
    // TODO Auto-generated method stub
    return this.getJdbcTemplate().query("select * from person", new RowMapper() {
      
      @Override
      public Object mapRow(ResultSet rs, int rowNum) throws SQLException {
        // TODO Auto-generated method stub
        Person person = new Person();
        person.setPid(rs.getLong("pid"));
        person.setPname(rs.getString("pname"));
        person.setPsex(rs.getString("psex"));
        return person;
      }
    });
  }

  @Override
  public void savePerson() {
    // TODO Auto-generated method stub
    this.getJdbcTemplate().update("insert into person values(11,'a','a')");
  }
  
}

```

### Service

```
package cn.itcast.spring.jdbc.transaction.sh.service;

import java.util.List;

import org.springframework.transaction.PlatformTransactionManager;

import cn.itcast.spring.jdbc.transaction.bean.Person;
import cn.itcast.spring.jdbc.transaction.sh.dao.PersonDao;

public class PersonServiceImpl implements PersonService{
  private PersonDao personDao;

  public PersonDao getPersonDao() {
    return personDao;
  }

  public void setPersonDao(PersonDao personDao) {
    this.personDao = personDao;
  }

  @Override
  public List<Person> getPerson() {
    // TODO Auto-generated method stub
    return this.personDao.getPerson();
  }

  @Override
  public void savePerson() {
    // TODO Auto-generated method stub
    this.personDao.savePerson();
    
  }
  
}
```

### spring配置文件

完成了以下功能      
1. 导入dataSource
2. 导入dao和service层的bean  
3. 引入事务管理器    
4. 进行aop的配置 

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
  
  <bean id="personDao" class="cn.itcast.spring.jdbc.transaction.sh.dao.PersonDaoImpl">
    <property name="dataSource">
      <ref bean="dataSource" />
    </property>
  </bean>
  
  
  <bean id="personService"
    class="cn.itcast.spring.jdbc.transaction.sh.service.PersonServiceImpl">
    <property name="personDao">
      <ref bean="personDao" />
    </property>
  </bean>
  
  <!-- 异常处理 -->
  <bean id="myException" class="cn.itcast.spring.jdbc.transaction.exception.MyException"></bean>
  
  <!-- 确定transactionManager的种类 -->
  <bean id="transactionManager"
    class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource">
      <ref bean="dataSource" />
    </property>
  </bean>
  
  <!--
    通知 1、告诉spring容器，采用什么样的方法处理事务 2、告诉spring容器，目标方法应该采用什么样的事务处理策略
  -->
  <tx:advice id="tx" transaction-manager="transactionManager">
    <tx:attributes>
      <!--
        name规定方法 isolation(隔离机制) 默认值为DEFAULT propagation(传播机制) REQUIRED
      -->
      <tx:method name="save*" read-only="false" />
    </tx:attributes>
  </tx:advice>
  
  <aop:config>
    <aop:pointcut
      expression="execution(* cn.itcast.spring.jdbc.transaction.sh.service.*.*(..))"
      id="perform" />
    <aop:advisor advice-ref="tx" pointcut-ref="perform" />
    <aop:aspect ref="myException">
      <aop:after-throwing method="myException"
        pointcut-ref="perform" throwing="ex" />
    </aop:aspect>
  </aop:config>

  
</beans>
```

### 测试  

```
package cn.itcast.spring.jdbc.transaction.test;

import javax.sql.DataSource;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import cn.itcast.spring.jdbc.transaction.sh.service.PersonService;

public class TransactionTest {
  public static ApplicationContext context;
  static{
    context = new ClassPathXmlApplicationContext("cn/itcast/spring/jdbc/transaction/config/applicationContext.xml");
  }
  @Test
  public void testDataSource(){
    DataSource dataSource = (DataSource)context.getBean("dataSource");
    System.out.println(dataSource);
  }
  
  @Test
  public void testPesonDao(){
    context.getBean("personDao");
  }
  
  @Test
  public void testPersonService(){
    PersonService personService = (PersonService)context.getBean("personService");
    personService.savePerson();
  }
}
``` 

### spring+hibernate声明事务处理  


### 主要注意sessionFactory在spring配置文件中的配置  

### 方法一  

```
配置sessionFactory第一种方式
  <bean id="sessionFactory"
    class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="mappingResources">
      <list>
        <value>cn/itcast/spring/hiberante/transaction/domain/Person.hbm.xml</value>
      </list>
    </property>
    <property name="hibernateProperties">
      <value>
        hibernate.dialect=org.hibernate.dialect.MySQLDialect
          </value>
    </property>
  </bean>
```

### 方法二  

```
 <!-- 第二种方式，导入hibernate配置文件 -->
  <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="configLocation">
      <value>classpath:cn/itcast/spring/hibernate/transaction/config/hibernate.cfg.xml</value>
    </property>
  </bean>
```

### 详细代码如下  

### dao 

```
package cn.itcast.spring.hibernate.transaction.dao.impl;

import org.springframework.orm.hibernate3.support.HibernateDaoSupport;

import cn.itcast.spring.hiberante.transaction.domain.Person;
import cn.itcast.spring.hibernate.transaction.dao.PersonDao;

public class PersonDaoImpl extends HibernateDaoSupport implements PersonDao{

  @Override
  public void savePerson(Person person) {
    // TODO Auto-generated method stub
    this.getHibernateTemplate().save(person);
  }
  
}
```

### service 

```
package cn.itcast.spring.hibernate.transaction.service.impl;

import org.hibernate.impl.SessionFactoryImpl;
import org.springframework.orm.jdo.JdoTemplate;
import org.springframework.orm.toplink.TopLinkTemplate;

import cn.itcast.spring.hiberante.transaction.domain.Person;
import cn.itcast.spring.hibernate.transaction.dao.PersonDao;
import cn.itcast.spring.hibernate.transaction.service.PersonService;

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
}
```

### 配置文件  

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


  <!--
    sessionFactory 1、sessionFactoryImpl 2、利用spring的IOC和DI的特征
  -->
  <!-- 
  
  配置sessionFactory第一种方式
  <bean id="sessionFactory"
    class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
    <property name="mappingResources">
      <list>
        <value>cn/itcast/spring/hiberante/transaction/domain/Person.hbm.xml</value>
      </list>
    </property>
    <property name="hibernateProperties">
      <value>
        hibernate.dialect=org.hibernate.dialect.MySQLDialect
          </value>
    </property>
  </bean>
   -->
   <!-- 第二种方式，导入hibernate配置文件 -->
  <bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
    <property name="configLocation">
      <value>classpath:cn/itcast/spring/hibernate/transaction/config/hibernate.cfg.xml</value>
    </property>
  </bean>
  <bean id="personDao"
    class="cn.itcast.spring.hibernate.transaction.dao.impl.PersonDaoImpl">
    <property name="sessionFactory">
      <ref bean="sessionFactory"/>
    </property>
  </bean>
  
  <bean id="personService" class="cn.itcast.spring.hibernate.transaction.service.impl.PersonServiceImpl">
    <property name="personDao">
      <ref bean="personDao"/>
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
    </tx:attributes>
  </tx:advice>
  
  <aop:config>
    <aop:pointcut expression="execution(* cn.itcast.spring.hibernate.transaction.service.impl.PersonServiceImpl.*(..))" id="perform"/>
    <aop:advisor advice-ref="tx" pointcut-ref="perform"/>
  </aop:config>
</beans>
```

### 测试  

```
package cn.itcast.spring.hibernate.transaction.test;

import javax.sql.DataSource;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import cn.itcast.spring.hiberante.transaction.domain.Person;
import cn.itcast.spring.hibernate.transaction.service.PersonService;

public class PersonTest {
  
  @Test
  public void testDataSource()
  {
    ApplicationContext context =  new ClassPathXmlApplicationContext("cn/itcast/spring/hibernate/transaction/config/applicationContext.xml");
    DataSource dataSource=(DataSource) context.getBean("dataSource");
    
  }
  @Test
  public void test(){
    ApplicationContext context =  new ClassPathXmlApplicationContext("cn/itcast/spring/hibernate/transaction/config/applicationContext.xml");
    PersonService personService = (PersonService)context.getBean("personService");
    Person person = new Person();
    person.setPname("a");
    personService.savePerson(person);
  }
}
```

### 完成   