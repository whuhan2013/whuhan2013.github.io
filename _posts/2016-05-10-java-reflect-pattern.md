---
layout: post
title: java之反射机制
date: 2016-5-10
categories: blog
tags: [java]
description: java之反射机制
---   

   
### 转载  

原文链接：[http://www.jianshu.com/p/1a60d55a94cd](http://www.jianshu.com/p/1a60d55a94cd)

今天介绍下Java的反射机制，以前我们获取一个类的实例都是使用new一个实例出来。那样太low了，今天跟我一起来学习学习一种更加高大上的方式来实现。


### Java反射机制定义

Java反射机制是指在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
用一句话总结就是反射可以实现在运行时可以知道任意一个类的属性和方法。

### 反射机制的优点与缺点   

为什么要用反射机制？直接创建对象不就可以了吗，这就涉及到了动态与静态的概念

- 静态编译：在编译时确定类型，绑定对象,即通过。
- 动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多态的应用，有以降低类之间的藕合性。

### 优点     

可以实现动态创建对象和编译，体现出很大的灵活性，特别是在J2EE的开发中它的灵活性就表现的十分明显。比如，一个大型的软件，不可能一次就把把它设计的很完美，当这个程序编译后，发布了，当发现需要更新某些功能时，我们不可能要用户把以前的卸载，再重新安装新的版本，假如这样的话，这个软件肯定是没有多少人用的。采用静态的话，需要把整个程序重新编译一次才可以实现功能的更新，而采用反射机制的话，它就可以不用卸载，只需要在运行时才动态的创建和编译，就可以实现该功能。

### 缺点

对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于只直接执行相同的操作。

### 理解Class类和类类型     

想要了解反射首先理解一下Class类，它是反射实现的基础。           
类是java.lang.Class类的实例对象，而Class是所有类的类（There is a class named Class）          
对于普通的对象，我们一般都会这样创建和表示：         

```
Code code1 = new Code();
```

上面说了，所有的类都是Class的对象，那么如何表示呢，可不可以通过如下方式呢：

```
Class c = new Class();
```

但是我们查看Class的源码时，是这样写的：

```
private  Class(ClassLoader loader) { 
    classLoader = loader; 
}
```

可以看到构造器是私有的，只有JVM可以创建Class的对象，因此不可以像普通类一样new一个Class对象，虽然我们不能new一个Class对象，但是却可以通过已有的类得到一个Class对象，共有三种方式，如下：

```
Class c1 = Code.class;
这说明任何一个类都有一个隐含的静态成员变量class，这种方式是通过获取类的静态成员变量class得到的
Class c2 = code1.getClass();
code1是Code的一个对象，这种方式是通过一个类的对象的getClass()方法获得的
Class c3 = Class.forName("com.trigl.reflect.Code");
这种方法是Class类调用forName方法，通过一个类的全量限定名获得
```

这里，c1、c2、c3都是Class的对象，他们是完全一样的，而且有个学名，叫做Code的类类型（class type）。

这里就让人奇怪了，前面不是说Code是Class的对象吗，而c1、c2、c3也是Class的对象，那么Code和c1、c2、c3不就一样了吗？为什么还叫Code什么类类型？这里不要纠结于它们是否相同，只要理解类类型是干什么的就好了，顾名思义，类类型就是类的类型，也就是描述一个类是什么，都有哪些东西，所以我们可以通过类类型知道一个类的属性和方法，并且可以调用一个类的属性和方法，这就是反射的基础。      

举个简单例子代码：

```
public class ReflectDemo {
    public static void main(String[] args) throws ClassNotFoundException {
        //第一种：Class c1 = Code.class;
        Class class1=ReflectDemo.class;
        System.out.println(class1.getName());

        //第二种：Class c2 = code1.getClass();
        ReflectDemo demo2= new ReflectDemo();
        Class c2 = demo2.getClass();
        System.out.println(c2.getName());

        //第三种：Class c3 = Class.forName("com.trigl.reflect.Code");
        Class class3 = Class.forName("com.tengj.reflect.ReflectDemo");
        System.out.println(class3.getName());
    }
}
```

执行结果：

```
com.tengj.reflect.ReflectDemo
com.tengj.reflect.ReflectDemo
com.tengj.reflect.ReflectDemo
```

### Java反射相关操作   

前面我们知道了怎么获取Class，那么我们可以通过这个Class干什么呢？
总结如下：

- 获取成员方法Method
- 获取成员变量Field
- 获取构造函数Constructor

下面来具体介绍

### 获取成员方法信息
单独获取某一个方法是通过Class类的以下方法获得的：

```
public Method getDeclaredMethod(String name, Class<?>... parameterTypes) // 得到该类所有的方法，不包括父类的
public Method getMethod(String name, Class<?>... parameterTypes) // 得到该类所有的public方法，包括父类的
两个参数分别是方法名和方法参数类的类类型列表。
例如类A有如下一个方法：

public void fun(String name,int age) {
        System.out.println("我叫"+name+",今年"+age+"岁");
    }
现在知道A有一个对象a，那么就可以通过：

Class c = Class.forName("com.tengj.reflect.Person");  //先生成class
Object o = c.newInstance();                           //newInstance可以初始化一个实例
Method method = c.getMethod("fun", String.class, int.class);//获取方法
method.invoke(o, "tengj", 10);                              //通过invoke调用该方法，参数第一个为实例对象，后面为具体参数值
```


完整代码如下：

```
public class Person {
    private String name;
    private int age;
    private String msg="hello wrold";
 public String getName() {
        return name;
  }

    public void setName(String name) {
        this.name = name;
  }

    public int getAge() {
        return age;
  }

    public void setAge(int age) {
        this.age = age;
  }

    public Person() {
    }

    private Person(String name) {
        this.name = name;
  System.out.println(name);
  }

    public void fun() {
        System.out.println("fun");
  }

    public void fun(String name,int age) {
        System.out.println("我叫"+name+",今年"+age+"岁");
  }
}

public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            Object o = c.newInstance();
            Method method = c.getMethod("fun", String.class, int.class);
            method.invoke(o, "tengj", 10);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
我叫tengj,今年10岁
```

怎样，是不是感觉很厉害，我们只要知道这个类的路径全称就能玩弄它于鼓掌之间。

有时候我们想获取类中所有成员方法的信息，要怎么办。可以通过以下几步来实现：
1.获取所有方法的数组：

```
Class c = Class.forName("com.tengj.reflect.Person");
Method[] methods = c.getDeclaredMethods(); // 得到该类所有的方法，不包括父类的
或者：
Method[] methods = c.getMethods();// 得到该类所有的public方法，包括父类的
```

2.然后循环这个数组就得到每个方法了：

for (Method method : methods)               
完整代码如下：             
person类跟上面一样，这里以及后面就不贴出来了，只贴关键代码

```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            Method[] methods = c.getDeclaredMethods();
            for(Method m:methods){
                String  methodName= m.getName();
                System.out.println(methodName);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```  

执行结果：

```
getName
setName
setAge
fun
fun
getAge
```

这里如果把c.getDeclaredMethods();改成c.getMethods();执行结果如下，多了很多方法，以为把Object里面的方法也打印出来了，因为Object是所有类的父类：

```
getName
setName
getAge
setAge
fun
fun
wait
wait
wait
equals
toString
hashCode
getClass
notify
notifyAll
```

### 获取成员变量信息

想一想成员变量中都包括什么：成员变量类型+成员变量名
类的成员变量也是一个对象，它是java.lang.reflect.Field的一个对象，所以我们通过java.lang.reflect.Field里面封装的方法来获取这些信息。

单独获取某个成员变量，通过Class类的以下方法实现：

```
public Field getDeclaredField(String name) // 获得该类自身声明的所有变量，不包括其父类的变量
public Field getField(String name) // 获得该类自所有的public成员变量，包括其父类变量
```

参数是成员变量的名字。
例如一个类A有如下成员变量：

private int n;
如果A有一个对象a，那么就可以这样得到其成员变量：

Class c = a.getClass();             
Field field = c.getDeclaredField("n");                 
完整代码如下：     


```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            //获取成员变量
            Field field = c.getDeclaredField("msg"); //因为msg变量是private的，所以不能用getField方法
            Object o = c.newInstance();
            field.setAccessible(true);//设置是否允许访问，因为该变量是private的，所以要手动设置允许访问，如果msg是public的就不需要这行了。
            Object msg = field.get(o);
            System.out.println(msg);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
hello wrold
```

同样，如果想要获取所有成员变量的信息，可以通过以下几步
1.获取所有成员变量的数组：

Field[] fields = c.getDeclaredFields();             
2.遍历变量数组，获得某个成员变量field              

for (Field field : fields)               
完整代码：                

```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            Field[] fields = c.getDeclaredFields();
            for(Field field :fields){
                System.out.println(field.getName());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```          
执行结果：

```
name
age
msg
```

### 获取构造函数

最后再想一想构造函数中都包括什么：构造函数参数
同上，类的成构造函数也是一个对象，它是java.lang.reflect.Constructor的一个对象，所以我们通过java.lang.reflect.Constructor里面封装的方法来获取这些信息。

单独获取某个构造函数,通过Class类的以下方法实现：

```
public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) //  获得该类所有的构造器，不包括其父类的构造器
public Constructor<T> getConstructor(Class<?>... parameterTypes) // 获得该类所以public构造器，包括父类
```

这个参数为构造函数参数类的类类型列表。
例如类A有如下一个构造函数：

```
public A(String a, int b) {
    // code body
}
```

那么就可以通过：

```
Constructor constructor = a.getDeclaredConstructor(String.class, int.class);
```  
来获取这个构造函数。

完整代码：

```
public class ReflectDemo {
    public static void main(String[] args){
        try {
            Class c = Class.forName("com.tengj.reflect.Person");
            //获取构造函数
            Constructor constructor = c.getDeclaredConstructor(String.class);
            constructor.setAccessible(true);//设置是否允许访问，因为该构造器是private的，所以要手动设置允许访问，如果构造器是public的就不需要这行了。
            constructor.newInstance("tengj");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
执行结果：

```
tengj
```  
注意：Class的newInstance方法，只能创建只包含无参数的构造函数的类，如果某类只有带参数的构造函数，那么就要使用另外一种方式：

```
fromClass.getDeclaredConstructor(String.class).newInstance("tengj");
```

获取所有的构造函数，可以通过以下步骤实现：
1.获取该类的所有构造函数，放在一个数组中：

Constructor[] constructors = c.getDeclaredConstructors();
2.遍历构造函数数组，获得某个构造函数constructor:

for (Constructor constructor : constructors)           
完整代码：


```
public class ReflectDemo {
    public static void main(String[] args){
            Constructor[] constructors = c.getDeclaredConstructors();
            for(Constructor constructor:constructors){
                System.out.println(constructor);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

执行结果：

```
public com.tengj.reflect.Person()
public com.tengj.reflect.Person(java.lang.String)
```

### 通过反射了解集合泛型的本质
首先下结论：

Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译到了运行期就无效了。
下面通过一个实例来验证：

```
/**
 * 集合泛型的本质
 * @description
 * @author Trigl
 * @date 2016年4月2日上午2:54:11
 */
public class GenericEssence {
    public static void main(String[] args) {
        List list1 = new ArrayList(); // 没有泛型 
        List<String> list2 = new ArrayList<String>(); // 有泛型


        /*
         * 1.首先观察正常添加元素方式，在编译器检查泛型，
         * 这个时候如果list2添加int类型会报错
         */
        list2.add("hello");
//      list2.add(20); // 报错！list2有泛型限制，只能添加String，添加int报错
        System.out.println("list2的长度是：" + list2.size()); // 此时list2长度为1


        /*
         * 2.然后通过反射添加元素方式，在运行期动态加载类，首先得到list1和list2
         * 的类类型相同，然后再通过方法反射绕过编译器来调用add方法，看能否插入int
         * 型的元素
         */
        Class c1 = list1.getClass();
        Class c2 = list2.getClass();
        System.out.println(c1 == c2); // 结果：true，说明类类型完全相同

        // 验证：我们可以通过方法的反射来给list2添加元素，这样可以绕过编译检查
        try {
            Method m = c2.getMethod("add", Object.class); // 通过方法反射得到add方法
            m.invoke(list2, 20); // 给list2添加一个int型的，上面显示在编译器是会报错的
            System.out.println("list2的长度是：" + list2.size()); // 结果：2，说明list2长度增加了，并没有泛型检查
        } catch (Exception e) {
            e.printStackTrace();
        }

        /*
         * 综上可以看出，在编译器的时候，泛型会限制集合内元素类型保持一致，但是编译器结束进入
         * 运行期以后，泛型就不再起作用了，即使是不同类型的元素也可以插入集合。
         */
    }
}
```

执行结果：

```
list2的长度是：1
true
list2的长度是：2
```

### 总结    
到此，Java反射机制入门的差不多了，我是复习SpringMVC里面IOC/DI的时候，底层原理是通过Java反射来实现的，希望这篇笔记也对你有用。


### 参考

[Java基础与提高干货系列——Java反射机制 - 简书](http://www.jianshu.com/p/1a60d55a94cd)     
[Java 反射基础（上） - 睿铖Bing - 博客频道 - CSDN.NET](http://blog.csdn.net/My_TrueLove/article/details/51298218)   
[Java反射机制深入详解](http://www.cnblogs.com/hxsyl/archive/2013/03/23/2977593.html)   
[Java反射入门](http://blog.csdn.net/trigl/article/details/51042403)   
[Java反射机制](http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html)

### 整理的思维导图

个人整理的Java反射机制的思维导图,导出的图片无法查看备注的一些信息，所以需要源文件的童鞋可以关注我个人主页上的公众号，回复反射机制即可获取源文件

![](http://upload-images.jianshu.io/upload_images/1637925-9ce9f69f8b7c26e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)   



