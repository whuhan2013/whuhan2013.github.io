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

