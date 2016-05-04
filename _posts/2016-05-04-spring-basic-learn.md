---
layout: post
title: Spring基础知识
date: 2016-5-04
categories: blog
tags: [java]
description: Spring基础知识
---   

### 本文主要包括以下内容  

1. 注解  

### 注解   

   1、注解就是为了说明java中的某一个部分的作用(Type)      
   2、注解都可以用于哪个部门是@Target注解起的作用              
   3、注解可以标注在ElementType枚举类所指定的位置上              
   4、 

```
         @Documented    //该注解是否出现在帮助文档中         
         @Retention(RetentionPolicy.RUNTIME) //该注解在java,class和运行时都起作用               
         @Target(ElementType.ANNOTATION_TYPE)//该注解只能用于注解上                  
         public @interface Target {            
             ElementType[] value();                    
         }    
```   
              
   5、用来解析注解的类成为注解解析器  

### 自定义注解实例  

### 类注解  

```
package cn.itcast.annotation.sh;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Description {
  String value();
}

```

### 方法注解  

```
package cn.itcast.annotation.sh;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MethodDesc {
  String name();
}
```  

### 注解的使用  


```
package cn.itcast.annotation.sh;

@Description("whuhan university")
public class ITCAST_SH {
  @MethodDesc(name="whu")
  public void java(){
    System.out.println("this is java");
  }
}
```

### 注解解析 

```
package cn.itcast.annotation.sh;

import java.lang.reflect.Method;

import org.junit.Test;

public class AnnotationParse {
  @Test
  public void testParse(){
    Class class1 = ITCAST_SH.class;
    if(class1.isAnnotationPresent(Description.class)){
      Description description = (Description)class1.getAnnotation(Description.class);
      if(description.value().contains("whuhan")){
        System.out.println("学费减半");
      }
    }
    Method[] methods = class1.getMethods();
    for(Method method:methods){
      if(method.isAnnotationPresent(MethodDesc.class)){
        MethodDesc methodDesc = (MethodDesc)method.getAnnotation(MethodDesc.class);
        if(methodDesc.name().contains("whu")){
          System.out.println("折上折");
        }
      }
    }
  }
}
```

### @Resource注解的使用(依赖注入)

1、在spring的配置文件中导入命名空间
         
```         xmlns:context="http://www.springframework.org/schema/context"
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context-2.5.xsd
```
  
2、引入注解解析器

```
        <context:annotation-config></context:annotation-config>
``` 

3、在spring的配置文件中把bean引入进来        
4、在一个类的属性上加

```
            @Resource(name="student_annotation")
            private Student student;
         从该注解本身
               @Target({TYPE, FIELD, METHOD})
               @Retention(RUNTIME)
               public @interface Resource {
                  String name() default "";
               }
```
  1、该注解可以用于属性上或者方法上，但是一般用户属性上        
 2、该注解有一个属性name,默认值为""           
  5、分析整个过程：             
        1、当启动spring容器的时候，spring容器加载了配置文件          
        2、在spring配置文件中，只要遇到bean的配置，就会为该bean创建对象            
        3、在纳入spring容器的范围内查找所有的bean,看哪些bean的属性或者方法上加有@Resource           
        4、找到@Resource注解以后，判断该注解name的属性是否为""(name没有写)                        
              如果没有写name属性，则会让属性的名称的值和spring中ID的值做匹配，如果匹配成功则赋值                 
                                        如果匹配不成功，则会按照类型进行匹配，如果匹配不成功，则报错                        
              如果有name属性，则会按照name属性的值和spring的bean中ID进行匹配，匹配成功，则赋值，不成功则报错                   


### 实例  

```
package cn.itcast.spring.annotation.di;

public class Student {
  public void show(){
    System.out.println("student");
  }
}
```

### Person.java  

```
package cn.itcast.spring.annotation.di;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

import javax.annotation.Resource;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;

public class Person {
  private Long pid;
  private String pname;
  
  @Resource(name="student_annotation")
//  @Autowired
//  @Qualifier("student")
  private Student student;
  
  private Set sets;
  
  private List lists;
  
  private Map map;
  
  private Properties properties;
  
  public void showStudent(){
    this.student.show();
  }
}

```

### XML文件  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd">
  <!-- 
      注解解析器
     -->
  <context:annotation-config></context:annotation-config>
  <bean id="person_annotation" class="cn.itcast.spring.annotation.di.Person"></bean>

  <bean id="student_annotation" class="cn.itcast.spring.annotation.di.Student"></bean>
</beans>
``` 


### 类扫描的注解Component使用  

1、在spring的配置文件中导入命名空间
         
```         xmlns:context="http://www.springframework.org/schema/context"
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context-2.5.xsd
```

  2、加入下面的语句        
  
```
<context:component-scan base-package="cn.itcast.annotation.scan"></context:component-scan>
``` 
       1、 该注解解析器包含了两个功能：依赖注入和类扫描          
       2、在base-package包及子包下查找所有的类     
           
  3、如果一个类上加了@Component注解，就会进行如下的法则

```
        如果其value属性的值为""          
              @Component
              public class Person {}
              ==
              <bean id="person" class="..Person">
       如果其value属性的值不为""
              @Component("p")
              public class Person {}
              ==
              <bean id="p" class="..Person">
```

  4、按照@Resource的法则再次进行操作  

### 实例  

```
package cn.itcast.annotation.scan;

import org.springframework.stereotype.Component;

@Component("b")
//<bean id="b" class="..Student">
public class Student {
  public void show(){
    System.out.println("student");
  }
}
```

### person.java  

```
package cn.itcast.annotation.scan;

import javax.annotation.Resource;

import org.springframework.stereotype.Component;

@Component("a")
//<bean id="a" class="..Person">
public class Person {
  
  @Resource(name="b")
  private Student student;

  public void showStudent(){
    this.student.show();
  }
}
```  

### XML文件的配置  

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd">
    <!-- 
      类扫描的注解解析器
         component 指的就是一个类
         base-package 在该包及子包中进行扫描
     -->
  <context:component-scan base-package="cn.itcast.annotation.scan"></context:component-scan>
</beans>
```


### 新特性  

Spring 2.5引入了更多典型化注解(stereotype annotations)： @Component、@Service和 @Controller。 @Component是所有受Spring管理组件的通用形式； 而@Repository、@Service和 @Controller则是@Component的细化， 用来表示更具体的用例(例如，分别对应了持久化层、服务层和表现层)。也就是说， 你能用@Component来注解你的组件类， 但如果用@Repository、@Service或@Controller来注解它们，你的类也许能更好地被工具处理，或与切面进行关联。 例如，这些典型化注解可以成为理想的切入点目标。当然，在Spring Framework以后的版本中，@Repository、@Service和 @Controller也许还能携带更多语义。如此一来，如果你正在考虑服务层中是该用 @Component还是@Service， 那@Service显然是更好的选择。同样的，就像前面说的那样， @Repository已经能在持久化层中进行异常转换时被作为标记使用了。

### 示例  

### PersonAction(控制层)


```
package cn.itcast.annotation.mvc;

import javax.annotation.Resource;

import org.springframework.stereotype.Controller;

@Controller("personAction")
public class PersonAction {
  @Resource(name="personService")
  private PersonService personService;
  
  public void savePerson(){
    this.personService.savePerson();
  }
}
```

### PersonServiceImpl(service层)

```
package cn.itcast.annotation.mvc;

import javax.annotation.Resource;

import org.springframework.stereotype.Service;

@Service("personService")
public class PersonServiceImpl implements PersonService{

  @Resource(name="personDao")
  private PersonDao personDao;
  
  @Override
  public void savePerson() {
    // TODO Auto-generated method stub
    this.personDao.savePerson();
  }

}
```

### PersonDaoImpl(dao层)

```
package cn.itcast.annotation.mvc;

import org.springframework.stereotype.Repository;

@Repository("personDao")
public class PersonDaoImpl implements PersonDao{

  @Override
  public void savePerson() {
    // TODO Auto-generated method stub
    System.out.println("save person dao");
  }
  
}
```

### xml与注解：        
   1、xml书写麻烦，但是效率高              
   2、注解书写简单，但是效率低                  

### 完成 

