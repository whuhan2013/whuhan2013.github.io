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


### 缓存  

### session的产生方式  

1. sessionFactory.openSession 产生一个新的session
2. sessionFactory.getCurrentSession 获取当前线程中的session  

### session.getCurrentSession的用法        
1、在hibernate的配置文件中：

```
        <property name="current_session_context_class">thread</property>
```

2、不需要写session.close方法，在事务提交的时候会自动关闭(由hibernate内部完成)    
3、crud都需要事务
      1、因为是一个线程，所以整个方法中一个session,一个事务
      2、保证了整个业务操作的安全性  


### 缓存        

   1、缓存的生命周期      
   2、数据库的数据是怎么样放入到缓存中的           
   3、缓存中的数据是怎么样放入到数据库中的    
   4、客户端怎么样从缓存中把数据取出来        
   5、客户端怎么样把一个数据放入到缓存中      
   6、怎么样把一个对象从缓存中取出       
   7、把缓存中所有的数据清空        

### session缓存

1、生命周期就是session的生命周期       
2、一级缓存存放的数据都是私有数据         
        把session存放在threadlocal中，不同的线程是不能访问的，所以保证了数据的安全性          
3、怎么样把数据存放到一级缓存中         
        利用session.save/update/load/get方法都可以存放在一级缓存中       
4、利用session.get/load方法可以把数据从一级缓存中取出           
5、session.evict方法可以把一个对象从一级缓存中清空            
6、利用session.clear方法可以把session中的所有的数据清空        
7、利用session.Refresh方法把数据库中的数据同步到缓存中              
8、session.flush       
    在session的缓存内部，会去检查所有的持久化对象        
           1、如果一个持久化对象没有ID值，则会发出insert语句         
           2、如果一个持久化对象有ID值，则会去检查快照进行对比，如果一样，则什么都不做，如果不一样，则发出update语句       
           3、检查所有的持久化对象是否有关联对象        
                检查关联对象的级联操作     
                检查关联对象的关系操作            
   9、批量操作     

### 实例  

```
package cn.itcast.hibernate.sh.test;

import java.util.Set;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.junit.Test;

import cn.itcast.hiberate.sh.domain.Classes;
import cn.itcast.hiberate.sh.domain.Student;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;

public class SessionCacheTest extends HiberanteUtils{
  static{
    url = "hibernate.cfg.xml";
  } 
  /**
   * session.get方法把数据存放在一级缓存中了
   */
  @Test
  public void testGet(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes = (Classes)session.get(Classes.class, 1L);
    classes = (Classes)session.get(Classes.class,1L);
    transaction.commit();
  }
  
  /**
   * session.load方法把数据存放在一级缓存中
   */
  @Test
  public void testLoad(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes = (Classes)session.load(Classes.class, 1L);
    classes.getCname();
    classes = (Classes)session.load(Classes.class,1L);
    classes.getCname();
    transaction.commit();
  }
  
  /**
   * session.save方法把数据保存在一级缓存中
   */
  @Test
  public void testSave(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes = new Classes();
    classes.setCname("aaa");
    classes.setDescription("asfd");
    session.save(classes);
    classes = (Classes)session.get(Classes.class, classes.getCid());
    transaction.commit();
  }
  
  /**
   * session.update方法把数据保存在一级缓存中
   */
  @Test
  public void testUpdate(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes = (Classes)session.get(Classes.class, 1L);
    session.evict(classes);//classes对象从session中清空了
    session.update(classes);//把classes对象放入到了session缓存中
    classes = (Classes)session.get(Classes.class, 1L);
    transaction.commit();
  }
  
  /**
   * session.clear
   */
  @Test
  public void testClear(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes = (Classes)session.get(Classes.class, 1L);
    session.clear();//classes对象从session中清空了
    classes = (Classes)session.get(Classes.class, 1L);
    transaction.commit();
  }
  
  @Test
  public void testClearTest(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes  = (Classes)session.get(Classes.class, 1L);
    session.clear();//如果不加这句话，两个不同的对象，相同的ID值，所以得把其中的一个清空
    Classes classes2 = new Classes();
    classes2.setCid(1L);
    classes2.setCname("asfd");
    session.update(classes2);
    transaction.commit();
  }
  
  /**
   * 把数据库中的数据刷新到缓存中
   */
  @Test
  public void testRefresh(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes = (Classes)session.get(Classes.class, 1L);
    classes.setCname("66");
    session.refresh(classes);//把cid为1的值从数据库刷到了缓存中
    System.out.println(classes.getCname());
    transaction.commit();
  }
  
  /**
   * session.flush
   */
  @Test
  public void testFlush(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes  =(Classes)session.get(Classes.class, 1L);
    classes.setCname("afdsasdf");
    Set<Student> students = classes.getStudents();
    for(Student student:students){
      student.setDescription("asdf");
    }
    session.flush();
    transaction.commit();
  }
  //插入大量数据
  @Test
  public void testSaveBatch(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    for(int i=6;i<1000000;i++){
      Classes classes = new Classes();
      classes.setCname("aaa");
      classes.setDescription("afds");
      session.save(classes);
      if(i%50==0){
        session.flush();
        session.clear();
      }
    }
    transaction.commit();
  }
  
  /**
   * session.flush只是发出SQL语句了，并没有清空session缓存
   */
  @Test
  public void testFlush2(){
    Session session = sessionFactory.getCurrentSession();
    Transaction transaction = session.beginTransaction();
    Classes classes = (Classes)session.get(Classes.class, 1L);
    session.flush();
    classes = (Classes)session.get(Classes.class, 1L);
    transaction.commit();
  }
}
```     

### 二级缓存   

存放公有数据      
   1、适用场合：        
        1、数据不能频繁更新          
        2、数据能公开，私密性不是很强           
   2、hibernate本身并没有提供二级缓存的解决方案             
   3、二级缓存的实现是依赖于第三方供应商完成的        
         ehcache        
         oscache         
         jbosscache         
         swamchache                 
   4、二级缓存的操作               
         1、二级缓存存在sessionFactory中          
         2、生命周期：与sessionFactory保持一致            
         3、使用二级缓存的步骤

```           
             1、在hibernate.cfg.xml 
                  <property name="cache.use_second_level_cache">true</property>
                  <property name="cache.provider_class">
                        org.hibernate.cache.EhCacheProvider
                  </property>
             2、让某一个对象进入到二级缓存中
                 * 在配置文件中
                   <class-cache usage="read-only" class="cn.itcast.hiberate.sh.domain.Classes"/>
                 *  在映射文件中
                    <cache usage="read-only"/>
             3、使用
                  session.get/session.load
```

### 实例  

```
package cn.itcast.hibernate.sh.test;

import java.util.List;
import java.util.Set;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.junit.Test;

import cn.itcast.hiberate.sh.domain.Classes;
import cn.itcast.hiberate.sh.domain.Student;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;

public class SecondCacheTest extends HiberanteUtils{
  static{
    url = "hibernate.cfg.xml";
  } 
  
  /**
   * session.get
   *    把数据存在一级缓存和二级缓存
   */
  @Test
  public void testGet(){
    Session session = sessionFactory.openSession();
    Classes classes = (Classes)session.get(Classes.class, 1L);
    session.close();
    session = sessionFactory.openSession();
    classes = (Classes)session.get(Classes.class, 1L);
    session.close();
  }
  
  /**
   * session.load
   *   同上
   */
  @Test
  public void testLoad(){
    Session session = sessionFactory.openSession();
    Classes classes = (Classes)session.load(Classes.class, 1L);
    classes.getCname();
    session.close();
    session = sessionFactory.openSession();
    classes = (Classes)session.load(Classes.class, 1L);
    classes.getCname();
    session.close();
  }
  
  /**
   * session.update   
   * 
   */
  @Test
  public void testUpdate(){
    Session session = sessionFactory.openSession();
    //session.beginTransaction();
    Classes classes = new Classes();
    classes.setCid(1L);
    classes.setCname("aaa");
    session.update(classes);
    
    session.close();
    session = sessionFactory.openSession();
    classes = (Classes)session.get(Classes.class, 1L);
    session.close();
  }
  
  @Test
  public void testAllClasses(){
    Session session = sessionFactory.openSession();
    List<Classes> classesList = session.createQuery("from Classes").list();
    session.close();
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }
}
```

### 查询缓存

使用方式，在classes.hbm.xml中开启 开启<property name="cache.use_query_cache">true</property>  

### 实例 

```
package cn.itcast.hibernate.sh.test;

import java.util.List;
import java.util.Set;

import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.junit.Test;

import cn.itcast.hiberate.sh.domain.Classes;
import cn.itcast.hiberate.sh.domain.Student;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;

public class QueryCacheTest extends HiberanteUtils{
  static{
    url = "hibernate.cfg.xml";
  } 
  
  @Test
  public void testQuery(){
    Session session = sessionFactory.openSession();
    Query query = session.createQuery("from Classes");
    query.setCacheable(true);//classes里的所有的数据要往查询缓存中存放了
    List<Classes> classesList = query.list();
    query = session.createQuery("from Classes");
    query.setCacheable(true);
    classesList = query.list();
    session.close();
  }
}
```

### 二级缓存大量数据解决方案，将一部分数据存储在磁盘中  

在src文件夹下添加ehcache.xml  

```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
         
    <diskStore path="C:\\TEMP1"/>
    <defaultCache
            maxElementsInMemory="12"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            maxElementsOnDisk="10000000"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU"
            />
            
    <Cache
            name="cn.itcast.hiberate.sh.domain.Classes"
            maxElementsInMemory="5" 
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="true"
            maxElementsOnDisk="10000000"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU"
            />
</ehcache>
```

### 示例  

```
@Test
  public void testAllClasses(){
    Session session = sessionFactory.openSession();
    List<Classes> classesList = session.createQuery("from Classes").list();
    session.close();
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
```

### HQL语句    

### 单表查询  

```
public List<Classes> queryAllClasses(){
    Session session = sessionFactory.openSession();
    List<Classes> cList = session.createQuery("from Classes").list();
    session.close();
    return cList;
  }
  //条件查询
  public List queryClasses_Properties(){
    Session session = sessionFactory.openSession();
    List cList = session.createQuery("select cid,cname from Classes").list();
    session.close();
    return cList;
  }
  //带构造函数的查询
  public List<Classes> queryClasses_Constructor(){
    Session session = sessionFactory.openSession();
    List<Classes> cList = session.createQuery("select new cn.itcast.hiberate.sh.domain.Classes(cname,description) from Classes").list();
    session.close();
    return cList;
  }
  
  public Classes queryClasses_Condition(){
    Session session = sessionFactory.openSession();
    Query query = session.createQuery("select new cn.itcast.hiberate.sh.domain.Classes(cname,description) from Classes where cid=:cid");
    query.setLong("cid", 1L);
    Classes classes = (Classes)query.uniqueResult();
    System.out.println(classes.getCname());
    session.close();
    return classes;
  }
  
  public Classes queryClasses_Condition_2(){
    Session session = sessionFactory.openSession();
    Query query = session.createQuery("select new cn.itcast.hiberate.sh.domain.Classes(cname,description) from Classes where cid=?");
    query.setLong(0, 3L);
    Classes classes = (Classes)query.uniqueResult();
    System.out.println(classes.getCname());
    session.close();
    return classes;
  }
```

### 子查询  

```
  /**
   * order by,group by,sun,min,max,avg,having等都适用
   * @return
   */
  
  /**
   * 子查询
   */
  public void queryClasses_SubSelect(){
    Session session = sessionFactory.openSession();
    List<Classes> cList = session.createQuery("from Classes where cid in(select cid from Classes where cid in(1,2,3))").list();
    session.close();
  }
```

  ### 一对多查询     


```
  /**
      * 一对多
      *    等值连接          查询出来的机构很差  
      *    内连接       查询出来的机构很差
      *    左外连接  
      *    迫切左外连接
      */
  public List<Classes> queryClasses_Student_EQ(){
    Session session = sessionFactory.openSession();
    List<Classes> cList = session.createQuery("from Classes c,Student s where c.cid=s.classes.cid").list();
    session.close();
    return cList;
  }
  
  /**
   * 内连接
   * @return
   */
  public List<Classes> queryClasses_Student_INNER(){
    Session session = sessionFactory.openSession();
    List<Classes> cList = session.createQuery("from Classes c inner join c.students").list();
    session.close();
    return cList;
  }
  
  /**
   * 迫切内连接
   * @return
   */
  public List<Classes> queryClasses_Student_INNER_FETCH(){
    Session session = sessionFactory.openSession();
    List<Classes> cList = session.createQuery("from Classes c inner join fetch c.students").list();
    session.close();
    return cList;
  }
  
  /**
   * 左外连接
   */
  public List<Classes> queryClasses_Student_LeftJoin(){
    Session session = sessionFactory.openSession();
    List<Classes> cList = session.createQuery("from Classes c left outer join c.students").list();
    session.close();
    return cList;
  }
  
  /**
   * 迫切左外连接
   */
  public List<Classes> queryClasses_Student_LeftJoin_fetch(){
    Session session = sessionFactory.openSession();
    String hql = "from Classes c left outer join fetch c.students";
    hql = "from Student s left outer join fetch s.classes c";
    List<Classes> cList = session.createQuery(hql).list();
    session.close();
    return cList;
  }
  
  /**
   * 带select的查询
   */
  public List<Classes> queryClasses_Student_Select(){
    Session session = sessionFactory.openSession();
    String hql = "select new cn.itcast.hiberate.sh.domain.ClassesView(c.cname,s.sname) " +
             "from Student s left outer join s.classes c";
    List<Classes> cList = session.createQuery(hql).list();
    session.close();
    return cList;
  }
```

### 多对多    

```
  /**
   * 多对多
   */
  public void testQueryCourse_Student(){
    Session session = sessionFactory.openSession();
    List<Student> studentList = session.createQuery("from Student s inner join fetch s.courses c").list();
    session.close();
  }
  ```

### 一对多结合多对多  

```
/**
   * 一对多结合多对多
   */
  public List<Student> queryClasses_Student_Course(){
    Session session = sessionFactory.openSession();
    String hql = "from Classes cs inner join fetch cs.students s inner join fetch s.courses c";
    hql = "from Student s inner join fetch s.classes cs inner join fetch s.courses c";
    //hql = "from Classes cs left outer join fetch cs.students s left outer join fetch s.courses c";
    List<Student> cList = session.createQuery(hql).list();
    
    //迫切主外连接会产生重复，将set转化成list，去除重复的部分
//    Set<Classes> cset = new HashSet<Classes>(cList);
//    cList = new ArrayList<Classes>(cset);
    System.out.println(cList.size());
//    for(Classes classes:cList){
//      System.out.println(classes.getCid());
//      Set<Student> students = classes.getStudents();
//      for(Student student:students){
//        System.out.println(student.getSname());
//        Set<Course> courses = student.getCourses();
//        for(Course course:courses){
//          System.out.println(course.getCname());
//        }
//      }
//    }
    session.close();
    return cList;
  }
```

### 总结   

1、如果页面上要显示的数据和数据库中的数据相差甚远，利用带select的构造器进行查询是比较好的方案      
2、如果页面上要显示的数据和数据库中的数据相差不是甚远，这个时候用迫切连接         
3、如果采用迫切连接，from后面跟的那个对象就是结构主体         
4、如果多张表进行查询，找核心表，找中间关联的那张表


### 注意    
在WEB项目中，hibernate.cf.xml中需要添加以下内容  

```
<property name="connection.driver_class">
    com.mysql.jdbc.Driver
  </property>

<property name="hibernate.dialect">
    org.hibernate.dialect.MySQLInnoDBDialect
  </property>
```


### 完成
