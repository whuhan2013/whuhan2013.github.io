---
layout: post
title: React之画廊应用
date: 2016-11-17
categories: blog
tags: [javascript]
description: React之画廊应用
---

#### 项目生成      
利用yo与webpack生成项目文件

```
npm install -g yo grunt-cli karma
npm install -g generator-react-webpack@1.2.11
mkdir react-grunt-example && cd $_
yo react-webpack
```

注意webpack版本发生了变化，所以需要指定1.2.11版本，安装时要注意安装npm install,把需要的依赖全装上，不然会出各种错误
完成后用grunt serve运行即可  

**使localhost:8000热更新实现**     
在index.html中添加<script type="text/javascript" src="/webpack-dev-server.js"></script>即可    

**安装loader** 

```
npm install autoprefixer-loader --save-dev
npm install json-loader --save-dev    
```



