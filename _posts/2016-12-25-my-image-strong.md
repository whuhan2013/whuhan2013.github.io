---
layout: post
title: 频率域波图像增强
date: 2016-12-25
categories: blog
tags: [图像处理]
description: 频率域图像增强
---


**本文主要包括以下内容**     

- 频率域图像增强
- 高通滤波器和低通滤波器
- 本章的典型案例分析
	+ 利用频域滤波消除周期噪声


### 频域滤波基础    

**频域滤波与空域滤波的关系**   
傅立叶变换可以将图像从空域变换到频域，而傅立叶反变换则可以将图像的频谱逆变换为空域图像，即人可以直接识别的图像。这样一来，我们可以利用空域图像与频谱之间的对应关系，尝试将空域卷积滤波变换为频域滤波，然后再将频域滤波处理后的图像反变换回空间域，从而达到图像增强的目的。这样做的一个最主要的吸引力在于频域滤波的直观性特点，关于这一点稍后将进行详细的阐述。      

根据著名的卷积定理：两个二维连续函数在空间域中的卷积可由其相应的两个傅立叶变换乘积的反变换而得：反之，在频域中的卷积、可由在空间域中乘积的傅立叶变换而得，即:      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p1.png) 

**频域滤波的基本步骤**    
根据式（6-47）进行频域滤波通常应遵循以下步骤。     
(1）计算原始图像f(x,y） 的DFT， 得到F(u, v).       
(2）将频谱F(u,v）的零频点移动到频谱图的中心位置。     
(3）计算滤波器函数H(u, v）与F(u,v）的乘积G(u, v).            
(4）将频谱G(u, v）的零频点移回到频谱图的左上角位置。     
(5）计算第（4）步计算结果的傅立叶反变换g(x,y） 。      
(6）取g(x,y ）的实部作为最终滤波后的结果图像．     
由上面的叙述易知， 滤波能否取得理想结果的关键取决于频域滤波函数H(u,v)。我们常常称之为滤波器，或滤波器传递函数，因为它在滤波中抑制或滤除了频谱中某些频率的分量，而保留其他的一些频率不受影响。本书中我们只关心值为实数的滤波器，这样滤波过程中H 的每一个实数元素分别乘以F中对应位置的复数元素，从而使F中元素的实部和虚部等比例变化，不会改变F的相位谱，这种滤波器也因此被称为“零相移” 滤波器。这样，最终反变换回空域得到的滤波结果图像 g(x,y）理论上也应当为实函数，然而由于计算舍入误差等原因， 可能会带有非常小的虚部，通常将虚部直接忽略。       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p2.png) 

**matlab实现**    
为方便读者在Matlab中进行频域滤波，我们编写了imfreqfilt函数，其用法同空域滤波时使用的imfilter函数类似，调用时需要提供原始图像和与原图像等大的频域滤波器作为参数，函数的输出为经过滤波处理又反变换回空域之后的图像。    

```
function out = imfreqfilt(I, ff)
% imfreqfilt函数			对灰度图像进行频域滤波
% 参数I				输入的空域图像
% 参数ff				应用的与原图像等大的频域滤镜

if (ndims(I)==3) && (size(I,3)==3)   % RGB图像
    I = rgb2gray(I);
end

if (size(I) ~= size(ff))
    msg1 = sprintf('%s: 滤镜与原图像不等大，检查输入', mfilename);
    msg2 = sprintf('%s: 滤波操作已经取消', mfilename);
    eid = sprintf('Images:%s:ImageSizeNotEqual',mfilename);
    error(eid,'%s %s',msg1,msg2);
end

% 快速傅立叶变换
f = fft2(I);

% 移动原点
s = fftshift(f);

% 应用滤镜及反变换
out = s .* ff; %对应元素相乘实现频域滤波
out = ifftshift(out);
out = ifft2(out);

% 求模值
out = abs(out);

% 归一化以便显示
out = out/max(out(:));
```

#### 频域低通滤波器       
在频谱中，低频主要对应图像在平滑区域的总体灰度级分布，而高频对应图像的细节部 分，如边缘和噪声。因此，图像平滑可以通过衰减图像频谱中的高频部分来实现，这就建立了空间域图像平滑与频域低通滤波之间对应关系。      

**理想低通滤波器及其实现**     
最容易想到的衰减高频成份的方法就是在一个称为“截止频率”的位置“截断”所有的 高频成份，将图像频谱中所有高于这一截止频率的频谱成份设为0，低于截止频率的成为保持不变。能够达到这种效果的滤波器如图6.16所示，我们称之为理想低通滤披器。如果图像的宽度为M，高度为N，那么理想低通频域滤波器可形式化地描述为：          
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p3.png) 

其中Do 表示理想低通滤波器的截止频率，滤波器的频率域原点在频谱图像的中心处，在以截止频率为半径的圆形区域之内的滤镜元素值全部为1， 而该圆之外的滤镜元素值全部为o. 理想低通滤波辑的频率特性在截止频率处十分陡峭，无法用硬件实现，这也是我们称之为理想的原因， 但其软件编程的模拟实现较为简单。          
理想低通滤波器可在一定程度上去除图像噪声， 但由此带来的图像边缘和细节的模糊效应也较为明显，其滤波之后的处理效果比较类似于5.3.1中的平均平滑。实际上，理想低通滤波器是一个与频谱图像同样尺寸的二维矩阵，通过将矩阵中对应较高频率的部分设为0，较低频率的部分〈靠近中心〉设为1，可在与频谱图像相乘后有效去除频谱的高频成份〈由于是矩阵对应元素相乘，频谱高频成份与滤波器中的0相乘〉。其中0与1的交界处即对应滤波器的截止频率。     

**matlab实现**    

```
function out = imidealflpf(I, freq)
% imidealflpf函数			构造理想的频域低通滤波器
% I参数				输入的灰度图像
% freq参数				低通滤波器的截止频率
% 返回值：out – 指定的理想低通滤波器
[M,N] = size(I);
out = ones(M,N);
for i=1:M
    for j=1:N
        if (sqrt(((i-M/2)^2+(j-N/2)^2))>freq)
            out(i,j)=0;
        end
    end
end

I = imread('baby_noise.bmp');

ff = imidealflpf(I,20);
out = imfreqfilt(I,ff);

figure(1);
subplot(2,2,1);
imshow(I);
title('Source');

temp = fft2(I);
temp = fftshift(temp);
temp = log(1+abs(temp));
figure(2);
subplot(2,2,1);
imshow(temp,[]);
title('Source');

figure(1);
subplot(2,2,2);
imshow(out);
title('Ideal LPF,frq = 20');

temp = fft2(out);
temp = fftshift(temp);
temp = log(1+abs(temp));
figure(2);
subplot(2,2,2);
imshow(temp,[]);
title('Ideal LPF,frq = 20');    
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p4.png) 
由图可知，当截止频率非常低时， 只有非常靠近原点的低频成份能够通过， 图像模糊严重：截止频率越高， 通过的频率成份就越多， 图像模糊的程度越小， 所获得的图像也就越接近原图像。 但可以看出， 理想低通滤波器并不能很好地兼顾噪声滤除与细节保留两个方面， 这和空域中采用平均模板时的情形比较类似。 下面将介绍频域的高斯低通滤 波器并比较它与理想低通滤波器的处理效果。

#### 高斯低通滤波器及其实现     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p5.png)  
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p6.png)  

从图6.19可以发现，当σ增大时，H(u）的图像倾向于变宽，而h(x）的图像倾向于变窄和变高。这也体现了频率域和空间域的对应关系。频率域滤波器越窄，滤除的高频成份越多， 图像就越平滑〈模糊〉：而在空间域，对应的滤波器就越宽，相应的卷积模板越平坦，平滑〈模糊）效果就越明显。      
我们在图6.20中分别给出σ取值为3和1时的频域二维高斯滤波器的三维曲面表示，可 以看出频域下的二维高斯滤波器同样具有上述一维情况时的特点．  

matlab实现      

```
function out = imgaussflpf(I, sigma)
% imgaussflpf函数     		构造频域高斯低通滤波器
% I参数				输入的灰度图像
% sigma参数			高斯函数的Sigma参数

[M,N] = size(I);
out = ones(M,N);
for i=1:M
    for j=1:N
        out(i,j) = exp(-((i-M/2)^2+(j-N/2)^2)/2/sigma^2);
    end
end

I = imread('baby_noise.bmp');

ff = imgaussflpf(I,20);
out = imfreqfilt(I,ff);

figure(1);
subplot(2,2,1);
imshow(I);
title('Source');

temp = fft2(I);
temp = fftshift(temp);
temp = log(1+abs(temp));
figure(2);
subplot(2,2,1);
imshow(temp,[]);
title('Source');

figure(1);
subplot(2,2,2);
imshow(out);
title('Gauss LPF,sigma=20');

temp = fft2(out);
temp = fftshift(temp);
temp = log(1+abs(temp));
figure(2);
subplot(2,2,2);
imshow(temp,[]);
title('Gauss LPF,sigma=20');
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p7.png) 

显然， 高斯滤波器的截止频率处不是陡峭的。高斯低通滤波器在Sigma参数取 40 的时候可以较好地处理被高斯噪声污染的图像，而且相比于理想低通滤波器而言， 处理效果上的改进是显而易见的． 高斯低通滤波器在有效抑制噪声的同时， 图像的模糊程度更低， 对边缘带来的混叠程度更小， 从而使高斯低通滤波器在通常情况下获得了比理想低通滤波器更为广泛的应用。

#### 频率域高通滤波器      
图像锐化可以通过衰减图像频谱中的低频成份来实现， 这就建立了空间域图像锐化与频域高通滤波之间对应关系。      

**高斯高通滤波器及其实现**    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p8.png) 

matlab实现     

```
function out = imgaussfhpf(I, sigma)
% imgaussfhpf函数         构造频域高斯高通滤波器
% I参数               输入的灰度图像
% sigma参数           高斯函数的Sigma参数

[M,N] = size(I);
out = ones(M,N);
for i=1:M
    for j=1:N
        out(i,j) = 1 - exp(-((i-M/2)^2+(j-N/2)^2)/2/sigma^2);
    end
end
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p9.png) 
高斯高通滤波器可以较好地提取图像中的边缘信息， Sigma参数取值越小， 边缘提取越
不精确， 会包含更多的非边缘信息： Sigmaa 参数取值越大， 边缘提取越精确， 但可能包含不完整的边缘信息。     

**频域拉普拉斯滤波器及其实现** 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p10.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p11.png) 

matlab实现     

```
function out = imlapf(I)
% imlapff函数         构造频域拉普拉斯滤波器
% I参数               输入的灰度图像

[M,N] = size(I);
out = ones(M,N);
for i=1:M
    for j=1:N
        out(i,j) = -((i-M/2)^2+(j-N/2)^2);
    end
end
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p12.png) 

#### Matlab 综合案例一一利用频域滤波消除周期噪声        
介绍了几种典型的频域滤波器，实现了频域下的低通和高通滤波．它们均可在空域下来用平滑和锐化算子实现。本节将给出一个特别适合在频域中完成的滤波案例， 即利用频域带阻滤波器消除图像中的周期噪声。下面就来看着这个在空域中几乎不可能完成的任务在频域中是如何中实现的。       

**频域带阻滤波器**       
顾名思义， 所谓“ 带阻” 就是阻止频谱中某一频带范围的分量通过， 其他频率成份则不受影响。常见的。带阻滤波器有理想带阻滤波器和高斯带阻滤波器。  

**理想带阻滤波器**      
理想带阻滤波器的表达式为：       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p13.png) 

**高斯带阻滤波器**      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p14.png) 

matlab实现    

添加躁声     

```
O = imread('pout.tif');    
[M,N] = size(O);
I = O;
for i = 1:M
    for j = 1:N
        I(i,j) = I(i,j)+20*sin(20*i)+20*sin(20*j);
    end
end

subplot(1,2,1);
imshow(O);
title('Source');

subplot(1,2,2);
imshow(I);
title('Added Noise');
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p15.png) 

**频谱分析**    
使用高斯带阻滤波器时，首先应对需要处理图像的频谱有一定了解．下面的命令得到了两幅图像的频谱．  

```
i_f = fft2(I);
i_f = fftshift(i_f);
i_f = abs(i_f);
i_f = log(1+i_f);

o_f = fft2(O);
o_f = fftshift(o_f);
o_f = abs(o_f);
o_f = log(1+o_f);

figure(1);
subplot(1,2,1);
imshow(o_f,[]);
title('Source');

subplot(1,2,2);
imshow(i_f,[]);
title('Added Noise');
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter62/p16.png)

