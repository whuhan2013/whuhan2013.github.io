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

#### 图片组件的构建       

```
var controllerUnits = [],
imgFigures = [];
imageDatas.forEach(function (value, index) {
imgFigures.push(<ImgFigure key={index} data={value} ref={'imgFigure' + index} 
};

return (
        <section className="stage" ref="stage">
            <section className="img-sec">
                {imgFigures}
            </section>
            <nav className="controller-nav">
                {controllerUnits}
            </nav>
        </section>
    );
```

**图片的定位**      
使用absolute定位，没有指定top与left时，默认在左上角，可以通过改变top与left值改变位置      

![](http://img.mukewang.com/582002430001a49412800720.jpg)


**图片的旋转**     
同上，在state中添加rotate属性，随机产生－30-－30的角度即可   

```
/ 如果图片的旋转角度有值并且不为0， 添加旋转角度
        if (this.props.arrange.rotate) {
          (['MozTransform', 'msTransform', 'WebkitTransform', 'transform']).forEach(function (value) {
            styleObj[value] = 'rotate(' + this.props.arrange.rotate + 'deg)';
          }.bind(this));
        }
```


**图片翻转** 

```
/*
   * 翻转图片
   * @param index 传入当前被执行inverse操作的图片对应的图片信息数组的index值
   * @returns {Function} 这是一个闭包函数, 其内return一个真正待被执行的函数
   */
  inverse: function (index) {
    return function () {
      var imgsArrangeArr = this.state.imgsArrangeArr;

      imgsArrangeArr[index].isInverse = !imgsArrangeArr[index].isInverse;

      this.setState({
        imgsArrangeArr: imgsArrangeArr
      });
    }.bind(this);
  },
```


**切换动画**      

```
  给父结点perspective属性：perspective: 1800px;

  transform-origin: 0 50% 0;
            transform-style: preserve-3d;
            transition: transform .6s ease-in-out, left .6s ease-in-out, top .6s ease-in-out;

            &.is-inverse {
                transform: translate(320px) rotateY(180deg);
            }
```

![](http://img.mukewang.com/577f93440001e5ef12800720.jpg)     


**控制组件**     

```
transform: scale(.5);
transition: transform .6s ease-in-out, background-color .3s;

&.is-center {
    background-color: #888;

    transform: scale(1);

    &::after {
        color: #fff;
        font-family: "icons-turn-arrow";
        font-size: 80%;
        line-height: 30px;

        content: "\e600";

        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
    }

    &.is-inverse {
        background-color: #555;

        transform: rotateY(180deg);
    }
```

**上传到github** 

```
grunt serve:dist    
访问dist目录下的内容
git add dist 
git commit -m 'adddist'
git subtree push --prefix=dist origin gh-pages
推送本地的dist目录到gh-pages分支
```

**SourceCode:**[gallery-by-react](https://github.com/whuhan2013/gallery-by-react)    
**liveDemo:**[https://whuhan2013.github.io/web/album/](https://whuhan2013.github.io/web/album/)

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/react/p1.png)

