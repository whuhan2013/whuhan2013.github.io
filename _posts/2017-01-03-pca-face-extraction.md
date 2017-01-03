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

**结合主成份脸图像的分析**        
首先，我们可以看到，图中的所有20个主成分脸图像的一个共同点是人脸区域之外的图像背景相对较暗。比较典型的如第1行的第3副图像，其背景几乎为黑色，这是因为 ORL数据集中的人脸图像背景较为均匀一致，在原始d维空间中样本在对应背景的这些维上差异很小，从样本分布云团上看，这些维上的云团的散布最小。因此$e_k$对应于这些维的加权系数很小，在显现出的图像中就是灰度小，从而表现为主成分脸的暗背景。           
继续按照这种思路分析， 拿第1个主成份脸来说，眉毛、鼻子和上嘴唇是图像中灰度相对较高的区域，这说明实验数据集中的200个人脸之间在这些位置存在较大的差异；再比如第1行的第5个主成分脸， 面部区域整体亮度较高，这可能是由数据集人脸之间的肤色差异
导致；其他典型如第2行的第3个以及第3行的第2个主成分脸中的眼睛，第2行第1个以及第4行第2个主成份脸的嘴，这些高亮度区域反映了实验数据集中人脸之间的五官差异。此外，我们还注意到同为五官之一的鼻子似乎井不“ 抢眼”，仅第3行的第1幅图像的鼻梁区
域亮度相对高一些。这说明正面为主的人脸图像中鼻子之间差异不大，这正好和学术界普遍认可的鼻子在正面图像为主的人脸识别中的作用不大的结论一致。         

**降维对分类性能的影响**        
人类能够识别人脸，正是由于不同的人在眼睛、嘴和眉毛等一些重要器官上的差别较大。经线性变换后，原始d维空间中那些差别较大的维在变换至低维空间的过程中被较大的加权而保留；而那些每幅人脸图像都类似（缺乏区分力）的特征，如背景、鼻子和额头等被赋予了较低的权值， 从而在d'维空间中几乎没有得到体现。这样经PCA处理后，在特征向量维数大大降低的同时，原图像中那些差异最大的特征被最大程度的保留（以一种线性组合的形式），而那些相对一致、区分力较差的特征则被丢弃。这就是为什么在很多
情况下降维后，分类的识别率并不会明显下降的原因。PCA降维丢弃某些特征所损失的信息通过在低维空间中更加精确地映射可以得到补偿， 从而可以在低维空间中得到和高维空间中相当的识别率。       

**PCA能够很好工作的前提**         
细心的读者可能会发现，在某些主成分脸图像的人脸边缘处也出现了较高的灰度，这是由数据集图像中人脸姿态和位置的差异造成的， 幸好这种差异不大(10%左右）。实际上在我们的系统中，经过PCA降维后的20维样本矩阵能够很好地用于入脸识别的另一个关键点在
于ORL人脸数据库中的大部分人脸在图像中占据着大致相同的区域，姿态差异度不大，井且眼睛、鼻子、发迹和嘴的位置也大体相同。否则， 200个人脸图像之间的差异就不再是人长相本身的差异， 而是这些人的脸部区域在图像中位置的差异，姿态的差异、以及器官位置的差异。当然，此时PCA可以照常计算，但将降维后的样本矩阵用于人脸识别就不会取得理想的识别率，而可能更适合于姿态分类


**基于主分量的人脸重建**       
下面利用式(10-19)来实现对个体人脸图像的重建。我们提供的的函数approx可以胜任这一工作。其中参数x是需要重建的个体人脸样本， K是重建使用的主分量数目， 输出xApprox为对于原样本向量x的重建（近似）。具体实现如:       

```
function [ xApprox ] = approx( x, k )
% 用 k 个主成分分量来近似（重建）样本 x
%
% 输入：x --- 原特征空间中的样本，被近似的对象
%       k --- 近似（重建）使用的主分量数目
%
% 输出：xApprox --- 样本的近似（重建）

% 读入 PCA 变换矩阵 V 和 平均脸 meanVec
load Mat/PCA.mat

nLen = length(x);

xApprox = meanVec;

for ii = 1:k
    xApprox=xApprox+((x-meanVec)*V(:,ii))*V(:,ii)';
end

load Mat/FaceMat.mat
x = FaceContainer(1,:);
[pcaA V]= fastPCA(FaceContainer,200);
xApprox = approx(x,50);
displayImage(xApprox,112,92);
xApprox = approx(x,100);
displayImage(xApprox,112,92);
xApprox = approx(x,200);
displayImage(xApprox,112,92);
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter10b/p3.png)

当使用200个主分量进行重建时，几乎没有差异。      













