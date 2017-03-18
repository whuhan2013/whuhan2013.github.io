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
git pull - -rebase origin master   
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

### 参考链接:    
[Github上传代码菜鸟超详细教程【转】 - 若风之觞 - 博客园](http://www.cnblogs.com/ruofengzhishang/p/3842587.html)    
[在GitHub上分享和展示你的代码 - teresa502的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/teresa502/article/details/7620127)       
[【Github教程】史上最全github使用方法：github入门到精通 - 水果君の日常 - 博客频道 - CSDN.NET](http://blog.csdn.net/hcbbt/article/details/11651229/)  


### github常用操作 

- windows下记住用户名与密码        
打开个人文件夹，一般为C:\Documents and Settings\用户名，其中有一个.gitconfig的文件，使用记事本打开。如果之前配置了名字和email的话，在这里面会看到。
我们追加如下配置即可              
[credential]            
helper=store            
下次我们再次输入用户名之后，git就会记住用户名密码，以后就不需再输入了。

这时在上述那个目录底下，可发现生成另外一个文件.git-credentials，里面记录的就是用户名密码了。 　　

- 设置全局用户名和email，作为每次提交的记录    
git config --global user.name “name"      
git config --global user.email “mail.com”      
  

- 添加一个仓库  
git remote add origin git@….git  
git push -u origin master  
  
- 当提示权限不够时，添加ssh公钥  
在用户的.ssh目录下找id_rsa.pub等文件，没有的话去生成  
ssh-keygen -t rsa -C "youremail@example.com”  
  
- 设置pull的默认地址  
git branch --set-upstream-to=origin/master  

- 设置push的默认地址  
git remote add origin git@….git  
  
- 配置别名  
git config --global alias.xx ''  
  
- 临时保存工作区  
git stash  
git stash pop  
  
- 回滚  
git reset —hard 版本号  
  
- 强行回滚远程服务器  
git push -f  


[利用Jekyll在GitHub Pages上部署博客 - Bannings的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/zhangao0086/article/details/37922607)  

**github发布githubPages**     
现在github发布pages简单了许多，只要在设置中开启即可。     
[Publishing with GitHub Pages, now as easy as 1, 2, 3](https://github.com/blog/2289-publishing-with-github-pages-now-as-easy-as-1-2-3)



### rebase的使用与意义

rebase与merge的区别

![](http://static.oschina.net/uploads/img/201511/09165236_lus8.jpg)

参考：[git rebase简介](https://my.oschina.net/china008/blog/528130)

 












