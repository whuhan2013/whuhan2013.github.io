---
layout: post
title: React学习入门
date: 2016-11-14
categories: blog
tags: [javascript]
description: React学习入门
---

![](http://img.mukewang.com/581f0f4500014daf12800720.jpg)    

**JSX: JS XML**    
语法糖：糖衣语法，计算机语言中添加的某种语法，对语言的功能没有影响，更方便程序员使用，增加程序的可读性，降低出错的可能性

类form结构，以及自定义html标签，这些统称为react Components，这并不是html标签，只是react Components的实例，通过调用ReactDOM.render(，)方法,第一个参数是需要渲染的react Components，第二个参数是渲染完以后需要呈现的Element位置
自定义的Components通过调用React.createClass()方法创建，参数是JS的object，
eg：

```
var Hello = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }

});

ReactDOM.render(
  <Hello name="World" />,
  document.getElementById('container')
);
```

{}表示其中要执行JS表达式的值    
this是指当前Components的实例，props表示使用react Components时，在其上添加的属性的集合，key值和定义时候的值相同例如{this.props.name}的值为World，和定义的name值相同

class 在ES6中，是一个关键字，用来声明一个类，在之前的ES5等标准中，是一个保留字，所以，在react中，不能再标签上直接写class，因为其是JS运行环境。声明css样式可以使用className。在react中行内样式不是 用字符串的形式表示，需要用样式对象来表示。eg：style={{color:'red'}},写法符合驼峰标示。    


**React生命周期**        

![](http://img.mukewang.com/581b34b10001eb2a12800720.jpg)       

**实例**     

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!-- es5 -->
    <!-- <script src="//cdn.bootcss.com/react/0.13.3/react.js"></script> -->
    <!-- <script src="//cdn.bootcss.com/react/0.13.3/JSXTransformer.js"></script> -->
    <!-- es6中react将react-dom单独剥离出来,所以需要单独加载react-dom,这里注意:如果babel-core的版本不匹配可能会在浏览器中报错提示缺少key值 -->
    <script src="http://cdn.bootcss.com/react/15.3.2/react.min.js"></script>
    <script src="http://cdn.bootcss.com/react/15.3.2/react-dom.min.js"></script>
    <script src="http://cdn.bootcss.com/babel-core/5.8.34/browser.min.js"></script>
</head>
<body>

    <div id="container"></div>

<script type="text/babel">
    /* es6 */
    class TestClickComponent extends React.Component{
        constructor(props){
            super(props);
            this.state = {}
            this.handleClick = this.handleClick.bind(this)
        }

        handleClick(){
            const tip = this.refs.tip;

            if (tip.style.display === 'none'){
                tip.style.display = 'inline'
            }else{
                tip.style.display = 'none'
            }
        }

        render() {
            return (
                <div>
                    <button onClick={this.handleClick}>显示|隐藏</button><span ref="tip">点击测试</span>
                </div>
            )
        }
    }

    class TestInputComponent extends React.Component{
        constructor(props){
            super(props);
            this.state={
                inputContent:''
            };
            this.changeHandle = this.changeHandle.bind(this)
        }

        changeHandle(){
            this.setState({
                inputContent:event.target.value
            });
            console.log(event.target.value)
        }

        render(){
            return(
                <div>
                    <input type="text" onChange={this.changeHandle} /><span>{this.state.inputContent}</span>
                </div>
            )

        }
    }

    ReactDOM.render(<div><TestClickComponent /><TestInputComponent /></div>,document.getElementById('container'));

    /* es5 */
//        var TestClickComponent = React.createClass({
//        render: function () {
//            return(
//                <div>
//                    <button>显示|隐藏</button><span>点击测试</span>
//                </div>
//            )
//        }
//    })
//
//    var TestInputComponent = React.createClass({
//        getInitialState: function () {
//            return{
//                inputContent:''
//            };
//        },
//
//        render: function () {
//            return(
//                    <div>
//                        <input type="text"/><span>{this.state.inputContent}</span>
//                    </div>
//            )
//        }
//    })
//
//    React.render(<div><TestClickComponent /><TestInputComponent /></div>,document.getElementById('container'))
</script>
</body>
</html>
```

stopPropagation()停止事件冒泡    
preventDefault()停止事件默认行为                              
React.findDOMNode(this.fess.<ref属性的值>): 索引子组键的dom元素     

**新版本需要注意的地方：**    

1，js文件引用换成最新的  

```
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/react/15.3.0/react.min.js"></script>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/react/15.3.0/react-dom.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>
```

2，
<script type="text/jsx">
换成<script type="text/babel">

3，var tipE = React.findDOMNode(this.refs.tip);
换成var tipE = ReactDOM.findDOMNode(this.refs.tip);

4,React.render
换成ReactDOM.render



