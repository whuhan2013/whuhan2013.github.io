---
layout: post
title: JAVA求职专项练习(一)
date: 2016-7-25
categories: blog
tags: [求职]
description: 
---

**1、Thread. sleep()是否会抛出checked exception?**               
checked exception：指的是编译时异常，该类异常需要本函数必须处理的，用try和catch处理，或者用throws抛出异常，然后交给调用者去处理异常。                  
runtime exception：指的是运行时异常，该类异常不必须本函数必须处理，当然也可以处理。           
Thread.sleep()抛出的InterruptException属于checked exception；IllegalArgumentException属于Runtime exception;      

**2、关于struts框架，下面那些说法是正确的？**               
A，structs可以进行文件上传          
B，structs基于MVC模式，MVC是模型，视图，控制器，是一种设计模式              
C，structs框架让流程结构更清晰             
D，structs需要很多 action类，会增加类文件数目                

**工作机制：**        
Struts的工作流程:                    
在web应用启动时就会加载初始化ActionServlet,ActionServlet从 
struts-config.xml文件中读取配置信息,把它们存放到各种配置对象 
当ActionServlet接收到一个客户请求时,将执行如下流程. 

(1)检索和用户请求匹配的ActionMapping实例,如果不存在,就返回请求路径无效信息;           
(2)如果ActionForm实例不存在,就创建一个ActionForm对象,把客户提交的表单数据保存到ActionForm对象中;        
(3)根据配置信息决定是否需要表单验证.如果需要验证,就调用ActionForm的validate()方法;       
(4)如果ActionForm的validate()方法返回或返回一个不包含ActionMessage的ActuibErrors对象, 就表示表单验证成功;       
(5)ActionServlet根据ActionMapping所包含的映射信息决定将请求转发给哪个Action,如果相应的Action实例不存在,就先创建这个实例,然后调用Action的execute()方法;                
(6)Action的execute()方法返回一个ActionForward对象,ActionServlet在把客户请求转发给 ActionForward对象指向的JSP组件;  
(7)ActionForward对象指向JSP组件生成动态网页,返回给客户;          

为什么要用：            
JSP、Servlet、JavaBean技术的出现给我们构建强大的企业应用系统提供了可能。但用这些技术构建的系统非常的繁乱，所以在此之上，我们需要一个规则、一个把这些技术组织起来的规则，这就是框架，Struts便应运而生。            
基于Struts开发的应用由3类组件构成：控制器组件、模型组件、视图组件                 

**3、下面有关List接口、Set接口和Map接口的描述，错误的是？**              
List接口中的对象按一定顺序排列，允许重复        
Set接口中的对象没有顺序，但是不允许重复              
Map接口中的对象是key、value的映射关系，key不允许重复                  
Map接口和Collection接口是同一等级的，所以并不是继承自collection             

**4、有如下4条语句：()**          
Integer i01=59;          
int i02=59;                              
Integer i03=Integer.valueOf(59);         
Integer i04=new Integer(59);            

Integer i01 = 59. 直接赋值数字，java会自动装箱，自动调用Integer.valueOf(59).    
Integer i03 = Integer.valueOf(59).  Integer.valueOf(int i)会返回一个Integer对象，当i在-128~127之间时，会返回缓存中已创建的Integer对象。               
Integer i04 = new Integer(59) 返回一个新的对象。              
所以这道题中，59在-128~127之间，所以前三条语句返回的是同一个对象（在缓存区已创建的对象），而i04使用new新创建了一个新的对象，所以i04与前面三个对象都不一样。           

**5、HttpSession session = request.getSession(false) 与HttpSession session = request.getSession(true)的区别？**   
HttpSession session = request.getSession(boolean create)                
返回当前请求的会话。如果当前请求不属于任何会话，而且create参数为true，则创建一个会话，否则返回null。此后所有来自同一个的请求都属于这个会话，通过它的getSession返回的是当前会话。   

- equest.getSession() 等价于 request.getSession(true)             
这两个方法的作用是相同的，查找请求中是否有关联的session id，如果有则返回这个号码所对应的session对象，如果没有则生成一个新的session对象。所以说，通过此方法是一定可以获得一个session对象。
- request.getSession(false) 查找请求中是否有关联的session id，如果有则返回这个号码所对应的session对象，如果没有则返回一个null。


**6、关于形参的说法正确的是？**             
A：形式参数可被视为localvariable。形参和局部变量一样都不能离开方法。都只有在方法内才会发生作用，也只有在方法中使用，不会在方法外可见。            
B： 对于形式参数只能用final修饰符，其它任何修饰符都会引起编译器错误。但是用这个修饰符也有一定的限制，就是在方法中不能对参数做任何修改。 不过一般情况下，一个方法的形参不用final修饰。只有在特殊情况下，那就是：方法内部类。一个方法内的内部类如果使用了这个方法的参数或者局部变量的话，这个参数或局部变量应该是final。               
C：形参的值在调用时根据调用者更改，实参则用自身的值更改形参的值（指针、引用皆在此列），也就是说真正被传递的是实参。 
D：方法的参数列表指定要传递给方法什么样的信息，采用的都是对象的形式。因此，在参数列表中必须指定每个所传递对象的类型及名字。想JAVA中任何传递对象的场合一样，这里传递的实际上也是引用，并且引用的类型必须正确。--《Thinking in JAVA》

**7、在使用super 和this关键字时，以下描述正确的是**             
1）调用super()必须写在子类构造方法的第一行，否则编译不通过。每个子类构造方法的第一条语句，都是隐含地调用super()，如果父类没有这种形式的构造函数，那么在编译的时候就会报错。                 
2）super()和this()类似,区别是，super从子类中调用父类的构造方法，this()在同一类内调用其它方法。         
3）super()和this()均需放在构造方法内第一行。                
4）尽管可以用this调用一个构造器，但却不能调用两个。                
5）this和super不能同时出现在一个构造函数里面，因为this必然会调用其它的构造函数，其它的构造函数必然也会有super语句的存在，所以在同一个构造函数里面有相同的语句，就失去了语句的意义，编译器也不会通过。            
6）this()和super()都指的是对象，所以，均不可以在static环境中使用。包括：static变量,static方法，static语句块。   
7）从本质上讲，this是一个指向本对象的指针, 然而super是一个Java关键字。                    
8）构造器中this()表示调用形式参数相同的同一个类中的另一个构造器，这样就可以代码复用       


**8、java的垃圾回收机制**            
垃圾回收主要针对的是堆区的回收，因为栈区的内存是随着线程而释放的。堆区分为三个区：年轻代（Young Generation）、年老代（Old Generation）、永久代（Permanent Generation，也就是方法区）。

年轻代：对象被创建时（new）的对象通常被放在Young（除了一些占据内存比较大的对象）,经过一定的Minor GC（针对年轻代的内存回收）还活着的对象会被移动到年老代（一些具体的移动细节省略）。      
年老代：就是上述年轻代移动过来的和一些比较大的对象。Minor GC(FullGC)是针对年老代的回收         
永久代：存储的是final常量，static变量，常量池。           

**9、下面有关JVM内存，说法错误的是？**          
运行时数据区包括：虚拟机栈区，堆区，方法区，本地方法栈，程序计数器      

- 虚拟机栈区 ：也就是我们常说的栈区，线程私有，存放基本类型，对象的引用和 returnAddress ，在编译期间完成分配。   
- 堆区 ， JAVA 堆，也称 GC 堆，所有线程共享，存放对象的实例和数组， JAVA 堆是垃圾收集器管理的主要区域。          
- 方法区 ：所有线程共享，存储已被虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码等数据。这个区域的内存回收目标主要是针对常量池的对象的回收和对类型的卸载。                                    
- 程序计数器 ：线程私有，每个线程都有自己独立的程序计数器，用来指示下一条指令的地址。                         
- 本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native方法服务。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun HotSpot虚拟机）直接就把本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。        

**10、下面有关 java 类加载器,说法正确的是?**           
类的加载是由类加载器完成的，类加载器包括：引导类加载器（ BootStrap ）、扩展加载器（ Extension ）、系统加载器（ System ）和用户自定义类加载器（ java.lang.ClassLoader 的子类）。从 Java 2 （ JDK 1.2 ）开始，类加载过程采取了父亲委托机制（ PDM ）。 PDM 更好的保证了 Java 平台的安全性，在该机制中， JVM 自带的 Bootstrap 是根加载器，其他的加载器都有且仅有一个父类加载器。类的加载首先请求父类加载器加载，父类加载器无能为力时才由其子类加载器自行加载。 JVM 不会向 Java 程序提供对 Bootstrap 的引用。下面是关于几个类加载器的说明： 

- Bootstrap ：一般用本地代码实现，负责加载 JVM 基础核心类库（ rt.jar ）；
- Extension ：从 java.ext.dirs 系统属性所指定的目录中加载类库，它的父加载器是 Bootstrap ；
- system class loader ：又叫应用类加载器，其父类是 Extension 。它是应用最广泛的类加载器。它从环境变量 classpath 或者系统属性 java.class.path 所指定的目录中记载类，是用户自定义加载器的默认父加载器。
- 用户自定义类加载器： java.lang.ClassLoader 的子类 

父类委托机制是可以修改的，有些服务器就是自定义类加载器优先的。