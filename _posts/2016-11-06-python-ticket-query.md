---
layout: post
title: 火车票爬虫查询
date: 2016-11-6
categories: blog
tags: [网络爬虫]
description: 火车票爬虫查询
---

**本文主要用到以下知识点**        

- Python 基础知识的综合运用
- docopt用于解析参数      
- requests用于网络请求      
- colorama，命令行着色工具
- prettytable， 格式化信息打印工具，能让你像 MySQL 那样打印数据
- setuptools 的使用,将应用注册，可以在任意地方访问       

**解析参数**      

```
# coding: utf-8

"""命令行火车票查看器

Usage:
    tickets [-gdtkz] <from> <to> <date>

Options:
    -h,--help   显示帮助菜单
    -g          高铁
    -d          动车
    -t          特快
    -k          快速
    -z          直达

Example:
    tickets 北京 上海 2016-10-10
    tickets -dg 成都 南京 2016-10-10
"""
from docopt import docopt

def cli():
    """command-line interface"""
    arguments = docopt(__doc__)
    print(arguments)

if __name__ == '__main__':
    cli()
```

上面的程序中， docopt 会根据我们在 docstring 中的定义的格式自动解析出参数并返回一个字典，也就是 arguments， 我们打印出了这个字典的内容。下面我们运行一下这个程序， 比如查询一下10月30号从成都到南京的动车和高铁：

```
$ python tickets.py -dg 成都 南京 2016-10-10 
```

![](http://7xqdxb.com1.z0.glb.clouddn.com/tickets13.png)

**获取数据**   

将中文名称转换成查询代号       

```
import re
import requests
from pprint import pprint

url = 'https://kyfw.12306.cn/otn/resources/js/framework/station_name.js?station_version=1.8971'
response = requests.get(url, verify=False)
stations = re.findall(u'([\u4e00-\u9fa5]+)\|([A-Z]+)', response.text)

pprint(dict(stations), indent=4)
```

**解析数据**       

我们封装一个简单的类来解析数据：

```
class TrainsCollection:

    header = '车次 车站 时间 历时 一等 二等 软卧 硬卧 硬座 无座'.split()

    def __init__(self, available_trains, options):
        """查询到的火车班次集合

        :param available_trains: 一个列表, 包含可获得的火车班次, 每个
                                 火车班次是一个字典
        :param options: 查询的选项, 如高铁, 动车, etc...
        """
        self.available_trains = available_trains
        self.options = options

    def _get_duration(self, raw_train):
        duration = raw_train.get('lishi').replace(':', '小时') + '分'
        if duration.startswith('00'):
            return duration[4:]
        if duration.startswith('0'):
            return duration[1:]
        return duration

    @property
    def trains(self):
        for raw_train in self.available_trains:
            train_no = raw_train['station_train_code']
            initial = train_no[0].lower()
            if not self.options or initial in self.options:
                train = [
                    train_no,        
                    '\n'.join([raw_train['from_station_name'],
                              raw_train['to_station_name']]),
                    '\n'.join([raw_train['start_time'],
                               raw_train['arrive_time']]),
                    self._get_duration(raw_train),
                    raw_train['zy_num'],
                    raw_train['ze_num'],
                    raw_train['rw_num'],
                    raw_train['yw_num'],
                    raw_train['yz_num'],
                    raw_train['wz_num'],
                ]
                yield train

    def pretty_print(self):
        pt = PrettyTable()
        pt._set_field_names(self.header)
        for train in self.trains:
            pt.add_row(train)
        print(pt)
```

**显示结果**

最后，我们将上述过程进行汇总并将结果输出到屏幕上：

```
...

class TrainCollection:
    ...
    ...

def cli():
    """Command-line interface"""
    arguments = docopt(__doc__)
    from_station = stations.get(arguments['<from>'])
    to_station = stations.get(arguments['<to>'])
    date = arguments['<date>']
    url = ('https://kyfw.12306.cn/otn/lcxxcx/query?'
           'purpose_codes=ADULT&queryDate={}&'
           'from_station={}&to_station={}').format(
                date, from_station, to_station
           )
    # 获取参数
    options = ''.join([
        key for key, value in arguments.items() if value is True
    ])
    r = requests.get(url, verify=False)
    available_trains = r.json()['data']['datas']
    TrainsCollection(available_trains, options).pretty_print()
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/python/p1.png)

**着色**

至此， 程序的主体已经完成了， 但是上面打印出的结果是全是黑白的，很是乏味， 我们来给它添加点颜色吧！ 这里我们使用 colorama 这个命令行着色工具：

```
from colorama import init, Fore

init()
修改一下程序，将出发车站与出发时间显示为绿色，到达车站与到达时间显示为红色：

...
'\n'.join([Fore.GREEN + raw_train['from_station_name'] + Fore.RESET,
           Fore.RED + raw_train['to_station_name'] + Fore.RESET]),
'\n'.join([Fore.GREEN + raw_train['start_time'] + Fore.RESET,
           Fore.RED + raw_train['arrive_time'] + Fore.RESET]),
...
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/python/p2.png)


#### Setup

上面的程序中我们运行程序的方式是这样的：

```
python3 tickets.py from to date
```

我们当然可以将脚本改成可执行的，然后这样执行：

```
./tickets.py from to date
```

但这也不是理想的方案，因为我们必须在脚本的目录下才能运行。我们想要的是在命令行的任何地方都可以这样运行：

```
ticktes from to date
```

这是可以实现的，我们需要借助 Python 的 SETUP 工具。写一个简单的 setup 脚本：

```
from setuptools import setup

setup(
    name='tickets',
    py_modules=['tickets', 'stations'],
    install_requires=['requests', 'docopt', 'prettytable', 'colorama'],
    entry_points={
        'console_scripts': ['tickets=tickets:cli']
    }
)
```

在命令行运行一下：

```
python3 setup.py install
```
现在我们可以愉快的查询了：

![](http://7xqdxb.com1.z0.glb.clouddn.com/ticketes12.png)      

**源码：**[ticket](https://github.com/whuhan2013/pythoncode/tree/master/ticket)      
**参考：**[Python 实现火车票查询工具](https://www.shiyanlou.com/courses/623/labs/2072/document)





