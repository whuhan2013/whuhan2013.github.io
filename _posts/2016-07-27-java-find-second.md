---
layout: post
title: JAVA求职专项练习(二)
date: 2016-7-27
categories: blog
tags: [求职]
description: 
---

**1、静态初始化代码块、构造代码块、构造方法**               

```
当涉及到继承时，按照如下顺序执行：
1、执行父类的静态代码块 
static {
        System.out.println("static A");
    }
输出:static A
2、执行子类的静态代码块
static {
        System.out.println("static B");
    }
输出:static B
3、执行父类的构造代码块
{
        System.out.println("I’m A class");
    }
输出:I'm A class
4、执行父类的构造函数
public HelloA() {
    }
输出：无
5、执行子类的构造代码块
{
        System.out.println("I’m B class");
    }
输出:I'm B class
6、执行子类的构造函数
public HelloB() {
    }
输出：无
```  

父类Ｂ静态代码块->子类Ａ静态代码块->父类Ｂ非静态代码块->父类Ｂ构造函数->子类Ａ非静态代码块->子类Ａ构造函数

**2、关于PreparedStatement与Statement描述错误的是**          
1:创建时的区别：         
    Statement statement = conn.createStatement();           
    PreparedStatement preStatement = conn.prepareStatement(sql);      

执行的时候:                                           
    ResultSet rSet = statement.executeQuery(sql);              
    ResultSet pSet = preStatement.executeQuery();                  
由上可以看出，PreparedStatement有预编译的过程，已经绑定sql，之后无论执行多少遍，都不会再去进行编译，
而 statement 不同，如果执行多变，则相应的就要编译多少遍sql，所以从这点看，preStatement 的效率会比 Statement要高一些

安全性问题          
这个就不多说了，preStatement是预编译的，所以可以有效的防止 SQL注入等问题      
所以 preStatement 的安全性 比 Statement 高 

代码的可读性 和 可维护性            
这点也不用多说了，你看老代码的时候  会深有体会
preStatement更胜一筹          

总得来说，PreparedStatement都优于Statement,尽量用PreparedStatement.

**3、关于volatile关键字，下列描述不正确的是？**           
一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：       
1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。  
2）禁止进行指令重排序。      

volatile只提供了保证访问该变量时，每次都是从内存中读取最新值，并不会使用寄存器缓存该值——每次都会从内存中读取。  
而对该变量的修改，volatile并不提供原子性的保证。                          
由于及时更新，很可能导致另一线程访问最新变量值，无法跳出循环的情况              
多线程下计数器必须使用锁保护。                   

**4、java中，StringBuilder和StringBuffer的区别，下面说法错误的是？**          
效率：StringString(大姐，出生于JDK1.0时代) 不可变字符序列   
  <StringBuffer(二姐，出生于JDK1.0时代)  线程安全的可变字符序列    <StringBuilder(小妹，出生于JDK1.5时代) 非线程安全的可变字符序列   。Java中的String是一个类，而并非基本数据类型。string是值传入，不是引用传入。      
  String对String  类型进行改变的时候其实都等同于生成了一个新的  String  对象，然后将指针指向新的  String  对象，而不是StringBuffer；StringBuffer每次结果都会对  StringBuffer  对象本身进行操作，而不是生成新的对象，再改变对象引用。 

StringBuffer和StringBuilder可以算是双胞胎了，这两者的方法没有很大区别。但在线程安全性方面，StringBuffer允许多线程进行字符操作。 这是因为在源代码中StringBuffer的很多方法都被关键字 synchronized  修饰了，而StringBuilder没有。 StringBuilder的效率比StringBuffer稍高，如果不考虑线程安全，StringBuilder应该是首选。另外，JVM运行程序主要的时间耗费是在创建对象和回收对象上。           

**5、在运行时，由java解释器自动引入，而不用import语句引入的包是()。**        
java.lang包是java语言的核心包，lang是language的缩写                                                                
java.lang包定义了一些基本的类型，包括Integer,String之类的，是java程序必备的包，有解释器自动引入，无需手动导入

**6、SpringMVC的原理**   
SpringMVC是Spring中的模块，它实现了mvc设计模式的web框架，首先用户发出请求，请求到达SpringMVC的前端控制器（DispatcherServlet）,前端控制器根据用户的url请求处理器映射器查找匹配该url的handler，并返回一个执行链，前端控制器再请求处理器适配器调用相应的handler进行处理并返回给前端控制器一个modelAndView，前端控制器再请求视图解析器对返回的逻辑视图进行解析，最后前端控制器将返回的视图进行渲染并把数据装入到request域，返回给用户。               

DispatcherServlet作为springMVC的前端控制器，负责接收用户的请求并根据用户的请求返回相应的视图给用户。            


**7、实现或继承了Collection接口的是（）**             
在java.util包中提供了一些集合类，常用的有List、Set和Map类，其中List类和Set类继承了Collection接口。这些集合类又称为容器，长度是可变的，数组用来存放基本数据类型的数据，集合用来存放类对象的引用。        
List接口、Set接口、Map接口以及Collection接口的主要特征如下：                
Collection接口是List接口和Set接口的父接口，通常情况下不被直接使用。             
List接口继承了Collection接口，List接口允许存放重复的对象，排序方式为按照对象的插入顺序。       
Set接口继承了Collection接口，Set接口不允许存放重复的对象，排序方式为按照自身内部的排序规则。            
Map接口以键值对（key—value）的形式存放对象，其中键（key）对象不可以重复，值（value）对象可以重复，排序方式为按照自身内部的规则。           
C：Vector实现了List接口，即间接实现Collection接口         
D：Iterator是Java迭代器最简单的实现，没有实现Collection接口            

![](http://uploadfiles.nowcoder.com/images/20150904/458054_1441352361202_2B9A4774B9C5C489A9E9564854FFCD6C)

**8、下面有关SPRING的事务传播特性，说法错误的是？**            
PROPAGATION_REQUIRED--支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。        
PROPAGATION_SUPPORTS--支持当前事务，如果当前没有事务，就以非事务方式执行。              
PROPAGATION_MANDATORY--支持当前事务，如果当前没有事务，就抛出异常。           
PROPAGATION_REQUIRES_NEW--新建事务，如果当前存在事务，把当前事务挂起。                      
PROPAGATION_NOT_SUPPORTED--以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。              
PROPAGATION_NEVER--以非事务方式执行，如果当前存在事务，则抛出异常。              


**8、以下哪些类是线程安全的**    
A，Vector相当于一个线程安全的List                             
B，HashMap是非线程安全的，其对应的线程安全类是HashTable           
C，Arraylist是非线程安全的，其对应的线程安全类是Vector             
D，StringBuffer是线程安全的，相当于一个线程安全的StringBuilder               
E，Properties实现了Map接口，是线程安全的            

Properties继承自HashTable， Properties中的 Store()方法把一个Properties对象的内容以一种可读的形式保存到一个文件中。Load()方法正好相反，用来读取文件，并设定Properties对象来包含keys和values。
Properties类用来方便 的读写配置文件，支持key-value形式和xml形式的配置文件，以key-value为例， Properties的load方法直接将文件读取到内存中并且以map形式来保存，用 getProperty("key")方法来取得对应的vaule值。


**9、jvm的垃圾回收方式**        
两个最基本的java回收算法：复制算法和标记清理算法           
复制算法：两个区域A和B，初始对象在A，继续存活的对象被转移到B。此为新生代最常用的算法       
标记清理：一块区域，标记要回收的对象，然后回收，一定会出现碎片，那么引出     
标记-整理算法：多了碎片整理，整理出更大的内存放更大的对象                   

两个概念：新生代和年老代           
新生代：初始对象，生命周期短的         
永久代：长时间存在的对象             
整个java的垃圾回收是新生代和年老代的协作，这种叫做分代回收。       
P.S：Serial New收集器是针对新生代的收集器，采用的是复制算法                   
Parallel New（并行）收集器，新生代采用复制算法，老年代采用标记整理       
Parallel  Scavenge（并行）收集器，针对新生代，采用复制收集算法        
Serial Old（串行）收集器，新生代采用复制，老年代采用标记清理       
Parallel   Old（并行）收集器，针对老年代，标记整理           
CMS收集器，基于标记清理                                  
G1收集器：整体上是基于标记清理，局部采用复制               

综上：新生代基本采用复制算法，老年代采用标记整理算法。cms采用标记清理。



**10、Java异常类**           
![](http://img.blog.csdn.net/20151023200049836?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
通常，Java的异常(包括Exception和Error)分为 可查的异常（checked exceptions）和不可查的异常（unchecked exceptions） 。 

可查异常（编译器要求必须处置的异常）： 正确的程序在运行中，很容易出现的、情理可容的异常状况 。 可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常 状况，就必须采取某种方式进行处理。

除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。
不可查异常(编译器不要求强制处置的异常):包括运行时异常（RuntimeException与其子类）和错误（Error）。

**11、下面有关struts1和struts2的区别，描述错误的是**　　　　　　　　　　　　
从action类上分析:
1.Struts1要求Action类继承一个抽象基类。Struts1的一个普遍问题是使用抽象类编程而不是接口。 
2. Struts 2 Action类可以实现一个Action接口，也可实现其他接口，使可选和定制的服务成为可能。Struts2提供一个ActionSupport基类去实现常用的接口。Action接口不是必须的，任何有execute标识的POJO对象都可以用作Struts2的Action对象。　　

从Servlet 依赖分析: 
3. Struts1 Action 依赖于Servlet API ,因为当一个Action被调用时HttpServletRequest 和 HttpServletResponse 被传递给execute方法。 　　　　　　　　
4. Struts 2 Action不依赖于容器，允许Action脱离容器单独被测试。如果需要，Struts2 Action仍然可以访问初始的request和response。但是，其他的元素减少或者消除了直接访问HttpServetRequest 和 HttpServletResponse的必要性。　　　　

从action线程模式分析: 　　　　　　　　　　
5. Struts1 Action是单例模式并且必须是线程安全的，因为仅有Action的一个实例来处理所有的请求。单例策略限制了Struts1 Action能作的事，并且要在开发时特别小心。Action资源必须是线程安全的或同步的。 　　　　　　　　　
6. Struts2 Action对象为每一个请求产生一个实例，因此没有线程安全问题。（实际上，servlet容器给每个请求产生许多可丢弃的对象，并且不会导致性能和垃圾回收问题）　　　　　　　　　　

**12、ArrayList list = new ArrayList(20);中的list扩充几次**        
ArrayList的构造函数总共有三个：         
（1）ArrayList()构造一个初始容量为 10 的空列表。           
（2）ArrayList(Collection<? extends E> c)构造一个包含指定 collection 的元素的列表，这些元素是按照该 collection 的迭代器返回它们的顺序排列的。           
（3）ArrayList(int initialCapacity)构造一个具有指定初始容量的空列表。           
调用的是第三个构造函数，直接初始化为大小为20的list，没有扩容，所以选择A       

**13、下面哪一项不是加载驱动程序的方法？**　　　　　
加载驱动方法　　　　　　　　　　
1.Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");　　　
2. DriverManager.registerDriver(new com.mysql.jdbc.Driver());　　　　　　　
3.System.setProperty("jdbc.drivers", "com.mysql.jdbc.Driver");　　　　　　　　

**14、关于ThreadLocal以下说法正确的是**　　　　　　
1、ThreadLocal的类声明：　　　　　　　　
public class ThreadLocal<T>　　　
可以看出ThreadLocal并没有继承自Thread，也没有实现Runnable接口。所以AB都不对。　　　　　　　
2、ThreadLocal类为每一个线程都维护了自己独有的变量拷贝。每个线程都拥有了自己独立的一个变量。　　　
所以ThreadLocal重要作用并不在于多线程间的数据共享，而是数据的独立，C选项错。　　　　　　
由于每个线程在访问该变量时，读取和修改的，都是自己独有的那一份变量拷贝，不会被其他线程访问，　
变量被彻底封闭在每个访问的线程中。所以E对。　　　　　　　
3、ThreadLocal中定义了一个哈希表用于为每个线程都提供一个变量的副本：　　　　　　　　　　　　　


**15、java类加载过程**     
类加载过程

类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：加载（Loading）、验证（Verification）、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段。其中准备、验证、解析3个部分统称为连接（Linking）。如图所示。

![](http://img.blog.csdn.net/20160308184325593)

**16、结构型模式中最体现扩展性的模式是**              
结构型模式是描述如何将类对象结合在一起，形成一个更大的结构，结构模式描述两种不同的东西：类与类的实例。故可以分为类结构模式和对象结构模式。       
在GoF设计模式中，结构型模式有：         
1.适配器模式 Adapter              
  适配器模式是将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。            
  两个成熟的类需要通信，但是接口不同，由于开闭原则，我们不能去修改这两个类的接口，所以就需要一个适配器来完成衔接过程。                    
2.桥接模式 Bridge                    
  桥接模式将抽象部分与它的实现部分分离，是它们都可以独立地变化。它很好的支持了开闭原则和组合锯和复用原则。实现系统可能有多角度分类，每一种分类都有可能变化，那么就把这些多角度分离出来让他们独立变化，减少他们之间的耦合。   
3.组合模式 Composite           
  组合模式将对象组合成树形结构以表示部分-整体的层次结构，组合模式使得用户对单个对象和组合对象的使用具有一致性。
4.装饰模式 Decorator             
装饰模式动态地给一个对象添加一些额外的职责，就增加功能来说，它比生成子类更灵活。也可以这样说，装饰模式把复杂类中的核心职责和装饰功能区分开了，这样既简化了复杂类，有去除了相关类中重复的装饰逻辑。 装饰模式没有通过继承原有类来扩展功能，但却达到了一样的目的，而且比继承更加灵活，所以可以说装饰模式是继承关系的一种替代方案。        
5.外观模式 Facade           
 外观模式为子系统中的一组接口提供了同意的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。
外观模式中，客户对各个具体的子系统是不了解的，所以对这些子系统进行了封装，对外只提供了用户所明白的单一而简单的接口，用户直接使用这个接口就可以完成操作，而不用去理睬具体的过程，而且子系统的变化不会影响到用户，这样就做到了信息隐蔽。                
6.享元模式 Flyweight             
 享元模式为运用共享技术有效的支持大量细粒度的对象。因为它可以通过共享大幅度地减少单个实例的数目，避免了大量非常相似类的开销。.
      享元模式是一个类别的多个对象共享这个类别的一个对象，而不是各自再实例化各自的对象。这样就达到了节省内存的目的。        
7.代理模式 Proxy               
为其他对象提供一种代理，并由代理对象控制对原对象的引用，以间接控制对原对象的访问。      

