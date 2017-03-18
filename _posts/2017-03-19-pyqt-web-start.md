---
layout: post
title: PyQt实现简易浏览器    
date: 2017-3-19
categories: blog
tags: [python]
description: python
---

本文主要学习 PyQt 的使用，所以在实现浏览器的时候将不涉及底层传输协议与页面解析等内容。那么该怎么实现浏览器呢？所幸 Qt 为开发者提供了 QtWebKit 模块。

QtWebKit 是一个基于开源项目 WebKit 的网页内容渲染引擎，借助该引擎可以更加快捷地将万维网集成到 Qt 应用中。

通常来说一个浏览器应该要具备以下几个功能：

- 有一个可用来展示网页的窗口
- 拥有导航栏，地址栏
- 拥有标签，支持同时访问多个页面 接下来我们将逐一来实现以上提及的这些特性。

**创建浏览器**                 

```
from PyQt5.QtCore import *
from PyQt5.QtWidgets import *
from PyQt5.QtGui import *
from PyQt5.QtWebKitWidgets import *

import sys

class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # 设置窗口标题
        self.setWindowTitle('My Browser')
        self.setFixedSize(800,600)
        # 设置窗口图标
        self.setWindowIcon(QIcon('icons/penguin.png'))
        self.show()

        # 设置浏览器
        self.browser = QWebView()
        url = 'https://www.shiyanlou.com/'
        # 指定打开界面的 URL
        self.browser.setUrl(QUrl(url))
         # 添加浏览器到窗口中
        self.setCentralWidget(self.browser)

# 创建应用
app = QApplication(sys.argv)
# 创建主窗口
window = MainWindow()
# 显示窗口
window.show()
# 运行应用，并监听事件
app.exec_()
```

![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/chapter1/p1.png)

