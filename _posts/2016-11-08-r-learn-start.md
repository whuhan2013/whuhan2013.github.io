---
layout: post
title: R语言入门
date: 2016-11-8
categories: blog
tags: [数据分析]
description: R语言入门
---

**相关学习网站**      

- [https://www.kaggle.com/](https://www.kaggle.com/)
- [http://www.rpubs.com/](http://www.rpubs.com/)
- [https://github.com/](https://github.com/)

**R语言常用领域**            

- 探索性数据分析
- 统计推断
- 回归分析      
1.线性模型拟合数据。（预测变量、结果变量），通过预测变量来预测结果变量。       
2.预测：利用建立好的线性模型预测未来情况        
- 机器学习-分类问题       
训练模型＋预测
- 开发数据产品
	+ googlevis API的使用：画地图等      
	+ Manipulate
	+ rChart:使用R制作交互式javascript可视化产品         
	+ Shiny:制作嵌入网页的交互式R程序的平台:http://www.shinyapps.io/
	+ Slidify:制作和发布基于R的报告(PPT)


**安装**        
R与RStudio的安装       

**包的安装**       
下载r-package：cran、bioconductor、github      

- 下载： install.packges()    
- 加载：library()  

#### R语言的数据结构          

**对象的5中基本类型**  

- 字符character
- 数值numeric：real numbers
- 整型integer
- 复数complex：1+2i
- 逻辑logical：true/false        

1.变量的赋值方式采用 <- 不要用 =          
2.默认变量赋值类型为numeric数值型（包括小数和整数），如果欲强调整数，可在数据后加大写L。例如：x <- 5L       
3.R中变量大小写敏感。查看变量类型的函数class(x)。查看变量值的方法（直接输入变量名称后回车）      
4.字符串变量加双引。逻辑变量TRUE/FALSE全部大写           
5.复数变量。例如：x <- 1+2i          

**向量**

1.#是注释开始的标记      
2.vector向量：支持自动扩容的数组        
3.y <- vector("character",10)  ---  初始化长度为10的字符串数组       
4.x <- 1:4  ---  x为1到4，含4个元素的数组        
5.x2 <- c(1, 2, 3, 4)  ---  用列表方式初始化数组      
6.x3 <- c(TRUE, 10, "a")  ---  数组元素将被强制转换为同一类型       
7.x4 <- c("a", "b", "c")       
as.numeric(x4)      
as.logical(x4)         
as.character(x4)  ---  几种常用的强制转换方式（可能报错）        
8.names(x1) <- c("a", "b", "c", "d")  ---  为x1数组每个角标添加引用       

**矩阵与数组**       
1.矩阵定义：向量+维度属性       
2.x <- matrix(nrow=3 ,ncol=2)#建立3行2列空矩阵       
3.x <- matrix(1:6, nrow=3, ncol=2)#先列后行的方式初始化并建立3行2列矩阵     
4.dim(x)#查看矩阵行列数      
5.attributes(x)#查看矩阵属性及行列数（例如：维度属性：3行2列）       
6.y <- 1:6       
dim(y) <- c(2,3)#先建立向量，然后附加维度属性，使之成为矩阵。      

y2 <- matrix(1:6, nrow=2, ncol=3)#建立一个和y相同的矩阵         

rbind(y,y2)#将矩阵按行合并（行数增加，列数不变）     
cbind(y,y2)#将矩阵按列合并（列数增加，行数不变）      

数组                                      
数组与矩阵相识，但数组可以是多维，矩阵只能是二维            
创建数组 x<-array(1:24,dim=c(4,6))#array(内容,维度)       

**列表**       
列表（list）：         
1.列表与向量的差别：列表可以同时包含不同类型的对象
2.
l <- list("a", 2, 10L, 3+4i, TRUE)#建立列表          
l2 <- list(a=1 ,b=2 ,c=3)#为列表元素命名        
l3 <- list(c(1,2,3),c(4,5,6,7))#列表元素可以是向量，创建元素个数大于1        

x <- matrix(1:6, nrow=2, ncol=3)                 
dimnames(x) <- list(c("a","b"),c("c","d","e"))#为矩阵维度（行、列）命名        

**因子（factor）**      

1.factor因子：因子=整数向量+标签           
2.
x <- factor(c("female","female","male","male","female"))

#设置基线水平           
x <- factor(c("female","female","male","male","female"),levels=c("male","female"))#levels是因子的属性  

3.table(x)#查看因子分类统计后的结果       
4.unclass(x)#去掉标签，将因子变为整数向量（因子=整数变量+标签）        
class(unclass(x))

**缺失值（missing value）**   

- NA/NaN #NaN属于NA，NA不属于NaN（一般表示数字缺失值），前者包含后者，后者只表示数字的缺失。缺失值在数据预处理中很重要
- NA有类型属性:integer NA,character NA等
- is.na()/is.nan()#判断向量中是否有缺失值

**数据框（data frame）**     

数据框在数据分析中经常被使用，用来存储表格数据（tabular data）;
视为各元素长度相同的列表      
  *每个元素代表一列数据      
  *每个元素的长度代表行数 
  *元素类型可以不同            
1.data fram数据框：同一列数据类型相同，且每一列长度相等       
2.df <- data.frame(id=c(1,2,3),name=c("a","b","c"),gender=c("TRUE","TRUE","FALSE"）     
3.nrow(df)#查看数据框行数      
  ncol(df)#查看数据框列数         
4.data.matrix(df2)#数据转矩阵，元素是同种类型的          

**日期和时间**   

#Date          
1.
x <- Date()#字符型        
x2 <- Sys.Date()#Date型                
x3 <- as.Date("2016-01-01")#创建Date型       

2.       
weekdays(x3)#星期几      
months(x3)#月份       
quarters(x3)#季度        

3.
julian(x3)#参数日期与1970-1-1相差几天     

x3-x2#天数差含字符说明      
as.numeric(x3-x2)#转换为数值      

#Time            
1.time创建与转换         
CT类型与lt类型，前者是整数，后者以列表存在        

x <- Sys.time()#POSIXct                
p <- as.POSIXlt(x)#POSIXlt        
as.POSIXct(p)#POSIXct             

2.names(unclass(p))#p去掉属性，保留内容，查看POSIXl$t的属性         
p$sec#获取p中指定属性的数据       

3.x1 <- "一月 1, 2015 01:01"       
strptime(x1, "%B %d, %Y %H:%M")#字符串形式的时间格式化函数            

![](http://img.mukewang.com/5815cf7a0001d3bb12800720.jpg)

#### 数据操作       

**构建子集**      
![](http://img.mukewang.com/5800cc300001feb812800720.jpg)      

```
x <- 1:10
x[1]#角标从1开始
x[1:5]#1到5的值
x[x>5]#大于5的值
x>5#返回的是逻辑向量，即每一个元素与5比较的逻辑值组成的向量
x[x>5 & x<7]#与
x[x<3 | x>7]#或
y < 1:4
names(y) <- c("a","b","c","d")
y["b"]#对于命名的向量，可以用列名代替角标
```

**矩阵的子集**       

```
x <- matrix(1:6, nrow=2, ncol=3)
x[1,2]#返回指定坐标向量
x[1,]#返回指定行的向量
x[,2]#返回指定列的向量
x[1,c(1,3)]#返回指定行中特定列的向量

x[1,2]#返回向量
x[1,2,drop=FALSE]#返回矩阵,而不是向量
```








