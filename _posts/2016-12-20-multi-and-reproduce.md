---
layout: post
title: 分段线性变换与直方图修正
date: 2016-12-20
categories: blog
tags: [图像处理]
description: 图像处理
---

**本文主要包括以下内容**

- 分段线性变换
- 两种实用的直方图修正技术:直方图均衡化和直方图规定化
- 本章的典型案例分析
	+ 基于直方图均衡化的图像灰度归一化
	+ 直方图匹配

#### 分段线性变换
分段线性变换有很多种， 包括灰度拉伸、 灰度窗口变换等， 本节仅讲述最为常用的灰度拉伸．      

利用分段线性变换函数来增强图像对比度的方法实际是增强原图各部分的反差，即增强输入图像中感兴趣的灰度区域，相对抑制那些不感兴趣的灰度区域。分段线性函数的主要优势在于它的形式可任意合成，而其缺点是需要更多的用户输入．          

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter32/p1.png)       

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter32/p2.png)   

分段的灰度拉伸可以更加灵活地控制输出灰度直方图的分布，可以有选择的拉伸某段灰
度区间以改善输出图像。如果一幅图像灰度集中在较暗的区域而导致图像偏暗，我们可以用
灰度拉伸功能来扩展（斜率＞1）物体灰度区间以改善图像：同样，如果图像灰度集中在较亮
的区域而导致图像偏亮，也可以用灰度拉伸功能来压缩〈斜率＜1)物体灰度区间以改善图像
质量。  

灰度拉伸是通过控制输出图像中灰度级的展开程度来达到控制对比度的效果。一般情况
下都限制$x_1<x_2,y_1<y_2$，从而保证函数单调递增，以避免造成处理过的图像中灰度级发生颠倒．   

**matlab实现**     

```
function out = imgrayscaling(varargin)
% IMGRAYSCALING     执行灰度拉伸功能
%   语法：
%       out = imgrayscaling(I, [x1,x2], [y1,y2]);
%       out = imgrayscaling(X, map, [x1,x2], [y1,y2]);
%       out = imgrayscaling(RGB, [x1,x2], [y1,y2]);
%   这个函数提供灰度拉伸功能，输入图像应当是灰度图像，但如果提供的不是灰度
%   图像的话，函数会自动将图像转化为灰度形式。x1，x2，y1，y2应当使用双精度
%   类型存储，图像矩阵可以使用任何MATLAB支持的类型存储。

[A, map, x1 , x2, y1, y2] = parse_inputs(varargin{:});

% 计算输入图像A中数据类型对应的取值范围
range = getrangefromclass(A);
range = range(2);

% 如果输入图像不是灰度图，则需要执行转换
if ndims(A)==3,% A矩阵为3维，RGB图像
  A = rgb2gray(A);
elseif ~isempty(map),% MAP变量为非空，索引图像
  A = ind2gray(A,map);
end % 对灰度图像则不需要转换
 
% 读取原始图像的大小并初始化输出图像
[M,N] = size(A);
I = im2double(A);		% 将输入图像转换为双精度类型
out = zeros(M,N);
 
% 主体部分，双级嵌套循环和选择结构
for i=1:M
    for j=1:N
        if I(i,j)<x1
            out(i,j) = y1 * I(i,j) / x1;
        elseif I(i,j)>x2
            out(i,j) = (I(i,j)-x2)*(range-y2)/(range-x2) + y2;
        else
            out(i,j) = (I(i,j)-x1)*(y2-y1)/(x2-x1) + y1;
        end
    end
end

% 将输出图像的格式转化为与输入图像相同
if isa(A, 'uint8') % uint8
    out = im2uint8(out);
elseif isa(A, 'uint16')
    out = im2uint16(out);
% 其它情况，输出双精度类型的图像
end

 % 输出:
if nargout==0 % 如果没有提供参数接受返回值
  imshow(out);
  return;
end
%-----------------------------------------------------------------------------
function [A, map, x1, x2, y1, y2] = parse_inputs(varargin);
% 这就是用来分析输入参数个数和有效性的函数parse_inputs
% A       输入图像，RGB图 (3D), 灰度图 (2D), 或者索引图 (X)
% map     索引图调色板 (:,3)
% [x1,x2] 参数组 1，曲线中两个转折点的横坐标
% [y1,y2] 参数组 2，曲线中两个转折点的纵坐标
% 首先建立一个空的map变量，以免后面调用isempty(map)时出错
map = [];
 
%   IPTCHECKNARGIN(LOW,HIGH,NUM_INPUTS,FUNC_NAME) 检查输入参数的个数是否
%   符合要求，即NUM_INPUTS中包含的输入变量个数是否在LOW和HIGH所指定的范围
%   内。如果不在范围内，则此函数给出一个格式化的错误信息。
iptchecknargin(3,4,nargin,mfilename);
 
%   IPTCHECKINPUT(A,CLASSES,ATTRIBUTES,FUNC_NAME,VAR_NAME, ARG_POS) 检查给定
%   矩阵A中的元素是否属于给定的类型列表。如果存在元素不属于给定的类型，则给出
%   一个格式化的错误信息。
iptcheckinput(varargin{1},...
              {'uint8','uint16','int16','double'}, ...
              {'real', 'nonsparse'},mfilename,'I, X or RGB',1);
 
switch nargin
 case 3 %            可能是imgrayscaling(I, [x1,x2], [y1,y2]) 或 imgrayscaling(RGB, [x1,x2], [y1,y2])
  A = varargin{1};
  x1 = varargin{2}(1);
  x2 = varargin{2}(2);
  y1 = varargin{3}(1);
  y2 = varargin{3}(2);
 case 4
  A = varargin{1};%               imgrayscaling(X, map, [x1,x2], [y1,y2])
  map = varargin{2};
  x1 = varargin{2}(1);
  x2 = varargin{2}(2);
  y1 = varargin{3}(1);
  y2 = varargin{3}(2);
end

% 检测输入参数的有效性
% 检查RGB数组
if (ndims(A)==3) && (size(A,3)~=3)   
    msg = sprintf('%s: 真彩色图像应当使用一个M-N-3维度的数组', ...
                  upper(mfilename));
    eid = sprintf('Images:%s:trueColorRgbImageMustBeMbyNby3',mfilename);
    error(eid,'%s',msg);
end
 
if ~isempty(map) 
% 检查调色板
  if (size(map,2) ~= 3) || ndims(map)>2
    msg1 = sprintf('%s: 输入的调色板应当是一个矩阵', ...
                   upper(mfilename));
    msg2 = '并拥有三列';
    eid = sprintf('Images:%s:inColormapMustBe2Dwith3Cols',mfilename);
    error(eid,'%s %s',msg1,msg2);
    
  elseif (min(map(:))<0) || (max(map(:))>1)
    msg1 = sprintf('%s: 调色板中各个分量的强度 ',upper(mfilename));
    msg2 = '应当在0和1之间';
    eid = sprintf('Images:%s:colormapValsMustBe0to1',mfilename);
    error(eid,'%s %s',msg1,msg2);
  end
end
 
% 将int16类型的矩阵转换成uint16类型
if isa(A,'int16')
  A = int16touint16(A);
end
```

调用    

```
I = imread('coins.png');
J1 = imgrayscaling(I,[0.3,0.75],[0.15,0.85]);
figure,imshow(J1,[]);
J2 = imgrayscaling(I,[0.15,0.85],[0.3,0.7]);
figure,imshow(J2,[]);
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter32/p3.png)  

从图中可以看出，第一组参数让图像灰度直方图上的非零区域扩展，而第二
组参数让图像的灰度直方图非零区域压缩， 这给目标图像带来了截然不同的效果。第一幅图
像中的细节更加清晰， 而第二幅图像更加柔和。  

#### 直方图均衡化     
直方图均衡化又称灰度均衡化．是指通过某种灰度映射使输入图像转换为在每一灰度级 上都有近似相同的像素点数的输出图像（即输出的直方图是均匀的〉。在经过均衡化处理后的图像中，像素将占有尽可能多的灰度级并且分布均匀．因此，这样的图像将具有较高的对比度和较大的动态范围。   

为了便于分析，首先考虑灰度范围为0～1且连续的情况。此时图像的归一化直方图即为概率密度函数（PDF):     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter32/p4.png)  
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter32/p5.png)  

直方图均衡化的数学原理:[直方图均衡化原理及编码实现](http://wenku.baidu.com/view/6c55ecbdc77da26925c5b090)     
这个推导很详细，是我看过最清楚明白的一个。     

**直方图均衡化的作用是图像增强**        
有两个问题比较难懂，一是为什么要选用累积分布函数，二是为什么使用累积分布函数处理后像素值会均匀分布。     
参见：[ 直方图均衡化原理](http://blog.csdn.net/rushkid02/article/details/9178117)

**matlab实现**    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter32/p6.png)  

下面的程序在读入了图像pout.tif后，分别对其进行了增加对比度，减小对比度，线性增 加亮度和线性减小亮度的处理， 得到了原图像的4个灰度变化版本；接着又分别对这4副图 像进行了立方图均衡化处理并显示了它们在处理前、 后的直方图．    

```
I = imread('pout.tif');
I = im2double(I);

I1 = 2*I - 55/255;
subplot(4,4,1);
imshow(I1);
subplot(4,4,2);
imhist(I1);
subplot(4,4,3);
imshow(histeq(I1));
subplot(4,4,4);
imhist(histeq(I1));

I2 = 0.5*I + 55/255;
subplot(4,4,5);
imshow(I2);
subplot(4,4,6);
imhist(I2);
subplot(4,4,7);
imshow(histeq(I2));
subplot(4,4,8);
imhist(histeq(I2));

I3 = I + 55/255;
subplot(4,4,9);
imshow(I3);
subplot(4,4,10);
imhist(I3);
subplot(4,4,11);
imshow(histeq(I3));
subplot(4,4,12);
imhist(histeq(I3));

I4 = I - 55/255;
subplot(4,4,13);
imshow(I4);
subplot(4,4,14);
imhist(I4);
subplot(4,4,15);
imshow(histeq(I4));
subplot(4,4,16);
imhist(histeq(I4));
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter32/p7.png)  

