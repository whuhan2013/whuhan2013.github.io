---
layout: post
title: github常用操作
date: 2016-4-7
categories: blog
tags: [github]
description: github常用操作
---

### 上传项目到github   

 1. 验证公钥是否正确ssh -T git@github.com   
 正确结果会显示：
 Warning:Permanently added 'github.com,207.97.227.239' (RSA) to the list of known hosts.
　　Hi whuhan2013! You've successfully authenticated, but GitHub does not provide shell access.    

2. clone刚才新建的repository 到本地，输入命令：git clone https://github.com/Flowerowl/stumansys.git  

3. 将想上传的代码目录拷贝到此文件夹下,并在git bash中将目录切换到该目录     
4. 输入以下命令   
- git init    
- git add .     
- git commit -m 'Test'     
- git remote add origin git@github.com:whuhan2013/XXX.git  
- git push -u origin master   

### 注意：有时会报failed to push some refs to git的错误，出现错误的主要原因是github中的README.md文件不在本地代码目录中，   
可以通过如下命令进行代码合并【注：pull=fetch+merge]     
git pull --rebase origin master   
执行上面代码后可以看到本地代码库中多了README.md文件  
此时再执行语句 git push -u origin master即可完成代码上传到github    

### github删除文件夹方法

1. git pull origin master  
2. git checkout 
3. git rm -rf dir   
4. git add .   
5. git commit -m ”remove dir”     
6. git push origin master   
7. input your name    
8. input your password    
 












