---
layout: post
title: SVM实现邮件分类
date: 2016-12-16
categories: blog
tags: [机器学习]
description: SVM实现邮件分类
---

首先学习一下svm分类的使用。     

主要有以下步骤：   

- Loading and Visualizing Dataj
- Training Linear SVM    
- Implementing Gaussian Kernel
- Training SVM with RBF Kernel    
- 选择最优的C, sigma参数     
- 画出边界线      

**线性keneral实现**    

```
C = 1;
model = svmTrain(X, y, C, @linearKernel, 1e-3, 20);
visualizeBoundaryLinear(X, y, model);
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/ex1/p1.png)


**高斯keneral实现**       

```
function sim = gaussianKernel(x1, x2, sigma)
x1 = x1(:); x2 = x2(:);
sim = 0;
sim = exp( - (x1-x2)'* (x1-x2) / (2 * sigma *sigma ) );
end


load('ex6data2.mat');

% SVM Parameters
C = 1; sigma = 0.1;

% We set the tolerance and max_passes lower here so that the code will run
% faster. However, in practice, you will want to run the training to
% convergence.
model= svmTrain(X, y, C, @(x1, x2) gaussianKernel(x1, x2, sigma)); 
visualizeBoundary(X, y, model);
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/ex1/p2.png)

**选择合适的参数**    

```
function [C, sigma] = dataset3Params(X, y, Xval, yval)
C = 1;
sigma = 0.3;
C_vec = [0.01 0.03 0.1 0.3 1 3 10 30]';
sigma_vec = [0.01 0.03 0.1 0.3 1 3 10 30]';
error_val = zeros(length(C_vec),length(sigma_vec));
error_train = zeros(length(C_vec),length(sigma_vec));
for i = 1:length(C_vec)
    for j = 1:length(sigma_vec)
      model= svmTrain(X, y, C_vec(i), @(x1, x2) gaussianKernel(x1, x2, sigma_vec(j))); 
      predictions = svmPredict(model, Xval);
      error_val(i,j) = mean(double(predictions ~= yval));
    end
end
% figure
% error_val
% surf(C_vec,sigma_vec,error_val)   % 画出三维图找最低点

[minval,ind] = min(error_val(:));   % 0.03
[I,J] = ind2sub([size(error_val,1) size(error_val,2)],ind);
C = C_vec(I)         %   1
sigma = sigma_vec(J)  %   0.100

% [I,J]=find(error_val ==  min(error_val(:)) );    % 另一种方式找最小元素位子
% C = C_vec(I)         % 1
% sigma = sigma_vec(J)  % 0.100
end

[C, sigma] = dataset3Params(X, y, Xval, yval);

% Train the SVM
model= svmTrain(X, y, C, @(x1, x2) gaussianKernel(x1, x2, sigma));
visualizeBoundary(X, y, model);
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/ex1/p3.png)


