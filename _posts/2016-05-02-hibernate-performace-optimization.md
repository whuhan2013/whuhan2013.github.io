---
layout: post
title: Hibernate之性能优化
date: 2016-5-02
categories: blog
tags: [java]
description: Hibernate之性能优化
---   

### 本文主要包括以下内容  

### 性能优化的方法  

发出的sql语句越少，性能越高           
 方法：                   
 1、懒加载                    
 2、抓取策略                
 3、缓存策略                    
 4、HQL语句                   


### 懒加载  

- 类的懒加载             
        1、利用session.load方法可以产生代理对象           
        2、在session.load方法执行的时候并不发出sql语句       
        3、在得到其一般属性的时候发出sql语句                
        4、只针对一般属性有效，针对标示符属性是无效的          
        5、默认情况就是懒加载                 

- 集合的懒加载            

1.  lazy= "false"  当session.get时，集合就被加载出来了
2.  lazy=" true"   在遍历集合的时候才加载
3.  lazy="extra"    针对集合做count,min,max,sum等操作
     
- 单端关联的懒加载(多对一)

```
        <many-to-one lazy="false/no-proxy/proxy">  no-porxy 默认值  true
```                   
  根据多的一端加载一的一端，就一个数据，所以无所谓


### 总结      
懒加载主要解决了一个问题：类、集合、many-to-one在时候发出SQL语句，加载数据 


### 实例    

### Classes.hbm.xml   

```
<!-- 
               
              lazy  
              true
              false
              extra 进一步的懒加载   
         -->
        <set name="students" cascade="save-update" inverse="true" lazy="extra" fetch="join">
            <!-- 
                key是用来描述外键
             -->
            <key>
                <column name="cid"></column>
            </key>
            <one-to-many class="cn.itcast.hiberate.sh.domain.Student"/>
        </set>
```

### lazyTest.java  

```
package cn.itcast.hibernate.sh.test;

import java.util.Set;

import org.hibernate.Session;
import org.junit.Test;

import cn.itcast.hiberate.sh.domain.Classes;
import cn.itcast.hiberate.sh.domain.Student;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;

public class LazyTest extends HiberanteUtils{
    static{
        url = "hibernate.cfg.xml";
    }
    /**
     * 类的懒加载
     */
    @Test
    public void testLoad(){
        Session session = sessionFactory.openSession();
        Classes classes = (Classes)session.load(Classes.class, 1L);
        
        System.out.println(classes.getCname());
        session.close();
        
    }
    
    /**
     * 集合的延迟加载
     */
    @Test
    public void testSet(){
        Session session = sessionFactory.openSession();
        Classes classes = (Classes)session.get(Classes.class, 10L);
        Set<Student> students = classes.getStudents();
        for(Student student:students){//这个时候才要发出sql语句
            System.out.println(student.getSname());
        }
        session.close();
    }
    
    /**
     * 集合的延迟加载
     */
    @Test
    public void testSet_EXTRA(){
        Session session = sessionFactory.openSession();
        Classes classes = (Classes)session.get(Classes.class, 10L);
        Set<Student> students = classes.getStudents();
        System.out.println(students.size());
        session.close();
    }
}
```  

### 经典错误            
could not initialize proxy no session             
在关闭session后取值          


### 抓取策略  

1、研究的主要是set集合如何提取数据
2、在Classes.hbm.xml文件中

```
       <set fetch="join/select/subselect">
            join        左外连接
               如果把需求分析翻译sql语句，存在子查询，这个时候用该策略不起作用
            select      默认
               先查询一的一端，再查询多的一端
            subselect   子查询
               如果需要分析翻译成sql语句存在子查询，这个时候用该策略效率最高
```

只要查询的不是一个班级的所有学生信息的时候，就是子查询,通常来说 


### FetchTest.java  

```
package cn.itcast.hibernate.sh.test;

import java.util.List;
import java.util.Set;

import org.hibernate.Session;
import org.junit.Test;

import cn.itcast.hiberate.sh.domain.Classes;
import cn.itcast.hiberate.sh.domain.Student;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;

public class FetchTest extends HiberanteUtils{
    static{
        url = "hibernate.cfg.xml";
    }
    
    /**
     * n+1的问题
     *    解决问题的方案：子查询  fetch="subselect"
     */
    @Test
    public void testAll_Classes(){
        Session session = sessionFactory.openSession();
        List<Classes> cList = session.createQuery("from Classes").list();
        for(Classes classes:cList){
            Set<Student> students = classes.getStudents();
            for(Student student:students){
                System.out.println(student.getSname());
            }
        }
        session.close();
    }
    
    /**
     * n+1的问题
     *    解决问题的方案：子查询  fetch="subselect"
     *    查询cid为1的学生
     */
    @Test
    public void testClasses_Some(){
        Session session = sessionFactory.openSession();
        List<Classes> cList = session.createQuery("from Classes where cid in(1,2,3)").list();
        for(Classes classes:cList){
            Set<Student> students = classes.getStudents();
            for(Student student:students){
                System.out.println(student.getSname());
            }
        }
        session.close();
    }
    
    /**
     * 查询一个班级的所有学生 
     */
    @Test
    public void testQueryClasses_Id(){
        Session session = sessionFactory.openSession();
        Classes classes = (Classes)session.get(Classes.class, 1L);
        Set<Student> students = classes.getStudents();
        for(Student student:students){
            System.out.println(student.getSname());
        }
        session.close();
    }
}
```  


### 懒加载和抓取策略结合：研究对象是set集合  

|fetch|lazy|sql| 什么时候发出sql |说明|
|:---:|:---:|:---:|:---:|:---:|
| join |false| 存在子查询|当查询classes时把 classes和student全部查询出来|这种情况下join没有用|
| join|true|存在子查询|当遍历student时发出查询student|join没用|
| join|true|不是子查询|在session.get(classes)时全部查询出来|这时，lazy没有用|
| subselect|true/false|存在子查询|发出两条sql语句|如果lazy为true,在遍历集合，如果lazy为false,在一开始就发出|
| select |true/false||发出n+1条sql|如果lazy为true,在遍历集合，如果lazy为false,在一开始就发出|


### 完成
