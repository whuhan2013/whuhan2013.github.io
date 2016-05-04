---
layout: post
title: Spring入门
date: 2016-5-04
categories: blog
tags: [java]
description: Spring入门
---   

### 本文主要包括以下内容  

1. 控制反转（IOC)
2. springDI
3. springIOC与DI实现MVC实例

### 控制反转

即把对象的创建交给spring容器来做

 - spring容器创建对象的方式     
         1、默认是调用默认的构造函数    
         2、利用静态工厂方法创建         
             spring调用工厂方法产生对象，但是真正创建对象还是由程序员来完成的            
         3、实例工厂方法          
        说明：         
           spring配置文件中，只要是一个bean就会为该bean创建对象      
    
 - spring容器创建对象的时机       
        在单例的情况下         
         1、在默认的情况下，启动spring容器创建对象        
         2、在spring的配置文件bean中有一个属性lazy-init="default/true/false"         
               1、如果lazy-init为"default/false"在启动spring容器时创建对象          
               2、如果lazy-init为"true",在context.getBean时才要创建对象
           意义：              
                在第一种情况下可以在启动spring容器的时候，检查spring容器配置文件的正确性，如果再结合tomcat,              
                如果spring容器不能正常启动，整个tomcat就不能正常启动。但是这样的缺点是把一些bean过早的放在了          
                内存中，如果有数据，则对内存来是一个消耗           
                在第二种情况下，可以减少内存的消耗，但是不容易发现错误       
        在多例的情况下
            就是一种情况：在context.getBean时才创建对象         
            
 - spring的bean中的scope       
        1、由spring产生的bean默认是单例的           
        2、可以在spring的配置文件中，scope的值进行修改             ="singleton/prototype/request/session/global session"        
        3、如果spring的配置文件的scope为"prototype"，则在得到该bean时才创建对象            
        
 - spring容器对象的生命周期：       
        1、spring容器创建对象       
        2、执行init方法          
        3、调用自己的方法            
        4、当spring容器关闭的时候执行destroy方法        

### spring容器创建对象实例  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
    <bean id="helloWorld_C_M" class="cn.itcast.spring.sh.createobject.method.HelloWorld"></bean>
    <!-- 
      factory-method
        工厂方法
     -->
  <bean id="helloFactory" class="cn.itcast.spring.sh.createobject.method.HelloWorldFactory"></bean>

  <bean id="aa" factory-bean="helloFactory" factory-method="getIns"></bean>
</beans>

```

### springDI  

通过依赖注入给属性赋值

### 有两种方法  

### set方法赋值  

```
package cn.itcast.spring.sh.di.set;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;


public class Person {
  private Long pid;
  private String pname;
  private Student student;
  
  private Set sets;
  
  public Long getPid() {
    return pid;
  }

  public void setPid(Long pid) {
    this.pid = pid;
  }

  public String getPname() {
    return pname;
  }

  public void setPname(String pname) {
    this.pname = pname;
  }

  public Student getStudent() {
    return student;
  }

  public void setStudent(Student student) {
    this.student = student;
  }

  public Set getSets() {
    return sets;
  }

  public void setSets(Set sets) {
    this.sets = sets;
  }

  public List getLists() {
    return lists;
  }

  public void setLists(List lists) {
    this.lists = lists;
  }

  public Map getMap() {
    return map;
  }

  public void setMap(Map map) {
    this.map = map;
  }

  public Properties getProperties() {
    return properties;
  }

  public void setProperties(Properties properties) {
    this.properties = properties;
  }

  private List lists;
  
  private Map map;
  
  private Properties properties;
}

```  

### XML文件中的配置   

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
  <!-- 
    把person和student放入到spring容器中
   -->
   <!-- 
    property是用来描述一个类的属性
       基本类型的封装类、String等需要值的类型用value赋值
      引用类型用ref赋值
    -->
  <bean id="person" class="cn.itcast.spring.sh.di.set.Person">
    <property name="pid" value="1"></property>
    <property name="pname" value="aaa"></property>
    <property name="student">
      <ref bean="student"/>
    </property>
    <property name="lists">
      <list>
        <value>list1</value>
        <ref bean="student"/>
        <value>list2</value>
      </list>
    </property>
    <property name="sets">
      <set>
        <value>set1</value>
        <ref bean="student"/>
        <value>set2</value>
      </set>
    </property>
    <property name="map">
      <map>
        <entry key="m1">
          <value>map1</value>
        </entry>
        <entry key="m2">
          <ref bean="student"/>
        </entry>
      </map>
    </property>
    <property name="properties">
      <props>
        <prop key="prop1">
          prop1
        </prop>
        <prop key="prop2">
          prop2
        </prop>
      </props>
    </property>
  </bean>
  
  <bean id="student" class="cn.itcast.spring.sh.di.set.Student"></bean>
</beans>
```

### 构造函数方法赋值   

1、如果spring的配置文件中的bean中没有<constructor-arg>该元素，则调用默认的构造函数          
 2、如果spring的配置文件中的bean中有<constructor-arg>该元素，则该元素确定唯一的构造函数        
          index  代表参数的位置  从0开始计算         
          type   指的是参数的类型           
          value  给基本类型赋值               
          ref    给引用类型赋值              

```
package cn.itcast.spring.sh.di.constructor;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

public class Person {
  private Long pid;
  private String pname;
  private Student student;
  
  private Set sets;
  
  public Person(String pname,Student student){
    this.pname = pname;
    this.student = student;
  }
  
  public Person(Long pid,String pname,Student student){
    this.pid = pid;
    this.pname = pname;
    this.student = student;
  }
  
  public Person(){}
  
  public Long getPid() {
    return pid;
  }

  public void setPid(Long pid) {
    this.pid = pid;
  }

  public String getPname() {
    return pname;
  }

  public void setPname(String pname) {
    this.pname = pname;
  }

  public Student getStudent() {
    return student;
  }

  public void setStudent(Student student) {
    this.student = student;
  }

  public Set getSets() {
    return sets;
  }

  public void setSets(Set sets) {
    this.sets = sets;
  }

  public List getLists() {
    return lists;
  }

  public void setLists(List lists) {
    this.lists = lists;
  }

  public Map getMap() {
    return map;
  }

  public void setMap(Map map) {
    this.map = map;
  }

  public Properties getProperties() {
    return properties;
  }

  public void setProperties(Properties properties) {
    this.properties = properties;
  }

  private List lists;
  
  private Map map;
  
  private Properties properties;
}

```

### 配置文件的配置  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
    <!-- 
      
     -->
  <bean id="person_Con" class="cn.itcast.spring.sh.di.constructor.Person">
    <constructor-arg index="0" type="java.lang.Long" value="1"></constructor-arg>
    <constructor-arg index="1" value="aaa"></constructor-arg>
    <constructor-arg index="2" ref="student_Con"></constructor-arg>
  </bean>
  
  <bean id="student_Con" class="cn.itcast.spring.sh.di.constructor.Student"></bean>
</beans>
```

### SPringIOC和DC实现MVC实例

### PersonAction 

```
package cn.itcast.spring.sh.mvc;

public class PersonAction {
  private PersonService personService;

  public PersonService getPersonService() {
    return personService;
  }

  public void setPersonService(PersonService personService) {
    this.personService = personService;
  }
  
  public void savePerson(){
    this.personService.savePerson();
  }
}
```

### PersonService  

```
package cn.itcast.spring.sh.mvc;

public interface PersonService {
  public void savePerson();
}
```

### PersonServiceImpl  

```
package cn.itcast.spring.sh.mvc;

public class PersonServiceImpl implements PersonService{

  private PersonDao personDao;
  public PersonDao getPersonDao() {
    return personDao;
  }
  public void setPersonDao(PersonDao personDao) {
    this.personDao = personDao;
  }
  @Override
  public void savePerson() {
    // TODO Auto-generated method stub
    this.personDao.savePerson();
  }

}
```   

### PersonDao  

```
package cn.itcast.spring.sh.mvc;

public interface PersonDao {
  public void savePerson();
}
```

### PersonDaoImpl  

```
package cn.itcast.spring.sh.mvc;

public class PersonDaoImpl implements PersonDao{

  @Override
  public void savePerson() {
    // TODO Auto-generated method stub
    System.out.println("save person dao");
  }
  
}
```

### 配置文件  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
  <bean id="personDao" class="cn.itcast.spring.sh.mvc.PersonDaoImpl"></bean>
  
  <bean id="personService" class="cn.itcast.spring.sh.mvc.PersonServiceImpl">
    <property name="personDao">
      <ref bean="personDao"/>
    </property>
  </bean>
  
  <bean id="personAction" class="cn.itcast.spring.sh.mvc.PersonAction">
    <property name="personService">
      <ref bean="personService"/>
    </property>
  </bean>
</beans>
```  

### 完成