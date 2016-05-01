---
layout: post
title: Hibernate基础知识
date: 2016-5-01
categories: blog
tags: [java]
description: Hibernate基础知识
---   

### 本文主要包括以下内容  

1. 对象的状态 
2. 一对多的单向关联
3. 一对多的双向关联
4. 多对多关联   


### 对象状态的变化  

![这里写图片描述](http://img.blog.csdn.net/20160501132514452)

### 对象的状态
- 临时状态    
      new       
 
-  持久化状态      
      get,save,update
      
- 脱管状态          
      clear  close  evict        


### 一对多单向操作，以班级表与学生表为例  


### Classes.java   

```
package cn.itcast.hiberate.sh.domain;

import java.io.Serializable;
import java.util.Set;

public class Classes implements Serializable{
    private Long cid;
    private String cname;
    private String description;
    
    public Long getCid() {
        return cid;
    }

    public void setCid(Long cid) {
        this.cid = cid;
    }

    public String getCname() {
        return cname;
    }

    public void setCname(String cname) {
        this.cname = cname;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public Set<Student> getStudents() {
        return students;
    }

    public void setStudents(Set<Student> students) {
        this.students = students;
    }

    private Set students;
}

```


### Student.java  

```
package cn.itcast.hiberate.sh.domain;

import java.io.Serializable;

public class Student implements Serializable{
    private Long sid;
    private String sname;
    public Long getSid() {
        return sid;
    }
    public void setSid(Long sid) {
        this.sid = sid;
    }
    public String getSname() {
        return sname;
    }
    public void setSname(String sname) {
        this.sname = sname;
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    private String description;
}
```

### class.hbm.xml 

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="cn.itcast.hiberate.sh.domain.Classes">
        <id name="cid" length="5" type="java.lang.Long">
            <generator class="increment"></generator>
        </id>
        <property name="cname" length="20" type="java.lang.String"></property>
        
        <property name="description" length="100" type="java.lang.String"></property>
        <!-- 
            set元素对应类中的set集合
            通过set元素使classes表与student表建立关联
               key是通过外键的形式让两张表建立关联
               one-to-many是通过类的形式让两个类建立关联
            
            cascade 级联
               save-update
                1、当 保存班级的时候，对学生进行怎么样的操作
                     如果学生对象在数据库中没有对应的值，这个时候会执行save操作
                     如果学生对象在数据库中有对应的值，这个时候会执行update操作
               delete   删除班级时同时删除学生
               all      save-update与delete都用 
            inverse  维护关系
               true      不维护关系     
               false     维护关系
               default   false
         -->
        <set name="students" cascade="all" inverse="false">
            <!-- 
                key是用来描述外键
             -->
            <key>
                <column name="cid"></column>
            </key>
            <one-to-many class="cn.itcast.hiberate.sh.domain.Student"/>
        </set>
    </class>
</hibernate-mapping>
```

### Student.hbm.xml 

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="cn.itcast.hiberate.sh.domain.Student">
        <id name="sid" length="5">
            <generator class="increment"></generator>
        </id>
        <property name="sname" length="20"></property>
        <property name="description" length="100"></property>
    </class>
</hibernate-mapping>
```

### 配置文件  

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
        作用：根据持久化类和映射文件生成表
        validate
        create-drop
        create
        update
    -->
    <property name="hbm2ddl.auto">update</property>
    <!-- 
        显示hibernate内部生成的sql语句
    -->
    <property name="show_sql">true</property>
    <mapping resource="cn/itcast/hiberate/sh/domain/Classes.hbm.xml" />
    <mapping resource="cn/itcast/hiberate/sh/domain/Student.hbm.xml" />

</session-factory>
</hibernate-configuration>
```


### 级联的概念与操作  

```
cascade 级联
               save-update
                1、当 保存班级的时候，对学生进行怎么样的操作
                     如果学生对象在数据库中没有对应的值，这个时候会执行save操作
                     如果学生对象在数据库中有对应的值，这个时候会执行update操作
               delete   删除班级时同时删除学生
               all      save-update与delete都用 
            inverse  维护关系
               true      不维护关系     
               false     维护关系
               default   false
```  


### 一对多增删改查实例  


```
package cn.itcast.hibernate.sh.test;

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.annotations.Type;
import org.junit.Test;

import cn.itcast.hiberate.sh.domain.Classes;
import cn.itcast.hiberate.sh.domain.Student;
import cn.itcast.hibernate.sh.utils.HiberanteUtils;

/**
 * 1、新建一个班级
 * 2、新建一个学生
 * 3、新建一个班级的时候同时新建一个学生
 * 4、已经存在一个班级，新建一个学生，建立学生与班级之间的关系
 * 5、已经存在一个学生，新建一个班级，把学生加入到该班级
 * 6、把一个学生从一个班级转移到另一个班级
 * 7、解析一个班级和一个学生之间的关系
 * 8、解除一个班级和一些学生之间的关系
 * 9、解除该班级和所有的学生之间的关系
 * 10、已经存在一个班级，已经存在一个学生，建立该班级与该学生之间的关系
 * 11、已经存在一个班级，已经存在多个学生，建立多个学生与班级之间的关系
 * 12、删除学生
 * 13、删除班级
 *     删除班级的时候同时删除学生
 *     在删除班级之前，解除班级和学生之间的关系
 * @author Think
 *
 */
public class OneToManySingleTest extends HiberanteUtils{
    @Test
    public void testSaveClasses(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Classes classes = new Classes();
        classes.setCname("空间信息与数字技术");
        classes.setDescription("pretty good");
        session.save(classes);
        transaction.commit();
        session.close();
    }
    
    
    
    @Test
    public void testSaveStudent(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Student student = new Student();
        student.setSname("班长");
        student.setDescription("老牛：很牛");
        session.save(student);
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testSaveClasses_Student(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Classes classes = new Classes();
        classes.setCname("空间信息与数字技术2:");
        classes.setDescription("很牛X");
        
        Student student = new Student();
        student.setSname("班长");
        student.setDescription("老牛：很牛X");
        
        session.save(student);
        session.save(classes);
        transaction.commit();
        session.close();
    }
    
    /**
     * 在保存班级的时候，级联保存学生
     */
    @Test
    public void testSaveClasses_Cascade_Student_Save(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Classes classes = new Classes();
        classes.setCname("空间信息与数字技术3:");
        classes.setDescription("很牛XX");
        
        Student student = new Student();
        student.setSname("班长");
        student.setDescription("老牛：很牛XX");
        
        Set<Student> students = new HashSet<Student>();
        students.add(student);
        
        //建立classes与student之间的关联
        classes.setStudents(students);
        session.save(classes);
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testSaveClasses_Cascade_Student_Update(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Classes classes = new Classes();
        classes.setCname("空间信息与数字技术4:");
        classes.setDescription("niubility");
        
        Student student = (Student)session.get(Student.class, 2L);
        
        student.setSname("secentary");
        student.setDescription("it is pretty beautiful");
        Set<Student> students = new HashSet<Student>();
        students.add(student);
        
        classes.setStudents(students);
    
        session.save(classes);
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testUpdateClasses_Cascade_Student_Save(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Classes classes = (Classes)session.get(Classes.class, 5L);
        Student student = new Student();
        student.setSname("class flower");
        student.setDescription("she is pretty girl");
        classes.getStudents().add(student);
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testUpdateClasses_Cascade_Student_Update(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Classes classes = (Classes)session.get(Classes.class, 5L);
        Set<Student> students = classes.getStudents();//为cid为5的班级的所有的学生
        for(Student student:students){
            student.setDescription("under pressure");
        }
        transaction.commit();
        session.close();
    }
    
    /**
     * 一个错误的演示
     */
    @Test
    public void testSaveClasses_Cascade_Student_Save_Error(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Classes classes = new Classes();
        classes.setCname("空间信息与数字技术5:");
        classes.setDescription("很牛XXXXXX");
        
        Student student = new Student();
        student.setSname("班长XXXXXX");
        student.setDescription("老牛：很牛XXXXXX");
        
        Set<Student> students = new HashSet<Student>();
        students.add(student);
        
        //建立classes与student之间的关联
        classes.setStudents(students);
        session.save(classes);
        transaction.commit();
        session.close();
    }
    
    
    /**
     * 已经存在一个班级，新建一个学生，建立学生与班级之间的关系
     *    通过更新班级级联保存学生  cascade
     *    建立班级和学生之间的关系  inverse
     */
    @Test
    public void testSaveStudent_R_1(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Student student = new Student();
        student.setSname("technical");
        student.setDescription("good at thechnical");
        Classes classes = (Classes)session.get(Classes.class, 1L);
        
        classes.getStudents().add(student);
        
        transaction.commit();
        session.close();
    }
    
    /**
     *  Hibernate: select classes0_.cid as cid0_0_, classes0_.cname as cname0_0_, classes0_.description as descript3_0_0_ from Classes classes0_ where classes0_.cid=?
        Hibernate: select max(sid) from Student
        Hibernate: select students0_.cid as cid0_1_, students0_.sid as sid1_, students0_.sid as sid1_0_, students0_.sname as sname1_0_, students0_.description as descript3_1_0_ from Student students0_ where students0_.cid=?
        Hibernate: insert into Student (sname, description, sid) values (?, ?, ?)
        更新关系的操作
        Hibernate: update Student set cid=? where sid=?
     */
    @Test
    public void testSaveStudent_R_2(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Student student = new Student();
        student.setSname("技术班长");
        student.setDescription("大神");
        Classes classes = (Classes)session.get(Classes.class, 1L);
        
        session.save(student);
        
        classes.getStudents().add(student);
        
        transaction.commit();
        session.close();
    }
    
    /**
     * 已经存在一个学生，新建一个班级，把学生加入到该班级
     */
    @Test
    public void testSaveClasses_R(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        
        Classes classes = new Classes();
        classes.setCname("空间信息与数字技术6");
        classes.setDescription("必看，杀手锏");
        
        Student student = (Student)session.get(Student.class, 2L);
        Set<Student> students = new HashSet<Student>();
        students.add(student);
        classes.setStudents(students);
        
        session.save(classes);
        transaction.commit();
        session.close();
    }
    
    /**
     * 把一个学生从一个班级转移到另一个班级
     *  Hibernate: select classes0_.cid as cid0_0_, classes0_.cname as cname0_0_, classes0_.description as descript3_0_0_ from Classes classes0_ where classes0_.cid=?
        Hibernate: select classes0_.cid as cid0_0_, classes0_.cname as cname0_0_, classes0_.description as descript3_0_0_ from Classes classes0_ where classes0_.cid=?
        Hibernate: select student0_.sid as sid1_0_, student0_.sname as sname1_0_, student0_.description as descript3_1_0_ from Student student0_ where student0_.sid=?
        Hibernate: select students0_.cid as cid0_1_, students0_.sid as sid1_, students0_.sid as sid1_0_, students0_.sname as sname1_0_, students0_.description as descript3_1_0_ from Student students0_ where students0_.cid=?
        Hibernate: select students0_.cid as cid0_1_, students0_.sid as sid1_, students0_.sid as sid1_0_, students0_.sname as sname1_0_, students0_.description as descript3_1_0_ from Student students0_ where students0_.cid=?
        Hibernate: update Student set cid=null where cid=? and sid=?
        Hibernate: update Student set cid=? where sid=?
     */
    @Test
    public void testTransformClass(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        //Classes classes5 = (Classes)session.get(Classes.class, 5L);
        Classes classes6 = (Classes)session.get(Classes.class, 6L);
        Student student = (Student)session.get(Student.class, 1L);
        //classes5.getStudents().remove(student);
        classes6.getStudents().add(student);
        transaction.commit();
        session.close();
    }
    
    /**
     * 解除一个班级和一些学生之间的关系
     */
    
    @Test
    public void testR_Some(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Classes classes = (Classes)session.get(Classes.class, 1L);
        Set<Student> students = classes.getStudents();
//      for(Student student:students){
//          if(student.getSid().longValue()==6||student.getSid().longValue()==7){
//              students.remove(student);
//          }
//      }
        //set-->list(set转化成list)
        List<Student> sList = new ArrayList<Student>(students);
        for(int i=0;i<sList.size();i++){
            if(sList.get(i).getSid().longValue()==6||sList.get(i).getSid().longValue()==7){
                sList.remove(sList.get(i));
                i--;
            }
        }
        //list转化成set
        students = new HashSet<Student>(sList);
        classes.setStudents(students);
        /**
         * 增强for循环只能修改一次
         *   1、用普通的for循环
         *   2、新建一个set集合，把原来的set集合要保留的数据存放到新的set集合中
         */
        transaction.commit();
        session.close();
    }
    
    /*
     * 解除班级与所有学生
     * classes.setStudents(null);直接把班级针对student的集合设置为null
     */
    @Test
    public void testRealseAll(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Classes classes = (Classes)session.get(Classes.class, 1L);
//      Set<Student> students = classes.getStudents();
//      students.clear();
        classes.setStudents(null);
        transaction.commit();
        session.close();
    }
    
    /**
     * 已经存在一个班级，已经存在一个学生，建立该班级与该学生之间的关系
     * 11、已经存在一个班级，已经存在多个学生，建立多个学生与班级之间的关系
     */
    
    
    
    //删除一个学生
    @Test
    public void testDeleteStudent(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Student student = (Student)session.get(Student.class, 8L);
        session.delete(student);
        transaction.commit();
        session.close();
    }
    
    
    //删除班级
    @Test
    public void testDeleteClasses(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Classes classes = (Classes)session.get(Classes.class, 5L);
        //classes.setStudents(null);
        session.delete(classes);
        transaction.commit();
        session.close();
    }
    
    @Test
    public void testDeleteClasses_Cascade(){
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Classes classes = (Classes)session.get(Classes.class, 5L);
        session.delete(classes);
        transaction.commit();
        session.close();
    }
    
}
```


### 完成