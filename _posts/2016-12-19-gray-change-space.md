---
layout: post
title: 图像处理中的matlab使用
date: 2016-12-19
categories: blog
tags: [matlab]
description: matlab
---

**图像的矩阵表示**     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/p2.png)   



**类和图像类型**     
虽然使用的是整数坐标， 但 MATLAB 中的像素值（亮度）并未限制为整数。 表 1-1 列出了 MATLAB 和图像处理工具箱为描述像素值而支持的各种类。 表中的前 8 项是数值型的数据类，第 9 项称为字符类， 最后一项称为逻辑类。
uint8 和 logical 类广泛用于图像处理， 当以 TIFF 或 JPEG 图像文件格式读取图像时，会用到这两个类。 这两个类用1个字节表示每个像素。某些科研数据源， 比如医学成像， 要求提供超出 uint8 的动态范围：针对此类数据， 会采用 uint16 和 int16 类。 这两个类为每个矩阵元素使用2 个字节。针对计算灰度的操作， 比如傅立叶变换（见第 3 章）， 使用 double 和single 浮点类。 双精度浮点数每个数组元素使用8 个宇节， 而单精度浮点数使用 4 个字节。尽管工具箱支持 int8 、 uint32 和 int32 类， 但在图像处理中并不常用    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/p3.png)   

#### 算子       
一般我们用字母M和N分别表示矩阵中的行与列。1x N
矩阵被称为行向量， M*1矩阵被称为列向量， 1*1矩阵则被称为标量。    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p1.png)  


#### Matlab 图像类型及其存储方式      
在介绍数字图像的分类时， 曾提及一些主要的图像类型。 本节将介绍这些主 要的图像类型在Matlab中是如何存储和表示的， 主要包括亮度图像、 RGB 图像、 索引图像、二值图像和多帧图像。   

1、亮度图像（Intensity Image)      
亮度图像即灰度图像。 Matlab使用二维矩阵存储亮度图像，矩阵中的每个元素直接表示 一个像素的亮度〈灰度〉信息。 例如， 一个 200 像素× 300 像素的图像被存储为一个 200 行 300 列的矩阵，可以使用 1.1.5小节介绍的选取矩阵元素〈或子块〉的方式来选择图像中的一个像素或一个区域．     

如果矩阵元素的类型是双精度的， 则元素的取值范围是从 0 到 1 ：如果是 8 位无符号整数，则取值范围从0到255。数据0表示黑色，而1（或255）表示最大亮度〈通常为白色〉      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p4.png)

2、RGB 图像（RGBlmage)      
RGB 图像使用3个一组的数据表达每个像素的颜色， 即其中的红色、绿色和蓝色分量。
在Matlab 中， RGB 图像被存储在一个m ×n×3的三维数组中。对于图像中的每个像素， 存
储的三个颜色分量合成像素的最终颜色。例如， RGB 图像I 中位置在11 行40 列的像素的
RGB 值为I( 11,40.1:3 ）。或I( 11,40,:)，该像素的红色分量为I(11,40,1)，蓝色分量为I( 11,40,3 ）。
而I (:,:,1 ）则表示整个的红色分量图像．

RGB 图像同样可以由观精度数组或8 位无符号整数数组存储。图1.6 是一个使用双精度
数组存储ROB 图像的例子。          
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p5.png)

3、索引图像（indexed Image)        
索引图像往往包含两个数组，一个图像数据矩阵（Image Matrix ）和一个颜色索引表( Colormap ）。对应于图像中的每一个像素， 图像数据数组都包含一个指向颜色索引表的索 引值。
颜色索引表是一个m× 3 的双精度型矩阵， 每一行指定一种颜色的 3 个 RGB 分量，即color = [R G B］。 其中 R、G、 B 是实数类型的现精度数， 取值0～ 1。 0 表示全黑， 1 表示最大亮度。 图 1.7 是一个索引图像的实例， 需要注意的是， 图像中的每个像素都用整数表示， 其含义为颜色索引表中对应颜色的索引。  
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p6.png)

图像数据矩阵和颜色索引表的关系取决于图像数据炬阵中存储的数据类型是双精度类
型还是8位无符号整数。     
如果图像数据使用双精度类型存储， 则像索数据1表示颜色索引表中的第一行， 像素数
据2表示颜色索引表中的第二行， 以此类推。如果图像数据使用8位无符号整数存储， 则存
在一个额外的偏移量－1， 像素数据0表示颜色索引表中的第一行， 而1表示索引表中的第二
行， 以此类推。         
8 位方式存储的图像可以支持256 种颜色（或256 级灰度〉。因1.7 中， 数据矩阵使用的
是双精度类型， 所以没有偏移量． 数据5表示颜色表中的第5种颜色。

4、二值图像（ Binary Image)
在二值图像中， 像素的颜色只有两种取值： 黑或白。Matlab 将二值图像存储为一个二维
矩阵， 每个元素的取值只有0和1两种情况，0表示黑色， 而1表示白色。
二值图像可以被看作是一种特殊的只存在黑和白两种颜色的亮度图像， 当然， 也可以将
二值图像看作是颜色索引表中只存在两种颜色〈黑和白〉的索引图像。  

Matlab 中使用uint8 型的逻辑数组存储二值图像， 通过一个逻辑标志表示数据有效范围是0到1，而如果逻辑标志未被置位，则有效范围为0到255。   
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p7.png)

5､多帧图像（ Multlframe Image Array)       
实际应用中， 可能需要处理多幅按时间或视角方式连续排列的图像， 我们把这种图像称
之为多帧图像（所谓“ 帧” 就是影像动画中最小单位的单幅影像，画面〉。例如核磁共振成像数
据或视频片断. Matlab提供了在同一个矩阵中存储多帧图像的方法， 实际上就是在图像矩阵
中增加一个维度来代表时间或视角信息． 例如， 一个拥有5 张连续的400 像素× 300 像素的
RGB图像的多帧连续片断的存储方式是一个400× 300 × 3× 5 的矩阵， 一组同样大小的灰皮
图像则可以使用一个400× 300 × 1× 5 的矩阵来存储．  

如果多帧图像使用索引图像的方式存储，则只有图像数据矩阵被按多帧形式存储， 而颜
色索引表只能公用。因此， 在多帧索引图像中， 所有的索引图像公用一个颜色索引表， 进而
只能使用相同的颜色组合。   
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p8.png)

默认情况下，matlab将绝大多数数据存储为双精度类型〈ω 位浮点数）以保证运算的精
确性． 而对于图像而言， 这种数据类型在图像尺寸较大时可能并不理想。例如， 一张1000
像萦见方的图像拥有一百万个像素， 如果每个像索用64位二进制数表示， 则总共需要大约
8MB的内存空间．        
为了减小图像信息的空间开销， 可以将图像信息存为8 位无符号整型数（uint8）和16
位无符号整型数（ uint16 ）的数组， 这样只需要双精度浮点数八分之一或四分之一的空间。      
在上述3种存储类型中双精度和 uint8 使用最多， uint16 的情况与 uint8 大致类似．  

#### Matlab的图像转换     
有时必须将图像存储格式加以转换才能使用某些图像处理函数。例如，当使用某些Matlab
内置的滤镜时， 需要将索引图像转换为RGB 图像或者灰度图像， Matlab 才会将图像滤镜应
用于图像数据本身， 而不是索引图像中的颜色索引值表〈这将产生无意义的结果〉．     
Matlab 提供了一系列存储格式转换函数，如表1.11 所示．它们的名字都便于记忆，例如，
ind2gray 可以将索引图像转化为灰度图像。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p9.png)

也可以使用一些矩阵操作函数实现某些格式转换。例如．下面的语句可以将一幅灰度图
像转换为RGB图像．     
RGBIMAGE = CAT(3, GRAY, GRAY, GRAY);

**2. 图像数据类型转换**      
Matlab 图像处理工具箱中支持的默认图像数据类型是uint8， 使用imread函数读取的
图像文件一般都为uint8类型。然而， 很多数学函数如sin等并不支持double以外的类
型， 例如， 当试图对uint8 类型直接使用sin函数进行操作时， Matlb会提示如下的错
误信息  

```
sin(D):
H? Undehned functioii <!>i; method ’sin’ for; input a,rguments of type- ’uint8'
```

针对这种情况， 除了使用1.1.4 小节介绍的强制类型转换方法外， 还可利用图像处理工
具箱中的内置图像数据类型转换函数． 内置转换函数的优势在于它们可以帮助处理数据偏移
量和归一化变换， 从而简化了编程工作．      
一些常用的图像类型转换函数如表1.12所示     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p10.png)

#### 图像的输入输出和显示    
matlab可以处理以下的图像文件类型： BMP、HDF、JPEG、PCX、TIFF、XWD、ICO、
GIF、CURo 可以使用imread和imwrite函数对国像文件进行读写操作，使用imfinfo函数来
获得数字图像的相关信息．

**1. imread函数**    
imread函数可以将指定位置的图像文件读入工作区。对于除索引图像以外的情况，其原
型为：      
A =imread(FileNAME, FMT);       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p11.png)

**2. IMWRITE函数**    
imwrite函数用于将指定的图像数据写入文件中，通过指定不同的保存文件扩展名，起到
图像格式转换的作用〈参见例2.4 ）. 其调用格式为：            
imwite(A, FileName，FMT);                
• FILENAME参数指定文件名（不必包含扩展名）．      
• FMT参数指定保存文件所采用的格式．      

存储索引图像时，还需要一并存储颜色索引表，则此时IMWRITE函数的使用方法应为：
imwrite(A, HAP, FILENAME, FMT);      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p12.png)

**3. imfinfo函数**
imfinfo函数可以读取图像文件中的某些属性信息〈比如修改日期、大小、
格式、高度、宽度、色深、颜色空间、存储方式等。其调用格式为：      
imfinfo(FileName,FMT);
参数说明        
• FILENAME 参数指定文件名．      
• FMT 参数是可选参数， 用于指定文件格式．    

**图像的显示**    
一般使用imshow函数来显示图像， 该函数可以创建一个图像对象， 并可以自动设置图
像的诸多属性，从而简化编程操作。这里介绍imshow函数的几种常见调用方式．      

1.imshow函数    
imshow函数用于显示工作区或图像文件中的图像．在显示的同时可控制部分效果〈参见
例12.6）， 常用的调用形式为：    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p13.png)

2.多帧图像的显示       
在显示多帧图像时，可以显示多帧中的一帧，或者将它们显示在同一个窗口内，也可以
将多帧图像转化成电影播放出来． 这3种方式的实现如例1.8所示。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p14.png)

