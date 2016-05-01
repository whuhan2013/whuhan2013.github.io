---
layout: post
title: Hibernate入门
date: 2016-4-30
categories: blog
tags: [java]
description: Hibernate入门
---   

### 本文主要是Hibernate的简单入门  

Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装.

### 优点
1、比较简单
2、数据缓存：一级缓存    二级缓存   查询缓存
3、移植性比较好

### 缺点
1、因为sql语句是hibernate内部生成的，所以程序员干预不了，不可控
2、如果数据库特别大，不适合用hibernate


### hibernate加载流程 


1、hibernate的组成部分   

- 持久化类
    + 实现对应的序列化接口
    + 必须有默认的构造函数
    + 持久化类的属性不能使用关键字
    + 标示符

- 映射文件
    + 类型:java类型和hibernate类型
    + 主键的产生器:increment identity  assigned uuid
    + id  prototype
    + set
    + cascade  对象与对象之间的关系
    + inverse  对象与外键之间的关系

- 配置文件
    + 数据库的链接信息
    + 存放了映射文件的信息
    + 其他信息：hibernate内部功能的信息:<property name="show_sql">true</property>

2、hibernate的流程   

- Configuraction
        加载了配置文件

- SessionFactory
        配置文件的信息、映射文件的信息、持久化类的信息

- Session
        1、crud的操作都是由session完成的
        2、事务是由session开启的
        3、两个不同的session只能用各自的事务
        4、session决定了对象的状态
        5、创建完一个session，相当于打开了一个数据库的链接
        
- Transaction
        1、事务默认不是自动提交的
        2、必须由session开启
        3、必须和当前的session绑定(两个session不可能共用一个事务)         

3、对象的状态的转化       

4、hibernate的原理：       
        根据客户端的代码，参照映射文件，生成sql语句，利用jdbc技术进行数据库的操作


- 创建对象，获得hibernate配置文件  

```
/**
         * 创建configuration对象
         */
        Configuration configuration = new Configuration();
        configuration.configure();
        /**
         * 加载hibernate的配置文件
         */
        //configuration.configure("");
        sessionFactory = configuration.buildSessionFactory()
```

### 序列化类 

```
package cn.itcast.hibernate.sh.domain;

import java.io.Serializable;


/**
 * 对象的序列化的作用：让对象在网络上传输,以二进制的形式传输
 * @author Think
 * Serializable标示接口
 */
public class Person implements Serializable{
    private Long pid;
    private String pname;
    
    public Person(){}
    
    public Person(String pname){
        this.pname = pname;
    }
    
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
    public String getPsex() {
        return psex;
    }
    public void setPsex(String psex) {
        this.psex = psex;
    }
    private String psex;
}
```  

### 映射文件   

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
     
    <class name="cn.itcast.hibernate.sh.domain.Person">
        <!-- 
            id描述标示属性  和数据库中的主键对应
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
            property描述一般属性
         -->
        <property name="pname" column="pname" length="20" type="string">
        </property>
        
        <property name="psex" column="psex" length="10" type="java.lang.String"></property>
    </class>
</hibernate-mapping>
```  


### hibernate配置文件  

```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <!-- 
        一个session-factory只能连接一个数据库
    -->
<session-factory>
    <!-- 
        数据库的用户名
    -->
    <property name="connection.username">root</property>
    <!-- 
        密码
    -->
    <property name="connection.password">root</property>
    <!-- 
        url
    -->
    <property name="connection.url">
        jdbc:mysql://localhost:3306/hibernate
    </property>
    <!-- 
        作用：根据持久化类和映射文件生成表(动态生成表)
        validate      只验证不生成
        create-drop   启动时生成表，结束时删除表
        create           启动时生成
        update           启动时判断是否对应，不对应则创建，对应则验证（常用）
    -->
    <property name="hbm2ddl.auto">update</property>
    <!-- 
        显示hibernate内部生成的sql语句
    -->
    <property name="show_sql">true</property>
    <mapping resource="cn/itcast/hibernate/sh/domain/Person.hbm.xml" />

</session-factory>
</hibernate-configuration>
```   


### 获得sessionFactory，进行CRUD操作

```
/**
1、hibernate把数据库的链接信息、把映射文件的信息、持久化类的信息
2、sessionFactory是由单例模式产生的
3、一般情况下一个hibernate应该有一个数据库链接
4、该类本身是线程安全的
5、是一个重量级类
**/
sessionFactory = configuration.buildSessionFactory();
//  打开了一个数据库的链接，准备进行数据库的操作
sessionFactory.openSession();

//用session进行CRUD操作 
```

### CRUD实现  

```
package cn.itcast.hibernate.sh.test;

import java.io.Serializable;
import java.util.List;






import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;
import org.junit.Test;

import cn.itcast.hibernate.sh.domain.Person;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;


public class PersonTest extends HiberanteUtils{
    @Test
    public void testSavePerson(){
        Session session=sessionFactory.openSession();
        Transaction transaction=session.beginTransaction();
        
        Person person=new Person();
        person.setPname("zj");
        person.setPsex("girl");
        session.save(person);
        
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testQueryPerson()
    {
        Session session=sessionFactory.openSession();
        List<Person> personList=session.createQuery("from Person").list();
        for(Person person:personList)
        {
            System.out.println(person.getPname());
        }
        session.close();
    }
    
    @Test
    public void testQueryPersonByID()
    {
        Session session=sessionFactory.openSession();
        /**
         * 按照主键的方式查询数据库中的记录
         * 第二个参数的类型必须与持久类中标示符的类型一致
         */
        
        Person person=(Person) session.get(Person.class,1L);
        System.out.println(person.getPsex());
        session.close();
    }
    
    /**
     * hibernate内部会检查标示符，看标示符中的值在数据库相应的表中有没有对应的记录，如果有，则删除
     */
    @Test
    public void testDeletePerson(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        /**
         * 方法一
         * 1、根据id把值从数据库中查找出来
         * 2、把对象删除掉
         */
        Person person = (Person)session.get(Person.class, 2L);
        session.delete(person);
        
        /**
         * 方法二
         * 1、新创建一个person对象
         * 2、给person对象的标示符赋值
         * 3、调用session.delete方法删除
         */
        //Person person = new Person();
        //person.setPid(2L);
        //session.delete(person);
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testUpdatePerson(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        /**
         * 1、根据id把持久化对象提取出来
         * 2、进行修改
         * 3、执行upate操作
         */
        Person person = (Person)session.get(Person.class, 1L);
        person.setPsex("不详");
//      Person person = new Person();
//      person.setPid(1L);
        session.update(person);
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testIdentity(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Person person = (Person)session.get(Person.class, 1L);
        Person person2 = new Person();
        //person2.setPid(1L);
        session.update(person2);
        transaction.commit();
        session.close();
    }

}
```  

###注意
利用session.get方法产生一个对象，调用的是默认的构造函数，所以一个持久化类中必须有一个默认的构造函数


### 各种主键生成器

```
package cn.itcast.hibernate.sh.generator;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.junit.Test;

import cn.itcast.hibernate.sh.domain.Person;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;

public class GeneratorTest extends HiberanteUtils{
    /**
     *  Hibernate: select max(pid) from Person
        Hibernate: insert into Person (pname, psex, pid) values (?, ?, ?)
        
        说明：
           1、主键的类型必须是数字
           2、主键的生成是由hibernate内部完成的，程序员不需要干预
           3、这种生成机制效率比较低
     */
    @Test
    public void testIncrement(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Person person = new Person();
        person.setPname("上海");
        person.setPsex("女");
        
        /**
         * 参数必须持久化对象
         */
        session.save(person);
        
        transaction.commit();
        session.close();
    }
    
    /*
     * Hibernate: insert into Person (pname, psex) values (?, ?)
     * 说明
     *   1、新的主键的产生是由数据库完成的，并不是由hibernate或者程序员完成的
     *   2、该表必须支持自动增长机制
     *   3、效率比较高
     */
    @Test
    public void testIdentity(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Person person = new Person();
        person.setPname("上海");
        person.setPsex("女");
        
        /**
         * 参数必须持久化对象
         */
        session.save(person);
        
        transaction.commit();
        session.close();
    }
    
    /**
     * 由程序设置主键
     */
    @Test
    public void testAssigned(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Person person = new Person();
        person.setPname("上海");
        person.setPsex("女");
        
        /**
         * 参数必须持久化对象
         */
        session.save(person);
        
        transaction.commit();
        session.close();
    }
    
    /**
     * 1、UUID是由hibernate内部生成的
     * 2、主键的类型必须是字符串
     */
    @Test
    public void testUUID(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Person person = new Person();
        person.setPname("上海");
        person.setPsex("女");
        
        /**
         * 参数必须持久化对象
         */
        session.save(person);
        
        transaction.commit();
        session.close();
    }
    
    
    /**
     * hibernate内部是根据主键生成器来生成主键的，在客户端设置主键不一定起作用，在assigned的时候起作用
     */
    @Test
    public void testIdentity_ID(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Person person = new Person();
        person.setPid(1L);
        person.setPname("aa");
        person.setPsex("aa");
        session.save(person);
        transaction.commit();
        session.close();
    }
}
```  

### 完成
