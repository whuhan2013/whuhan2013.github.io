---
layout: post
title: PyQt开始    
date: 2017-3-19
categories: blog
tags: [python]
description: python
---

**创建窗口**         

```
from PyQt5.QtWidgets import *
from PyQt5.QtCore import *
from PyQt5.QtGui import *
from PyQt5 import *
import sys

class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        # 设置窗口标题
        self.setWindowTitle('My First App')
        self.setFixedSize(400,300)

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

![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/p2.png)

这里需要说明的是 Qt 的执行机制。 Qt 的程序通过创建 QApplication 类实例来调用 app.exec_ 方法进入事件循环。此时程序一直在循环监听各种事件并把它们放入消息队列中，在适当的时候从队列中取出处理。

#### 信号与槽          
Qt 中每种组件都有所谓的信号槽（slot）机制。可用来将信号与相应的处理函数进行连接绑定。

```
...
class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.windowTitleChanged.connect(self._my_func)
        ...

    # 自定义的信号处理函数
    def _my_func(self, s='my_func', a=100):
        dic = {'s': s, 'a': a}
        print(dic)

...
app.exec_()
```

这里将 QMainWindow 的信号 windowTitleChanged 与 _my_func 槽函数相绑定，当窗口标题被更改的信号发出的时候便会触发函数 _my_func 进行处理。

其中在自定义函数 _my_func 中允许设置任意多个参数。

当 windowTitleChanged 信号在被触发的时候向处理函数 _my_func 传递了一个参数也就是窗口标题。当然也有办法忽略这个值，就是采用 lamda 产生式。其原理就是通过 lamda 产生式的参数 x 捕获标题字符串，然后将 x 废弃不用，从而避免标题传入 _my_func。

修改如下：

```
self.windowTitleChanged.connect(lambda x: self._my_func('Shiyanlou', 666))
```

为了更加直观地理解信号与槽，我们进一步修改代码，通过创建按钮响应按钮事件来展示信号与槽机制。

```
class MainWindow(QMainWindow):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        # 设置窗口标题
        self.setWindowTitle('My First App')

        # 添加布局
        layout = QHBoxLayout()

        # 创建按钮
        for i in range(5):
            button = QPushButton(str(i))
            # 将按钮按压信号与自定义函数关联
            button.pressed.connect(lambda x=i: self._my_func(x))
            # 将按钮添加到布局中
            layout.addWidget(button)

        # 创建部件
        widget = QWidget()
        # 将布局添加到部件
        widget.setLayout(layout)
        # 将部件添加到主窗口上
        self.setCentralWidget(widget)

    # 自定义的信号处理函数
    def _my_func(self, n):
        print('click button %s' % n)
```


![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/p3.png)