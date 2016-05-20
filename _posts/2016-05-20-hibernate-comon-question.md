---
layout: post
title: Hibernate常见问题
date: 2016-5-20
categories: blog
tags: [java]
description: Hibernate常见问题
---   

### 问题1，hql条件查询报错

执行Query session.createQuery(hql) 报错误直接跳到finally

### 解决方案

 加入

```
<prop key="hibernate.query.factory_class">org.hibernate.hql.classic.ClassicQueryTranslatorFactory</prop>
```
 节点

### 加入之后再次报错  

```
org.hibernate.HibernateException: could not instantiate QueryTranslatorFactory: org.hibernate.hql.classic.ClassicQueryTransactionFactory
```

### 解决方案  


### 修改成以下 

```
<property name="hibernate.query.factory_class">org.hibernate.hql.internal.classic.ClassicQueryTranslatorFactory</property>
```

### 参考链接

[Hibernate 问题,在执行Query session.createQuery(hql) 报错误直接跳到finally - morning99的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/morning99/article/details/17807077)

[org.hibernate.HibernateException: could not instantiate QueryTranslatorFactory: org.hibernate.hql.classic.ClassicQueryTransactionFactory - Stack Overflow](http://stackoverflow.com/questions/5582478/org-hibernate-hibernateexception-could-not-instantiate-querytranslatorfactory)


### hql条件查询   


### 查询最后一条数据 

```
public RunTimePO getLast(String sin)
    {
        
         Session session = sessionFactory.openSession();
         //System.out.println("22222222222222");
         String hql="from RunTimePO where SIN=? order by id desc";   
         //String hql="from RunTimePO where SIN=?";
          Query query = session.createQuery(hql); 
          query.setString(0, sin);
         
          query.setMaxResults(1);
          //query.setInteger(0, 139);  
          //int id=(Integer) query.uniqueResult();
     
         RunTimePO rp=(RunTimePO) query.uniqueResult();  
            
         if(rp!=null)
         {
         System.out.println("id==="+rp.getId());
         }
            session.close();
            return rp; 
    }
```


### 时间段查询

```
public List<RunTimePO> getBetweenSeg(String sin,String startTime,String endTime)
    {
        Session session = sessionFactory.openSession();
        
         String hql="from RunTimePO as u where SIN=? and u.date between ? and ?"; 
         Query query = session.createQuery(hql);
         query.setString(0, sin);
         query.setString(1, startTime);
         query.setString(2, endTime);
         System.out.println(query.list().size());
         List<RunTimePO> runTimeList=query.list();
         return runTimeList;
    }

```

### 完成
