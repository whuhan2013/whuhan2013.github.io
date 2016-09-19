---
layout: post
title: 多张图片拼接与层叠
date: 2016-9-19
categories: blog
tags: [图像处理]
description: 图像处理
---

本节实验将学习和实践下列知识点：

- Python基本语法
- PIL第三方库的使用
- Numexpr库的使用
- 图像处理相关知识


**安装第三方库**

```
$ sudo apt-get install python-imaging
$ sudo apt-get install python-numpy
$ sudo apt-get install python-numexpr
```


**相关知识简介**


照片模式

![](https://dn-anything-about-doc.qbox.me/userid49391labid948time1430314494671?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


我们这里将用到RGB与RGBA；这两者的区别是RGBA有个透明通道，常见的照片格式中.png属于RGBA，而.jpg格式属于RGB模式。

RGB

RGB即是代表红、绿、蓝三个通道的颜色，这个标准几乎包括了人类视力所能感知的所有颜色，是目前运用最广的颜色系统之一

numpy

Numpy是Pyhon很强大的开源数字扩展库，可以用来处理大型矩阵等等。

PIL

PIL是python与图片处理相关的第三方扩展库，可以完成图片的粘贴，拷贝，添加文字，层叠，合并，图片加强，操作像素点等等，非常强大。


**两张照片层叠的两种方式**

- 方式一：两张照片同一点的像素按照一定比例叠加。假设两张照片同一点的像素分别为A、B，则层叠之后该点像素点（alpha取值在0和1之间）为： A*alpha+B*(1-alpha)
- 方式二：正片叠底。结果色 = 混合色 * 基色 / 255，PS中也采用这种方式； 正片叠底的特点： 明度变化：混合色不会大于255，故结果色一定小于1，混合模式之后必定比基色暗。0为黑色，若混合两色中有黑色，混合之后必定是黑色。若有白色，则混合色为另外一色的原色。故正片叠底可以改变非黑即白，处于灰度区间的明度，变黑。可以采用操作像素点，提高像素点的亮度。



**详细代码**  

常量设置

现在开始写代码，首先便是导入相关文件，如下图所示:

```
from __future__ import division

import PIL

import Image

import numpy

import os

import random

import time

import ImageFont, ImageDraw



STAG = time.time()



# W_num: 一行放多少张照片

# H_num: 一列放多少张照片

# W_size: 照片宽为多少

# H_size: 照片高为多少

# root: 脚本的根目录

root=""

W_num =15

H_num = 15

W_size = 640

H_size = 360

```


- 这里，我们导入PIL，numexpr，Image模块等。
- 同时设置了几个变量，注释详细介绍了每个常量的意思。

获取所有照片信息

```

# name: getAllPhotos
# todo: 获得所有照片的路径
def getAllPhotos():
    STA = time.time()
    root = os.getcwd() + "/"
    src = root+"/photos/"
    for i in os.listdir(src):
	    if os.path.splitext(src+i)[-1] == ".jpg" or os.path.splitext(src+i)[-1] == ".png":
		    aval.append(src+i)
    print "getAllPhotos Func Time %s"%(time.time()-STA)
```


**伸缩图片**

```

# name: transfer
# todo: 将照片转为一样的大小
def transfer(img_path, dst_width,dst_height):

    STA = time.time()
    im = Image.open(img_path)
    if im.mode != "RGBA":
        im = im.convert("RGBA")
    s_w,s_h = im.size
    if s_w < s_h:
        im = im.rotate(90)

    #if dst_width*0.1/s_w > dst_height*0.1/s_h:
    #    ratio = dst_width*0.1/s_w
    #else:
    #    ratio = dst_height*0.1/s_h
    resized_img = im.resize((dst_width, dst_height), Image.ANTIALIAS)
    resized_img = resized_img.crop((0,0,dst_width,dst_height))
    print "transfer Func Time %s"%(time.time()-STA)

    return resized_img
```

- 由于每张照片的大小是不一样的，这里主要是将照片转为一样的大小。
- 由于每张照片的模式既有RGB，也有RGBA，我们直接利用PIL中的convert方法，将所有照片的模式都转为RGBA。然后在调用resize方法，将照片放缩到一样的大小。
- 这里要着重介绍一下numpy.array方法，numpy.array可以创建一个数组，numpy.array(resized_img)[:dst_height, :dst_width]是讲resized_img转换为了一个(dst_height, dst_width)的两维数组，每个数组元素是有四个常数值的列表。打印出来如下图所示：


![](https://dn-anything-about-doc.qbox.me/userid49391labid948time1430316000419?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


**数百张照片拼接，并且将拼接后的照片与另外一张照片进行层叠（past3.py）**

```

# name: createNevImg
# todo: 创造一张新的图片，并保存
def createNevImg():
    iW_size = W_num * W_size
    iH_size = H_num * H_size
    I = numpy.array(transfer(root+"lyf.jpg", iW_size, iH_size))
    I = numexpr.evaluate("""I*(1-alpha)""")

    for i in range(W_num):
	    for j in range(H_num):
		    SH = I[(j*H_size):((j+1)*H_size), (i*W_size):((i+1)*W_size)]
		    STA = time.time()
		    DA = transfer(random.choice(aval), W_size, H_size)
		    print "Cal Func Time %s"%(time.time()-STA)
		    res  = numexpr.evaluate("""SH+DA*alpha""")
		    I[(j*H_size):((j+1)*H_size), (i*W_size):((i+1)*W_size)] = res
```


- 这部分是本项目的核心，着重介绍一下这里。
- root+'lyf.jpg'是要与拼接之后的照片进行层叠的照片；并利用numpy.array的方法将其转换成矩阵I; 注意：I的大小为(W_num*W_size, H_num*H_size)，是第7步伸缩照片生成矩阵的W_num*H_num倍
- 第三行代码是调用numexpr.evaluate方法，将矩阵中每个元素都乘以(1-alpha)。
- 接下来的双重遍历便是具体拼接的实现。
- 由于矩阵I的规模是第六步伸缩照片生成矩阵的W_num*H_num倍，所以我们从左向右，从上向下依次取(W_size, H_size)大小的矩阵SH
- DA是调用第7步方法生成的矩阵
- 然后计算SH+DA*alpha，并将结果放回SH在I矩阵中位置。这里是讲两张照片中相同一点的像素分别乘以(1-alpha)、alpha，然后相加，如此两个照片便层叠在一块。alpha的取值可以自己设置，这里设置的是0.5。不难看出，这里选择的层叠的方法是方式一，将两张照片同一点的像素按照一定比例想加，这里选择的是alpha=0.5。
- 调用fromarray方法将矩阵I转为图片对象，并保存为createNevImage_0.5_past3.jpg，照片如下图所示。


![](https://dn-anything-about-doc.qbox.me/userid49391labid948time1430726023874?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


**数百张照片拼接，并且将拼接之后的照片与另外一张照片层叠（past.py）**

```

# name: createNevImg
# todo: 创建一张新的照片并保存
def createNevImg():
    STAA = time.time()
    iW_size = W_num * W_size
    iH_size = H_num * H_size
    print root
    I = numpy.array(transfer(root+"lyf.jpg", iW_size, iH_size)) * 1.0

    for i in range(W_num):
        for j in range(H_num):
            s = random.choice(aval)
            res = I[ j*H_size:(j+1)*H_size, i*W_size:(i+1)*W_size] * numpy.array(transfer(s, W_size, H_size))/255
            I[ j*H_size:(j+1)*H_size, i*W_size:(i+1)*W_size] = res

    img = Image.fromarray(I.astype(numpy.uint8))
    img = img.point(lambda i : i * 1.5)
    img.save("createNevImg_past.jpg")
    print "createNevImg Func time %s"%(time.time()-STAA)

```

我们看到这里代码与上面的代码是很相似的，只是I矩阵的计算方法是不同的。这里要简单介绍一下I矩阵的计算方法。

- 这里采用是两照片层叠的方式二正片叠底，利用公式： 结果色 = 混合色 * 基色 / 255，至此代码就很明了了
- 利用PIL库以对每个像素点进行操作。 img.point(lambda i : i * 1.5) 将每个像素点的亮度(不知道有没有更专业的词)增大50%
- 保存照片为crateNevImg_past.jpg

![](https://dn-anything-about-doc.qbox.me/userid49391labid948time1430725736547?watermark/1/image/aHR0cDovL3N5bC1zdGF0aWMucWluaXVkbi5jb20vaW1nL3dhdGVybWFyay5wbmc=/dissolve/60/gravity/SouthEast/dx/0/dy/10)


**旋转照片**

```

# name: newRotateImage
# todo: 将createnevimg中得到的照片旋转，粘贴到另外一张照片中
def newRotateImage():
    imName = "createNevImg_past.jpg"
    print "正在将图片旋转中..."
    STA = time.time()
    im = Image.open(imName)
    im2 = Image.new("RGBA", (W_size * int(W_num + 1), H_size * (H_num + 4)))
    im2.paste(im, (int(0.5 * W_size), int(0.8 * H_size)))
    im2 = im2.rotate(359)
    im2.save("newRotateImage_past.jpg")
    print "newRotateImage Func Time %s"%(time.time()-STA)
```

- mName是获取前一步中保存的照片名。
- 并调用Image.open打开照片，并赋给图片对象im。
- 第五行是调用Image.new来创建一张新的照片im2，第一个参数是照片模式，第二个参数是照片的宽高，用元组表示。
- 然后调用paste方法将im放到第二个图片对象im2中，第二个参数是放置的中心点。
- 调用rotate方法，旋转照片
- 调用save方法，保存照片为newRoatateImage_o.5_past3.jpg，具体照片效果如下图所示：


**照片中添加文字**

```
# name: writetoimage
# todo: 在图片中写祝福语
def writeToImage():
    print "正在向图片中添加祝福语..."
    STA = time.time()
    img = Image.open("newRotateImage_past.jpg")
    font = ImageFont.truetype('xindexingcao57.ttf', 600)
    draw = ImageDraw.Draw(img)
    draw.ink = 21 + 118*256 + 65*256*256

#    draw.text((0,H_size * 6),unicode("happy every day",'utf-8'),(0,0,0),font=font)

    tHeight = H_num + 1
    draw.text((W_size * 0.5, H_size * tHeight), "happy life written by python", font = font)
    img.save("final_past.jpg")
    print "writeToImage Func Time %s"%(time.time()-STA)
```

- PIL功能很强大，可以向图片中添加文字
- font = ImageFont.truetype: 导入字体 ，第一个参数是字体文件，第二个参数是字体的大小
- draw = ImageDraw(img): 生成画笔，参数为要再图片中写字的图片对象。
- draw.ink = R + G*256 + B*256*256: 来设置画笔的颜色
- draw.text: 向图片对象中写字，第一个参数为字体的位置，用元组表示，第二参数是内容，第三个参数是字体。
- 最终利用save方法讲图片保存为final_0.5_past3.jpg。到此，工作基本完成了。


**相关说明**

- alpha：这个参数决定着层叠的效果，可以自由的改动这个参数，观察照片的变化，取值范围为(0--1)。
- 这里对PIL，numpy介绍并不详细，应该在阅读相关文档。相关文档
- 这里拼接的196张照片都是风景照，当把风景照换成人物照时，效果更好，最终的照片也更为漂亮。
- 由于采用了numpy，numexpr等库，加快了运算速度，past.py处理196张照片拼接、选择、层叠、添加文字，总共大概需要55秒左右，past3.py所需时间大概需要 23秒左右。
- 层叠的第一张照片很重要（即这里的lyf.jpg），可以使用PS修改一下尺寸等等。
- 分别运行past.py，past3.py，可以得到6张照片，后缀名分别为.past和.past3，可以进行比较，来体会两种层叠方式的不同


**参考链接：**[多张图片拼接与层叠](https://www.shiyanlou.com/courses/308/labs/948/document)
