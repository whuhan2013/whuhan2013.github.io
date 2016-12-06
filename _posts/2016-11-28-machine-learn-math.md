---
layout: post
title: 机器学习数学基础
date: 2016-11-28
categories: blog
tags: [机器学习]
description: 机器学习数学基础
---


**矩阵和向量**     
Rectangular array of numbers    
矩阵的维数即行数×列数       
vector:an n*1 matrix    

**矩阵乘法的性质:**      
矩阵的乘法不满足交换律:A×B≠B×A 矩阵的乘法满足结合律。即:A×(B×C)=(A×B)×C 

**单位矩阵**:在矩阵的乘法中,有一种矩阵起着特殊的作用,如同数的乘法中的 1,我们称
这种矩阵为单位矩阵.它是个方阵,一般用 I 或者 E 表示,本讲义都用 I 代表单位矩阵,从 左上角到右下角的对角线(称为主对角线)上的元素均为 1 以外全都为 0。         
对于单位矩阵,有 AI=IA=A       

**矩阵的逆**    
如矩阵 A 是一个 m×m 矩阵(方阵),如果有逆矩阵,则:    
$AA^{-1}=A^{-1}A=I$     
我们一般在 OCTAVE 或者 MATLAB 中进行计算矩阵的逆矩阵。

**矩阵的转置**      
设 A 为 m×n 阶矩阵(即 m 行 n 列),第 i 行 j 列的元素是 a(i,j),即: A=a(i,j)     
定义 A 的转置为这样一个 n×m 阶矩阵 B,满足 B=a(j,i),即 b (i,j)=a (j,i)(B 的第 i 行第 j 列元素是 A 的第 j 行第 i 列元素),记 $A^T=B$

直观来看,将 A 的所有元素绕着一条从第 1 行第 1 列元素出发的右下方 45 度的射线作 镜面反转,即得到 A 的转置。

矩阵的转置基本性质:     
$(A±B)^T=A^T±B^T$     
$(A×B)^T= B^T×A^T$      
$(A^T)^T=A$     
$(KA)^T=KA^T$        

**链式法则**       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class1/p1.png)   


