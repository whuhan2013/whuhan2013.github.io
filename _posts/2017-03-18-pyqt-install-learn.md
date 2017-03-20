---
layout: post
title: PyQt5安装    
date: 2017-3-18
categories: blog
tags: [python]
description: python
---

PyQt是python平台下的QT库，在mac与Linux平台下可直接通过pip安装，在windows下要麻烦一点。     

Windows平台：win7-64bit

Python版本：Python 3.4.1

下载地址：

（官网）https://www.python.org/downloads/release/python-343/

（网盘）链接: http://pan.baidu.com/s/1geKjgdH 密码: hjnf

PyQt5版本：PyQt5-5.4-gpl-Py3.4-Qt5.4.0-x64.exe，
下载地址：

（官网）https://riverbankcomputing.com/software/pyqt/download5

（网盘）链接: http://pan.baidu.com/s/1miKVdOG 密码: eqar


安装步骤：

1.  安装Python3.4.1，默认安装路径：C:\Python34

2. 安装PyQt5，会根据Python的安装路径自动进行安装，不需要修改。

3. 测试安装是否成功。创建一个py文件，写入以下代码，运行后弹出widget窗口就说明安装成功了。

```
from PyQt5.QtWidgets import *
from PyQt5.QtCore import *
from PyQt5.QtGui import *

import sys

class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        # 设置窗口标题
        self.setWindowTitle('My First App')

        # 设置标签
        label = QLabel('Welcome to Shiyanlou!')
        # 设置标签显示在中央
        label.setAlignment(Qt.AlignCenter)
        self.setCentralWidget(label)

# 创建应用实例，通过 sys.argv 传入命令行参数
app = QApplication(sys.argv)
# 创建窗口实例
window = MainWindow()
# 显示窗口
window.show()
# 执行应用，进入事件循环
app.exec_()
```

![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/p1.png)

**如果需要PyQT的IDE界面编辑工具，要安装Eric。**  

Eric是一款python,ruby的IDE，其代码功能强大，与QT完美结合，使开发Python的图形界面应用程序变得更加容易。

```
官方网站：http://eric-ide.python-projects.org/
下载地址：http://sourceforge.net/projects/eric-ide/files/eric6/stable/6.0.3/
其他可选择的下载地址：
     http://nchc.dl.sourceforge.net/project/eric-ide/eric5/stable/5.1.5/eric5-5.1.5.zip
中文汉化包：http://nchc.dl.sourceforge.net/project/eric-ide/eric5/stable/5.1.5/ eric5-i18n-zh_CN.GB2312-5.1.5.zip   

安装：解压eric5-5.1.5.zip和eric5-i18n-zh_CN.GB2312-5.1.5.zip文件解压到相应的目录中，将eric5-i18n-zh_CN.GB2312-5.1.5文件夹中的文件全部拷贝至eric5-5.1.5文件夹中。再将eric5-5.1.5文件夹复制到C:\根目录下，点击eric5-5.1.5中的install.py文件进行安装。
配置：点击Eric安装目录中的eric5.bat（首次）或eric5-configure.bat进行配置。
           Editor—>Autocompation—>勾上所有的选框；
           QScintilla—>勾上左右的两个选框，然后在下面source中，选择from Document and API files.
           Editor—>APIs—>勾上Complie APIs Autocompation，然后在Language中，选择python。
           点面下面的Add from installed APIs按钮，选择需要的.api文件，最后点击Compile APIs。
```

**参考 ：**          
[Python PyQt5在Windows平台安装](http://blog.csdn.net/youngwhz1/article/details/51178104)    
[ windows安装PyQt5](http://blog.csdn.net/chaofanchang/article/details/50595446?locationNum=10)
[完美安装 Anaconda3 + PyQt5 + Eric6](http://blog.csdn.net/weiaitaowang/article/details/52045360)