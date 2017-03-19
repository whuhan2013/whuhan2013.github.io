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

**添加导航栏**             
在这里我们要为浏览器添加导航栏，我们需要使用到 QToolBar 来创建，然后再往上边添加 QAction 创建按钮实例。      

```
...
class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        ...
        # 添加导航栏
        navigation_bar = QToolBar('Navigation')
        # 设定图标的大小
        navigation_bar.setIconSize(QSize(16, 16))
        self.addToolBar(navigation_bar)

        # 添加前进、后退、停止加载和刷新的按钮
        back_button = QAction(QIcon('icons/back.png'), 'Back', self)
        next_button = QAction(QIcon('icons/next.png'), 'Forward', self)
        stop_button = QAction(QIcon('icons/cross.png'), 'stop', self)
        reload_button = QAction(QIcon('icons/renew.png'), 'reload', self)

        back_button.triggered.connect(self.browser.back)
        next_button.triggered.connect(self.browser.forward)
        stop_button.triggered.connect(self.browser.stop)
        reload_button.triggered.connect(self.browser.reload)

        # 将按钮添加到导航栏上
        navigation_bar.addAction(back_button)
        navigation_bar.addAction(next_button)
        navigation_bar.addAction(stop_button)
        navigation_bar.addAction(reload_button)
...
```

现在我们不仅为浏览器添加了前进、后退、停止加载和刷新的按钮，同时还利用 QWebView 封装的槽实现了这些按钮的实际功能。

![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/chapter1/p2.png)

接下来我们再给浏览器添加地址栏。

```
...
class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        ...

        # 添加 URL 地址栏
        self.urlbar = QLineEdit()
        navigation_bar.addSeparator()
        navigation_bar.addWidget(self.urlbar)

        self.browser.urlChanged.connect(self.renew_urlbar)

    def renew_urlbar(self, q):
        # 将当前网页的链接更新到地址栏
        self.urlbar.setText(q.toString())
        self.urlbar.setCursorPosition(0)
...
```

![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/chapter1/p3.png)

这时候我们不仅添加了地址栏，而且地址栏还能实时更新成当前网页的 URL 。当然现在还不支持回车访问，下面
来实现这一点。          

```
...
class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        ...

        # 添加 URL 地址栏
        self.urlbar = QLineEdit()
        # 让地址栏能响应回车按键信号
        self.urlbar.returnPressed.connect(self.navigate_to_url)

        navigation_bar.addSeparator()
        navigation_bar.addWidget(self.urlbar)

        self.browser.urlChanged.connect(self.renew_urlbar)

    # 响应回车按钮，将浏览器当前访问的 URL 设置为用户输入的 URL
    def navigate_to_url(self):
        q = QUrl(self.urlbar.text())
        if q.scheme() == '':
            q.setScheme('http')
        self.browser.setUrl(q)
...
```

**添加标签页**            
为了添加标签页，我们对程序的 web 界面实现做较大的改动。               
主要实现以下功能

- 使用 QTabWidget 来给浏览器添加标签栏用于展示标签页面。
- 为标签栏添加按钮能动态地扩展标签页面。
- 添加关闭功能
- 将标签标题自动设置为网页所提供的标题

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
        # 设置窗口图标
        self.setWindowIcon(QIcon('icons/penguin.png'))
        self.show()

        # 添加标签栏
        self.tabs = QTabWidget()
        self.tabs.setDocumentMode(True)
        self.tabs.tabBarDoubleClicked.connect(self.tab_open_doubleclick)
        self.tabs.currentChanged.connect(self.current_tab_changed)
        self.tabs.setTabsClosable(True)
        self.tabs.tabCloseRequested.connect(self.close_current_tab)

        self.add_new_tab(QUrl('http://shiyanlou.com'), 'Homepage')

        self.setCentralWidget(self.tabs)

        new_tab_action = QAction(QIcon('icons/add_page.png'), 'New Page', self)
        new_tab_action.triggered.connect(self.add_new_tab)


        # 添加导航栏
        navigation_bar = QToolBar('Navigation')
        # 设定图标的大小
        navigation_bar.setIconSize(QSize(16, 16))
        self.addToolBar(navigation_bar)

        # 添加前进、后退、停止加载和刷新的按钮
        back_button = QAction(QIcon('icons/back.png'), 'Back', self)
        next_button = QAction(QIcon('icons/next.png'), 'Forward', self)
        stop_button = QAction(QIcon('icons/cross.png'), 'stop', self)
        reload_button = QAction(QIcon('icons/renew.png'), 'reload', self)

        back_button.triggered.connect(self.tabs.currentWidget().back)
        next_button.triggered.connect(self.tabs.currentWidget().forward)
        stop_button.triggered.connect(self.tabs.currentWidget().stop)
        reload_button.triggered.connect(self.tabs.currentWidget().reload)

        # 将按钮添加到导航栏上
        navigation_bar.addAction(back_button)
        navigation_bar.addAction(next_button)
        navigation_bar.addAction(stop_button)
        navigation_bar.addAction(reload_button)

        # 添加 URL 地址栏
        self.urlbar = QLineEdit()
        # 让地址栏能响应回车按键信号
        self.urlbar.returnPressed.connect(self.navigate_to_url)

        navigation_bar.addSeparator()
        navigation_bar.addWidget(self.urlbar)

    
    # 响应回车按钮，将浏览器当前访问的 URL 设置为用户输入的 URL
    def navigate_to_url(self):
        q = QUrl(self.urlbar.text())
        if q.scheme() == '':
            q.setScheme('http')
        self.tabs.currentWidget().setUrl(q)

    def renew_urlbar(self, q, browser=None):
        # 如果不是当前窗口所展示的网页则不更新 URL
        if browser != self.tabs.currentWidget():
            return
        # 将当前网页的链接更新到地址栏
        self.urlbar.setText(q.toString())
        self.urlbar.setCursorPosition(0)

    # 添加新的标签页
    def add_new_tab(self, qurl=QUrl(''), label='Blank'):
        # 为标签创建新网页
        browser = QWebView()
        browser.setUrl(qurl)
        i = self.tabs.addTab(browser, label)

        self.tabs.setCurrentIndex(i)

        browser.urlChanged.connect(lambda qurl, browser=browser: self.renew_urlbar(qurl, browser))

        browser.loadFinished.connect(lambda _, i=i, browser=browser: 
            self.tabs.setTabText(i, browser.page().mainFrame().title()))

    # 双击标签栏打开新页面
    def tab_open_doubleclick(self, i):
        if i == -1:
            self.add_new_tab()

    # 
    def current_tab_changed(self, i):
        qurl = self.tabs.currentWidget().url()
        self.renew_urlbar(qurl, self.tabs.currentWidget())

    def close_current_tab(self, i):
        # 如果当前标签页只剩下一个则不关闭
        if self.tabs.conut() < 2:
            return
        self.tabs.removeTab(i)

        
# 创建应用
app = QApplication(sys.argv)
# 创建主窗口
window = MainWindow()
# 显示窗口
window.show()
# 运行应用，并监听事件
app.exec_()
```

![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/chapter1/p4.png)

