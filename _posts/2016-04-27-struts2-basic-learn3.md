---
layout: post
title: Struts2基础知识(三)
date: 2016-4-27
categories: blog
tags: [java]
description: Struts2基础知识(三)
---   

### 本文主要包括以下内容   

1. OGNL表达式
2. 标签  


### OGNL表达式


OGNL是Object Graphic Navigation Language(对象图导航语言)的缩写，它是一个开源项目。Struts2框架使用OGNL作为默认的表达式语言。

相对EL表达式，它提供了平时我们需要的一些功能，如：           
支持对象方法调用，如xxx.sayHello();            
支持类静态方法调用和值访问，表达式的格式为@[类全名(包括包路径)]@[方法名|值名]，例如：@java.lang.String@format(‘foo %s’,’bar’)或@cn.itcast.Constant@APP_NAME;
操作集合对象                   

Ognl有一个上下文(Context)概念，说白了上下文就是一个MAP结构，它实现了java.utils.Map接口，在Struts2中上下文(Context)的实现为ActionContext，下面是上下文(Context)的结构示意图    


![这里写图片描述](http://img.blog.csdn.net/20160427211545368)


### 小技巧：在页面中使用<s:debug/>查看上下文中的对象

### OGNL表达式语言

- 访问上下文(Context)中的对象需要使用#符号标注命名空间，如#application、#session
另外OGNL会设定一个根对象(root对象)，在struts2中根对象就是ValueStack(值栈)。如果要访问根对象(即ValueStack)中对象的属性，则可以省略#命名空间，直接访问该对象的属性即可。

- 在Struts2中，根对象ValueStack的实现类为OgnlValueStack，该对象不是我们想象的只存放单个值，而是存放一组对象。在OgnlValueStack类里有一个List类型的root变量，就是使用它存放一组对象。

- Context-----OnglValueStack  root变量[action,OgnlUtil,…]

- 在root变量中处于第一位的对象叫栈顶对象。通常我们在OGNL表达式里直接写上属性的名称即可访问root变量里对象的属性，搜索顺序是从栈顶对象开始寻找，如果栈顶对象不存在该属性，就会从第二个对象寻找，如果没有找到就从第三个对象寻找，依次往下访问，直到找到为止。



### 注意：Struts2中，OGNL表达式需要配合Struts标签才可以使用。如：

```
<s:property value=“name”/>
```  


- 由于ValueStack(值栈)是Struts2中OGNL的根对象，如果用户需要访问值栈中的对象，在JSP页面可以直接通过下面的EL表达式访问ValueStack(值栈)中对象的属性：
${foo}//获得值栈中某个对象的foo属性。

- 如果访问其他Context中的对象，由于他们不是根对象，所以在访问时，需要添加#前缀
- application对象：用于访问ServletContext，例如#application.userName或者#application[‘userName’]，相当于调用ServletContext的getAttribute(“username”)。

- session对象：用来访问HttpSession，例如#session.userName或者#session[‘userName’]，相当于调用session.getAttribute(“userName”)。

- request对象：用来访问HttpServletRequest属性(attribute)的Map，例如#request.userName或者#request[‘userName’]，相当于调用request.getAttribute(“userName”)。

- parameters对象：用与访问HTTP的请求参数，例如#parameters.userName或者#parameters[‘userName’]，相当于调用request.getParameter(“username”)。

- attr对象：用于按page->request->session->application顺序访问其属性。


### 采用OGNL表达式创建List/Map集合对象


如果需要一个集合元素的时候(例如List对象或者Map对象)，可以使用OGNL中同集合相关的表达式。使用如下代码直接生成一个List对象：

```
<s:set name=“list” value=“{‘a’,’b’,’c’}”/>
<s:iterator value=“#list”>
<s:property/><br/>
</s:iterator>
```

Set标签用于将某个值放入指定范围。
scope：指定变量被放置的范围，该属性可以接受application、session、request、page或action。如果没有设置该属性，则默认放置在OGNL Context中。
value：赋给变量的值。如果没有设置该属性，则将ValueStack栈顶的值赋给变量。

生成一个Map对象：

```
<s:set name=“foobar” value=“#{‘foo1’:’bar1’,’foo2’:’bar2’}”/>
<s:iterator value=“#foobar”>
<s:property value=“key”/>=<s:property value=“value”/><br/>
</s:iterator>
```

### 采用OGNL表达式判断对象是否存在于集合中

对于集合类型，OGNL表达式可以使用in和not in两个元素符号。其中，in表达式用来判断某个元素是否在指定的集合对象中；not in判断某个元素是否不在指定的集合对象中，如下所示：
in表达式：

```
<s:if test=“’foo’ in {‘foo’,’bar’}”
在
</s:if>
<s:else>
不在
</s:else>
```

not in 表达式：

```
<s:if test=“’foo’ in {‘foo’,’bar’}”
不在
</s:if>
<s:else>
在
</s:else>
```

### OGNL表达式的投影功能

出了in和not in之外，OGNL还允许使用某个规则获得集合对象的子集，常用的有以下3个相关操作符。

?：获得所有符合逻辑的元素
^：获得符合逻辑的第一个元素
$：获得符合逻辑的最后一个元素
例如代码：

```
<s:iterator value=“books.{?#this.price>35}”>
<s:property value=“title”/>-$<s:property value=“price”/><br/>
</s:iterator>
```

在上面代码中，直接在集合后紧跟.{}运算符表明用于取出该集合的子集，{}内的表达式用于获取符合条件的元素，this指的是为了从大集合books筛选数据到小集合，需要对大集合books进行迭代，this代表当前迭代的元素。本例的表达式用于获取集合中价格大于35的书集合。

```
public class BookAction extends ActionSupport{
private List<Book> books;
public String execute(){
books = new ArrayList<Book>();
books.add(new Book(“a”,”spring”,67));
books.add(new Book(“b”,”ejb”,15));
}
}
```


### 常用标签  

### Property标签

```
Property标签用于输出指定值：
<s:set name=“name” value=“kk”/>
<s:property value=“#name”/>
default:可选属性，如果需要输出的属性值为null，则显示该属性指定的值
escape：可选属性，指定是否格式化HTML代码
value：可选属性，指定需要输出的属性值，如果没有指定该属性，则默认输出ValueStack栈顶的值。
id：可选属性，指定该元素的标识。（过时）
```  

### if/elseif/else标签

```
<s:set name=“age” value=“21”/>
<s:if test=“#age==23”>
23
</s:if>
<s:elseif test=“#age==21”>
21
</s:if>
<s:else>
都不等
</s:else>
```

### Iterator标签

```
Iterate标签用于对集合进行迭代，这里的集合包含List、Set和数组。
<s:set name=“list” value=“{‘a’,’b’,’c’}”/>
<s:iterator value=“#list” status=“st”>
<font color=<s:if test=“#st.odd”>red</s:if><s:else>blue</s:else>>
<s:property/></font><br/>
</s:iterator>
Value:可选属性，指定被迭代的集合，如果没有设置该属性，则使用ValueStack栈顶的集合。
id：可选属性，指定该元素的标识。（过时）
status：可选属性，该属性指定迭代时的IterateStatus实例。该实例包含如下几个方法：
int getCount()，返回当前迭代了几个元素。
int getIndex()，返回当前迭代元素的索引。
boolean isEven()，返回当前被迭代元素的索引是否是偶数。
boolean isOdd()，返回当前被迭代元素的索引是否是奇数。
boolean isFirst(),返回当前被迭代元素是否是第一个元素
boolean isLast()，返回当前被迭代元素是否是最后一个元素
```  


### 一个实例  

```
 <s:set var="records" value="{'辟邪剑法','玉女心经','葵花宝典','金瓶梅','摄影艺术指导','道德与法制'}"></s:set>
    
    <table border="1">
    	<tr>
    		<th>序号</th>
    		<th>书名</th>
    	</tr>
    	<s:iterator value="#records" status="vs">
    		<tr bgcolor="<s:property value='#vs.even?"red":"green"'/>">
	    		<td>
	    			<s:property value="#vs.count"/>
	    		</td>
	    		<td>
	    			<s:property/>
	    		</td>
	    	</tr>
    	</s:iterator>
    </table>
```  

### URL标签  

![这里写图片描述](http://img.blog.csdn.net/20160427213220257)

### 实例  

```
<s:url action="a12" var="url"><!-- 还对URL进行了重写 -->
    	<s:param name="username" value="'admin'"></s:param><!-- value的取值当做表达式了 -->
    	<s:param name="age" value="'38'"></s:param>
    </s:url>
    <a href="<s:property value="#url"/>">猛点</a>
    <hr/>
    <s:set value="'addCustomer'" var="addr"></s:set><!-- 存放的动作名称 -->
    <s:url  value="%{#addr}"></s:url><!-- url标签的value中的取值，默认是当做字符串的。 如果想把当做表达式来做，请使用%{}-->
```

### 如果value取值是字符串的话，用单引号('')表示，如果取值是值的话，用%{}


### checkboxlist  

![这里写图片描述](http://img.blog.csdn.net/20160428093301665)

![这里写图片描述](http://img.blog.csdn.net/20160428093321337)

### 实例   

```
 <s:checkboxlist name="hobby" list="{'吃饭','睡觉','学习'}" value="{'学习','吃饭'}"></s:checkboxlist><br/>
         <s:checkboxlist list="hobby1" name="hh" value="hobby2"></s:checkboxlist><br/>
          <s:checkboxlist list="#{'北京':'0','上海':'1','山东':'2'}" name="province" listKey="value" listValue="key" value="{'2'}"></s:checkboxlist>

```   

### radio

![这里写图片描述](http://img.blog.csdn.net/20160428093415493)

### select

![这里写图片描述](http://img.blog.csdn.net/20160428093435806)

### 实例  

```
  <s:radio list="#{'0':'女','1':'男'}" listKey="key" listValue="value"></s:radio>
        <hr/>
        <s:select list="#{'021':'上海','010':'北京','0531':'济南'}" listKey="key" listValue="value" value="'010'"></s:select>
``` 