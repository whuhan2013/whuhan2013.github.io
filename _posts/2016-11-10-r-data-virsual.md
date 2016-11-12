---
layout: post
title: R语言之数据可视化
date: 2016-11-10
categories: blog
tags: [数据分析]
description: R语言之数据可视化
---

数据科学家需要具备的知识     
![](http://img.mukewang.com/57aef95700011c1312800720.jpg)      
![](http://img.mukewang.com/57aefac50001ea4612800720.jpg)

完整的数据分析流程      
![](http://img.mukewang.com/57d0a55a000115c012800720.jpg)    

观测，变量和数据矩阵      
![](http://img.mukewang.com/575193810001e09112800720.jpg)      

变量的类型      
数值型+分类型      
![](http://img.mukewang.com/5816e7540001636412800720.jpg)     

变量之间的关系            
变量类型不同，分析方法也不同    
![](http://img.mukewang.com/5816e79c00010c4712800720.jpg)      

**数值变量的特征与可视化**         
集中趋势量
![](http://img.mukewang.com/5816e8390001506512800720.jpg)
分散趋势量
![](http://img.mukewang.com/57af010100016d3312800720.jpg)    
稳健统计量：受到均值的影响是否大
![](http://img.mukewang.com/57af01800001896b12800720.jpg)       
可视化一个变量：箱图 四分位点 以及  极值的定义
![](http://img.mukewang.com/57af03250001f27912800720.jpg)       


**分类变量的特征与可视化**             

![](http://img.mukewang.com/5816edef00018b4512800720.jpg)       

两个分类变量的关系    
列联表 +相对频率表     
![](http://img.mukewang.com/5816eed100012c8a12800720.jpg)     

两个分类变量的关系（可视化）：分段条形图，相对频率分段条形图     
![](http://img.mukewang.com/57af07040001628212800720.jpg)      

![](http://img.mukewang.com/57af07730001bfca12800720.jpg)   

一个分类变量、一个数值变量的关系：并排箱图      
![](http://img.mukewang.com/57cfb02000012bd912800720.jpg)

**小结**    
![](http://img.mukewang.com/5816f08e0001417012800720.jpg)      

#### 三大绘图系统  

1. 基本绘图系统      
2. Lattice绘图系统    
3. ggplot2绘图系统     

![](http://img.mukewang.com/57b024980001a82f12800720.jpg)       
![](http://img.mukewang.com/57b024db000168dd12800720.jpg)
![](http://img.mukewang.com/57b0251f00010a1a12800720.jpg)

**基本绘图系统**

- 绘图函数 -> graphics包
- hist/ boxplot/ points/lines/text/title/axis
- 柱状图/箱图/点/线/文字/命名/坐标轴

**plot**    
xlab/ylab/lwd/lty/pch/col   x标签/y标签/线宽/线型/点型/颜色      
全局参数 : bg/mar/las/mfrow/mfcol   背景颜色/边距/横竖排版/分行列按行/列填充      

![](http://img.mukewang.com/58033bd90001630112800720.jpg)    

**实例**       

```
hist(airquality$Wind,xlab = "Wind")
boxplot(airquality$Wind,xlab="Wind",ylab="speed(mph)")
boxplot(Wind~Month,airquality,xlab="Month",ylab="speed(mph)")
plot(airquality$Wind,airquality$Temp)
with(airquality,plot(Wind,Temp))
title(main="Wind and Temp in NYC")

with(airquality,plot(Wind,Temp,
                     main="Wind and Temp in NYC",
                     type = "n"))
with(subset(airquality,Month==9),
     points(Wind,Temp,col = "red"))
with(subset(airquality,Month==5),
     points(Wind,Temp,col = "blue"))
with(subset(airquality,Month==8),
     points(Wind,Temp,col = "black"))

with(subset(airquality,Month %in% c(6,7,8)),
     points(Wind,Temp,col = "black"))
fit <- lm(Temp ~ Wind,airquality)
abline(fit,lwd=2)

#添加图例
legend("topright", pch  = 1, cex = 1,
       col = c("red", "blue", "black"),
       legend = c("sep", "May", "Other"))
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/R/p1.png)

**全局参数**        

```
par("bg")#背景颜色
par("col")#颜色
par("mar")#(bottom,left,right, right)
par("mfrow")
par("mfcol")

par(mfrow = c(1,2))
hist(airquality$Temp)
hist(airquality$Wind)
par(mfrow = c(1,1))
boxplot(airquality$Temp)
```


**Lattice绘图系统**        

![](http://img.mukewang.com/5817faf50001f07712800720.jpg)
![](http://img.mukewang.com/5817fb660001158112800720.jpg)  


**实例**            

```
library(lattice)
xyplot(Temp ~ Ozone, data = airquality)
airquality$Month <- factor(airquality$Month)
xyplot(Temp ~ Ozone | Month, data = airquality,
       layout = c(5,1))

q <- xyplot(Temp ~ Wind, data = airquality)
print(q)

set.seed(1)
x <- rnorm(100)
f <- rep(0:1, each=50)
y <- x + f - f * x + rnorm(100, sd=0.5)#使用随机数时,切记使用种子,保证后期检查,纠错方便.
f <- factor(f, labels = c("Group1", "Group2"))
xyplot(y ~ x | f, layout = c(2, 1))
xyplot(y ~ x | f, panel = function(x,y){
  panel.xyplot(x,y)
  panel.abline(v = mean(x), h = mean(y), lty = 2)
  panel.lmline(x,y, col = "red")
})
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/R/p2.png)

#### ggplot2 绘图系统               

![](http://img.mukewang.com/581709a80001a48412800720.jpg)
![](http://img.mukewang.com/581709fb00014bc912800720.jpg)         

实例      

```
library(ggplot2)
airquality$Month <- factor(airquality$Month)
qplot(Wind, Temp, data = airquality, col = Month, shape = Month, size = Month,
      xlab = "Wind(mph)",
      ylab = "Temperature",
      main = "Wind vs.Temperature"
)
qplot(Wind, Temp, data = airquality,
      geom = c("point", "smooth"))

qplot(Wind, Temp, data = airquality,
      facets = Month~.)
qplot(Wind, Temp, data = airquality,
      facets = .~Month)

qplot(Wind, data = airquality, facets = .~Month)

qplot(Wind, data = airquality, fill = Month)

qplot(Wind, data = airquality, geom = "dotplot")
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/R/p3.png)

**ggplot函数的使用**     

```
library(ggplot2)
ggplot(airquality, aes(Wind, Temp))+
  geom_point(aes(color = factor(Month), group = 1,alpha = 0.4, size = 5))+
  geom_smooth(method = "lm", se = F, aes(group = 1))#前一个group只输出群体拟合,后一个控制再做一条群体拟合


ggplot(airquality, aes(Wind, Temp)) + 
  geom_point()+
  geom_smooth(method = "lm", se = F, aes(group = 1))

library(RColorBrewer)
myColors<-c(brewer.pal(5,"Dark2"),"black")

ggplot(airquality, aes(Wind, Temp, col = factor(Month))) + 
  geom_point()+
  geom_smooth(method = "lm", se = F, aes(group = 1))+ 
  scale_color_manual("Month", values = myColors)+
  facet_grid(.~Month)+
  theme_classic()
```         

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/R/p1.jpg)

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/R/p5.png)

**R语言绘图之颜色**        

![](http://img.mukewang.com/57b18f2200011cbf12800720.jpg)      

![](http://img.mukewang.com/57b18f5f00014a8a12800720.jpg)    

```

library(RColorBrewer)

pal<-colorRamp(c("red","blue"))
pal(0)
pal(1)
pal(0.5)
pal(seq(0,1,len=10))



pal<-colorRampPalette(c("red","blue"))
pal(0)
pal(1)
pal(0.5)
pal(10)

brewer.pal.info

cols<-brewer.pal(3,"Greens")
cols
pal<-colorRampPalette(cols)
pal
image(volcano,col = pal(20))

display.brewer.pal(3,"Greens")
```

**图形设备**              
![](http://img.mukewang.com/57b26c81000192de12800720.jpg)           
![](http://img.mukewang.com/57b26fec0001cd0b12800720.jpg) 

