<<<<<<< HEAD
---
layout: post
title: python实现统计你一共写了多少行代码
date: 2016-3-22
categories: blog
tags: [总结,python]
description: 通告一下，我准备开始了.
---
###程序员要保证一定的代码量就必须勤奋的敲代码，但怎么知道自己一共写了多少代码呢，笔者用python写了个简单的脚本，遍历所有的.java,.cpp,.c文件的行数，但是正如大家所知，java生成了许多代码，所以有许多水分，准确性并不太高，只具有一定的参考价值。

```
import os
import easygui as g
import sys
import chardet
path = 'C:/'
path='D:/Data/CProject/dataStruct/PAT/PAT'
path='F:/Workspaces'
path2='D:/Data/CProject'
path3='D:/WorkSpace/SoftWare/cocos2d-x-2.2.3/projects'
path4='F:/MyWrokspace'
iforjava=0;
iforcpp=0;
iforc=0;
```
```
for root, dirs, files in os.walk(path):
        #files=''.join(files)
        
        #print(type(files))
        for str1 in files:
                if 'java'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforjava=iforjava+int(count)
                        print('行数为:',count)
```
```
for root, dirs, files in os.walk(path2):
        #files=''.join(files)
        #print(type(files))
        for str1 in files:
                if 'c'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforc=iforc+int(count)
                        print('行数为:',count)
                if 'cpp'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforcpp=iforcpp+int(count)
                        print('行数为:',count)
```
```
for root, dirs, files in os.walk(path3):
        #files=''.join(files)      
        #print(type(files)) 
        for str1 in files:
                if 'c'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforc=iforc+int(count)
                        print('行数为:',count)
                if 'cpp'==str1.split('.').pop() and len(str1.split('.'))==2 and root.split('\\').pop()!='proj.wp8' and root.split('\\').pop()!='proj.winrt':
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        '''file =  open(root+'\\'+str1, "rb")#要有"rb"，如果没有这个的话，默认使用gbk读文件。           
                        buf = file.read()   
                        result = chardet.detect(buf)   
                        file = open(root+'\\'+str1,"r",encoding=result["encoding"])
                        count=len(file.readlines())'''
                        iforcpp=iforcpp+int(count)
                        print('行数为:',count)
```
```
for root, dirs, files in os.walk(path4):
        #files=''.join(files)
        
        #print(type(files))
        for str1 in files:
                if 'java'==str1.split('.').pop()and root.split('\\').pop()!='style':
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= ('gbk' or 'utf-8')).readlines())
                        iforjava=iforjava+int(count)
                        print('行数为:',count)
```
```
i=iforjava+iforc+iforcpp                       
print('总行数为:',i)
lineall=str(i)

g.msgbox("嗨，你一共写了"+lineall+"行代码，要继续加油哦^_^")
g.msgbox("其中\nC语言"+str(iforc)+"行\nC++"+str(iforcpp)+"行\njava"+str(iforjava)+"行")

os.system('pause')
```
###在打开文件的时候，老是因为GBK编码与UTF-8编码出错，因为不知道文件的编程格式，所以会以错误的编码方式打开，所说可以通过chardet包解决，但似乎还没有引入到python3中，所以只能手动改。。。。
##运行截图如下：
![运行](http://img.blog.csdn.net/20160313155458929)
![结果](http://img.blog.csdn.net/20160313155520367)
=======
---
layout: post
title: python实现统计你一共写了多少行代码
date: 2016-3-22
categories: blog
tags: [总结,python]
description: 通告一下，我准备开始了.
---
###程序员要保证一定的代码量就必须勤奋的敲代码，但怎么知道自己一共写了多少代码呢，笔者用python写了个简单的脚本，遍历所有的.java,.cpp,.c文件的行数，但是正如大家所知，java生成了许多代码，所以有许多水分，准确性并不太高，只具有一定的参考价值。
```
import os
import easygui as g
import sys
import chardet
path = 'C:/'
path='D:/Data/CProject/dataStruct/PAT/PAT'
path='F:/Workspaces'
path2='D:/Data/CProject'
path3='D:/WorkSpace/SoftWare/cocos2d-x-2.2.3/projects'
path4='F:/MyWrokspace'
iforjava=0;
iforcpp=0;
iforc=0;

for root, dirs, files in os.walk(path):
        #files=''.join(files)
        
        #print(type(files))
        for str1 in files:
                if 'java'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforjava=iforjava+int(count)
                        print('行数为:',count)

for root, dirs, files in os.walk(path2):
        #files=''.join(files)
        
        #print(type(files))
        for str1 in files:
                if 'c'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforc=iforc+int(count)
                        print('行数为:',count)
                if 'cpp'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforcpp=iforcpp+int(count)
                        print('行数为:',count)

for root, dirs, files in os.walk(path3):
        #files=''.join(files)
        
        #print(type(files))
        
        for str1 in files:
                if 'c'==str1.split('.').pop():
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        iforc=iforc+int(count)
                        print('行数为:',count)
                if 'cpp'==str1.split('.').pop() and len(str1.split('.'))==2 and root.split('\\').pop()!='proj.wp8' and root.split('\\').pop()!='proj.winrt':
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= 'gbk').readlines())
                        '''file =  open(root+'\\'+str1, "rb")#要有"rb"，如果没有这个的话，默认使用gbk读文件。           
                        buf = file.read()   
                        result = chardet.detect(buf)   
                        file = open(root+'\\'+str1,"r",encoding=result["encoding"])
                        count=len(file.readlines())'''
                        iforcpp=iforcpp+int(count)
                        print('行数为:',count)

for root, dirs, files in os.walk(path4):
        #files=''.join(files)
        
        #print(type(files))
        for str1 in files:
                if 'java'==str1.split('.').pop()and root.split('\\').pop()!='style':
                        print("Root = ", root, "dirs = ", dirs, "files = ", str1)
                        count= len(open(root+'\\'+str1,'rU',encoding= ('gbk' or 'utf-8')).readlines())
                        iforjava=iforjava+int(count)
                        print('行数为:',count)

i=iforjava+iforc+iforcpp                       
print('总行数为:',i)
lineall=str(i)

g.msgbox("嗨，你一共写了"+lineall+"行代码，要继续加油哦^_^")
g.msgbox("其中\nC语言"+str(iforc)+"行\nC++"+str(iforcpp)+"行\njava"+str(iforjava)+"行")

os.system('pause')
```
###在打开文件的时候，老是因为GBK编码与UTF-8编码出错，因为不知道文件的编程格式，所以会以错误的编码方式打开，所说可以通过chardet包解决，但似乎还没有引入到python3中，所以只能手动改。。。。
##运行截图如下：
![运行](http://img.blog.csdn.net/20160313155458929)
![结果](http://img.blog.csdn.net/20160313155520367)
>>>>>>> 09850fa2464b21194e80cd6f88ec2b52eb221e9c
![种类](http://img.blog.csdn.net/20160313155551134)