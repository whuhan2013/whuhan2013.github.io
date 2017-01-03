---
layout: post
title: 基于PCA的人脸特征抽取
date: 2017-1-3
categories: blog
tags: [图像处理]
description: 基于PCA的人脸特征抽取
---

我们将应用PCA技术来抽取人脸特征。一幅人脸照片往往由比较多的像素构成，如果以每个像素作为1维特征，将得到一个维数非常高的特征向量， 计算将十分困难；而且这些像素之间通常具有相关性。这样，利用PCA技术在降低维数的同时在一定程度上去除原始特征各维之间的相关性自然成为了一个比较理想的方案。      

**数据集简介**       
本案例采用的数据集来自著名的ORL人脸库。首先对该人脸库做一个简单的介绍：  

- ORL数据库共有400幅人脸图像(40人， 每人1O幅， 大小为112像素x92像素)          
- 这个数据库比较规范， 大多数图像的光照方向和强度都差不多．        
- 但有少许表情、姿势、伸缩的变化， 眼睛对得不是很准， 尺度差异在10%左右。            
- 并不是每个人都有所有的这些变化的图像，即有些人姿势变化多一点，有些人表情变化多一点， 有些还戴有眼镜， 但这些变化都不大   

正是基于ORL人脸库图像在光照， 以及关键点如眼睛、嘴巴的位置等方面比较统一的特点， 实验可以在该图片集上直接展开， 而不是必须要进行归一化和校准等工作。     

**生成样本矩阵**      
首先要做的是将这 200 幅人脸图像转换为向量形式，进而组成祥本矩阵。函数 ReadFaces()用于实现

```
function [imgRow,imgCol,FaceContainer,faceLabel]=ReadFaces(nFacesPerPerson, nPerson, bTest)
% 读入ORL人脸库的指定数目的人脸前前五张(训练)
%
% 输入：nFacesPerPerson --- 每个人需要读入的样本数，默认值为 5
%       nPerson --- 需要读入的人数，默认为全部 40 个人
%       bTest --- bool型的参数。默认为0，表示读入训练样本（前5张）；如果为1，表示读入测试样本（后5张）
%
% 输出：FaceContainer --- 向量化人脸容器，nPerson * 10304 的 2 维矩阵，每行对应一个人脸向量

if nargin==0 %default value
    nFacesPerPerson=5;%前5张用于训练
    nPerson=40;%要读入的人数（每人共10张，前5张用于训练）
    bTest = 0;
elseif nargin < 3
    bTest = 0;
end

img=imread('Data/ORL/S1/1.pgm');%为计算尺寸先读入一张
[imgRow,imgCol]=size(img);


FaceContainer = zeros(nFacesPerPerson*nPerson, imgRow*imgCol);
faceLabel = zeros(nFacesPerPerson*nPerson, 1);

% 读入训练数据
for i=1:nPerson
    i1=mod(i,10); % 个位
    i0=char(i/10);
    strPath='Data/ORL/S';
    if( i0~=0 )
        strPath=strcat(strPath,'0'+i0);
    end
    strPath=strcat(strPath,'0'+i1);
    strPath=strcat(strPath,'/');
    tempStrPath=strPath;
    for j=1:nFacesPerPerson
        strPath=tempStrPath;
        
        if bTest == 0 % 读入训练数据
            strPath = strcat(strPath, '0'+j);
        else
            strPath = strcat(strPath, num2str(5+j));
        end
        
        strPath=strcat(strPath,'.pgm');
        img=imread(strPath);
       
        %把读入的图像按列存储为行向量放入向量化人脸容器faceContainer的对应行中
        FaceContainer((i-1)*nFacesPerPerson+j, :) = img(:)';
        faceLabel((i-1)*nFacesPerPerson+j) = i;
    end % j
end % i

% 保存人脸样本矩阵
save('Mat/FaceMat.mat', 'FaceContainer')
```

**主成分分析**     
经过上面的处理，矩阵FaceContainter 每一行就成了一个代表某个人脸样本的特征向量。通过主成份分析的方法可将这些10304维的样本特征向量降至20维。这样数据集中每个人脸样本都可以由一个20维的特征向量来表示， 以作为后续分类所采用的特征。   

我们将对样本矩阵FaceContainer进行主成份分析的整个过程封装在下面的main函数中，其参数k 是主分量的数目， 即降维至K 维。该函数首先调用ReadFaces函数得到了人脸样本矩阵FaceContainer,而后利用10.3.4小节中的fastPCA算法计算出样本矩阵的低维表示LowDimFaces 和主成分分量矩阵W, 并将LowDimFaces 保存至Mat 目录下的LowDimFaces.m出文件中。 

```
function main(k)
% ORL 人脸数据集的主成分分析
%
% 输入：k --- 降至 k 维

% 定义图像高、宽的全局变量 imgRow 和 imgCol，它们在 ReadFaces 中被赋值
global imgRow;
global imgCol;

% 读入每个人的前5副图像
nPerson=40;
nFacesPerPerson = 5;
display('读入人脸数据...');
[imgRow,imgCol,FaceContainer,faceLabel]=ReadFaces(nFacesPerPerson,nPerson);
display('..............................');


nFaces=size(FaceContainer,1);%样本（人脸）数目
display('PCA降维...');
% LowDimFaces是200*20的矩阵, 每一行代表一张主成分脸(共40人，每人5张)，每个脸20个维特征
% W是分离变换矩阵, 10304*20 的矩阵
[LowDimFaces, W] = fastPCA(FaceContainer, 20); % 主成分分析PCA
visualize_pc(W);%显示主成分脸
save('Mat/LowDimFaces.mat', 'LowDimFaces');
display('计算结束。');
```
上述命令运行后会在Mat目录下生成LowDimFaces.mat文件，其中的200X20维矩阵LowDimFaces是经过PCA降维后，原样本矩阵FaceContainer的低维表示。200个人脸样本所对应的每一个特征向量由原来的10304维变成了20维，这就将后续的分类问题变成一个在 20维空间中的划分问题，过程大大简化。       

**主成分脸可视化分析**       
fastPCA函数的另一个输出为主分量阵w. 它是一个10304X20的矩阵，每列是一个10304维的主分量（样本协方差矩阵的本征向量），在人脸分析中，习惯称之为主成份脸.事实上我们可以将这些列向量以112X92的分辨率来显示，该工作由函数visualizepc完成，实现如下：       

```
function visualize_pc(E)
% 显示主成分分量（主成分脸，即变换空间中的基向量）
%
% 输入：E --- 矩阵，每一列是一个主成分分量


[size1 size2] = size(E);
global imgRow;
global imgCol;
row = imgRow;
col = imgCol;

if size2 ~= 20
   error('只用于显示 20 个主成分');
end;

figure
img = zeros(row, col);
for ii = 1:20
    img(:) = E(:, ii);
    subplot(4, 5, ii);
    imshow(img, []);
end
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter10b/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter10b/p2.png)











