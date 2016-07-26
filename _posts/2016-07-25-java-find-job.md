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


**11、在Java中，关于HashMap类的描述，以下错误的是**                
HashMap不能保证元素的顺序,HashMap能够将键设为null，也可以将值设为null，与之对应的是Hashtable,(注意大小写：不是HashTable)，Hashtable不能将键和值设为null，否则运行时会报空指针异常错误；
HashMap线程不安全，Hashtable线程安全.                
HashMap是Hashtable的轻量级实现（非线程安全的实现），他们都完成了Map接口，
主要区别在于HashMap允许空（null）键值（key）,由于非线程安全，效率上可能高于Hashtable。
HashMap 把Hashtable的contains方法去掉了，改成containsvalue和containsKey。因为contains方法容易让人引起误解。 


**12、抽象类和接口**                
相同点：都不能被实例化,位于继承树的顶端，都包含抽象方法            
不同点：1、设计目的：接口体现的一种规范，类似与整个系统的总纲，制订了系统各模块应该遵循的标准，因此接口不应该经常改变，一旦改变对整个系统是辐射性的。                     
抽象类作为多个子类的共同父类，体现的是一种模板式设计，可以当作系统实现过程中的中间产品，已经实现了系统部分功能。
2、使用不同：（1）接口只能包含抽象方法，抽象类可以包含普通方法。              
（2）接口里不能定义静态方法，抽象类可以。                     
（3）接口只能定义静态常量属性不能定义普通属性，抽象类可以。             
（4）接口不包含构造器，抽象类可以（不是用于创建对象而是让子类完成初始化）。             
（5）接口里不能包含初始化块，抽象类完全可以。              
（6）接口多继承，抽象类但继承（只能有一个直接父类）。                
总结：接口所有方法全是抽象方法只能 public abstract修饰 （默认public abstract修饰 ），属性默认public static final修饰。            
抽象类除了包含抽象方法外与普通类无区别。                    

**13、 JAVA 异常类**             
![](http://uploadfiles.nowcoder.com/images/20151113/140047_1447376765880_373DC390B08E99ABC340DB1F78F35FCB)
都是Throwable的子类：           
1.Exception（异常） :是程序本身可以处理的异常。            
2.Error（错误）:               是程序无法处理的错误。这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，一般不需要程序处理。    
3.检查异常（编译器要求必须处置的异常） ：  除了Error，RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。            
4.非检查异常(编译器不要求处置的异常): 包括运行时异常（RuntimeException与其子类）和错误（Error）。           


**14、Servlet的生命周期**           
- init()：仅执行一次，负责在装载Servlet时初始化Servlet对象             
- service() ：核心方法，一般HttpServlet中会有get,post两种处理方式。在调用doGet和doPost方法时会构造servletRequest和servletResponse请求和响应对象作为参数。           
- destory()：在停止并且卸载Servlet时执行，负责释放资源        
初始化阶段：Servlet启动，会读取配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，将ServletConfig作为参数来调用init()方法。所以选ACD。B是在调用service方法时才构造的         

每一次请求来到容器时，会产生HttpServletRequest与HttpServlceResponse对象，并在调用service()方法时当做参数传入。
在WEB容器启动后，会读取Servlet设置信息，将Servlet类加载并实例化，并为每个Servlet设置信息产生一个ServletConfig对象，而后调用Servlet接口的init()方法，并将产生的ServletConfig对象当作参数传入。

**15、下面有关final, finally, finalize的区别描述错误的是**          
final修饰的方法不能被覆盖      
final修饰的字段为常量              
final修饰的类不能被继承               

final        
修饰符（关键字）如果一个类被声明为final，意味着它不能再派生出新的子类，不能作为父类被继承。因此一个类不能既被声明为 abstract的，又被声明为final的。将变量或方法声明为final，可以保证它们在使用中不被改变。被声明为final的变量必须在声明时给定初值，而在以后的引用中只能读取，不可修改。

finally        
异常处理时提供 finally 块来执行任何清除操作。如果抛出一个异常，那么相匹配的 catch 子句就会执行，然后控制就会进入 finally 块（如果有的话）。一般异常处理块需要。

finalize              
方法名。Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的。它是在 Object 类中定义的，因此所有的类都继承了它。子类覆盖 finalize() 方法以整理系统资源或者执行其他清理工作。finalize() 方法是在垃圾收集器删除对象之前对这个对象调用的。 
Java中所有类都从Object类中继承finalize()方法。
当垃圾回收器(garbage colector)决定回收某对象时，就会运行该对象的finalize()方法。


**16、变量修饰符，重写，与重载**       
![](http://uploadfiles.nowcoder.com/images/20151012/458054_1444618871663_E93E59ACFE1791E0A5503384BEBDC544)

方法的重写（override）两同两小一大原则：        
方法名相同，参数类型相同         
子类返回类型小于等于父类方法返回类型，         
子类抛出异常小于等于父类方法抛出异常，        
子类访问权限大于等于父类方法访问权限。           


**17、对象序列化**           
使用ObjectOutputStream和ObjectInputStream可以将对象进行传输.
声明为static和transient类型的成员数据不能被串行化。因为static代表类的状态， transient代表对象的临时数据。


**18、对于线程局部存储TLS(thread local storage)，以下表述正确的是**        
同一全局变量或者静态变量每个线程访问的是同一变量，多个线程同时访存同一全局变量或者静态变量时会导致冲突，尤其是多个线程同时需要修改这一变量时，通过TLS机制，为每一个使用该全局变量的线程都提供一个变量值的副本，每一个线程均可以独立地改变自己的副本，而不会和其它线程的副本冲突。