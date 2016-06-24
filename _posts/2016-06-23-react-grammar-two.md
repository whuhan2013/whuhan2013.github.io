---
layout: post
title: React之JSX入门
date: 2016-6-23
categories: blog
tags: [React Native]
description: React之JSX入门
---

React是由ReactJS与React Native组成，其中ReactJS是Facebook开源的一个前端框架，React Native
是ReactJS思想在native上的体现！  

JSX并不是一门新的语言，仅仅是个语法糖，允许开发者在JavaScript中书写HTML语法。，最后每个
HTML标签都转化为JavaScript代码来运行  

1.环境              
2.载入方式                                                    
3.标签 HTML标签 与 ReactJS创建的组件类标签(首字母一定要大写)          
4.转换 解析器        
<h3>输入</h3> 转换成：                
React.createElement("h3",null,"输入") 返回一个ReactElement对象    

5.执行JavaScript表达式

var msg="我是东方耀";           
'<h1>{msg}</h1>'       
React.createElement("h1",null,msg)             

6.注释 单行：// 多行：/注释文本/          

7.属性   

var msg=<h1 width="10px">我是东方耀</h1>           
React.createElement("h1",{width:"10px"},"我是东方耀")        

8.延展属性          
使用ES6的语法        
var props={}; props.foo="1"; props.bar="1";              
<h1 {...props} foo="2" >欢迎关注我的微信号</h1> 转换成：                
React.createElement("h1",React.__spread({},props,{foo:"2"}),"欢迎关注我的微信号")           

9.自定义属性(HTML5给出了方案，凡是以data-开头的自定义属性，可渲染到页面)           
10.显示HTML 显示一段HMTL字符串，而不是html节点        
借助一个属性 _html               
<div>{{_html:'<h1>我是东方耀，欢迎关注我的微信号</h1>'}}</div>         
11.样式的使用               
style属性 js对象 外层{}按照JSX语法 内层{}是JavaScript对象            
12.事件绑定              
注意：onClick 调用bind方法（设定作用域，要传递的参数）        


### ReactJS

ReactJS核心思想：组件化 维护自己的状态和UI 自动重新渲染      

多个组件组成了一个ReactJS应用           

- React是全局对象 顶层API与组件API
- React.createClass创建组件类的方法
- React.render渲染，将指定组件渲染到指定DOM节点
- render:组件级API,返回组件的内部结构
- React.render被ReactDOM.render替代  


**ReactJS组件生命周期**  

1.创建阶段            
getDefaultProps：处理props的默认值 在React.createClass调用           
2.实例化阶段          
React.render(<HelloMessage 启动之后          
getInitialState、componentWillMount、render、componentDidMount           

state：组件的属性，主要是用来存储组件自身需要的数据，每次数据的更新都是通过修改state属性的
值，ReactJS内部会监听state属性的变化，一旦发生变化的话，就会主动触发组件的render方法来更
新虚拟DOM结构

虚拟DOM：将真实的DOM结构映射成一个JSON数据结构      

3.更新阶段              
主要发生在用户操作之后或父组件有更新的时候，此时会根据用户的操作行为进行相应的页面结构的
调整              
componentWillReceiveProps、shouldComponentUpdate、componentWillUpdate、render、
componentDidUpdate     

4.销毁阶段               

销毁时被调用，通常做一些取消事件绑定、移除虚拟DOM中对应的组件数据结构、销毁一些无效的定
时器等工作
componentWillUnmount   


**ReactJS组件通信**  

ReactJS组件关系是一层套一层，DOM结构 组织结构比较清晰            
父组件 子组件            

1.子组件如何调用父组件            
this.props  

2.父组件如何调用子组件       
首先用属性ref给子组件取个名字吧              
this.refs.名字.getDOMNode().         


**生命周期实例** 

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>React组件的生命周期</title>
    <script type="text/javascript" src="react.js"></script>
    <script type="text/javascript" src="react-dom.js"></script>
    <script type="text/javascript" src="browser.min.js"></script>
</head>
<body>
<div id="example"></div>
<script type="text/babel">
    var HelloMessage=React.createClass(
            {
                //1、创建阶段
                getDefaultProps:function(){
                  //在创建类的时候被调用   this.props该组件的默认属性
                    console.log("getDefaultProps");
                    return {};
                },

                //2、实例化阶段
                getInitialState:function(){
                    //初始化组件的state值，其返回值会赋值给组件的this.state属性
                  //获取this.state的默认值
                    console.log("getInitialState");
                    return {};
                },

                componentWillMount:function(){
                  //在render之前调用此方法
                    //业务逻辑的处理都应该放在这里，如对state的操作等
                    console.log("componentWillMount");
                },



                render:function(){
                    //根据state值，渲染并返回一个虚拟DOM
                    console.log("render");
                    return <h1 style={{color:'#ff0000',fontSize:'24px'}} >Hello {this.props.name}!我是东方耀</h1>;
                    //这是注释  React.createElement
                },

                componentDidMount:function(){
                  //该方便发生在render方法之后
                    //在该方法中，ReactJS会使用render方法返回的虚拟DOM对象来创建真实的DOM结构
                    //组件内部可以通过this.getDOMNode()来获取当前组件的节点
                    console.log("componentDidMount");
                },

         //3、更新阶段，主要发生在用户操作之后或父组件有更新的时候，此时会根据用户的操作行为进行相应的页面结构的调整


             componentWillReceiveProps:function(){
                 //该方法发生在this.props被修改或父组件调用setProps()方法之后
                 //调用this.setState方法来完成对state的修改
                 console.log("componentWillRecieveProps");
             },
    shouldComponentUpdate:function() {
        //用来拦截新的props或state，根据逻辑来判断
        //是否需要更新
        console.log("shouldComponentUpdate");

        return true;
    },

        componentWillUpdate:function(){
            //shouldComponentUpdate返回true的时候执行
            //组件将更新
            console.log("componentWillUpdate");

        },

          componentDidUpdate:function(){
              //组件更新完毕，我们常在这里做一些DOM操作
              console.log("componentDidUpdate");
          },

    //4、销毁阶段
           componentWillUnmount:function(){
               //销毁时被调用，通常做一些取消事件绑定、移除虚拟DOM中对应的组件数据结构、销毁一些无效的定时器等工作
               console.log("componentWillUnmount");
           }

            }
    );

    ReactDOM.render(<HelloMessage name="React 语法基础8" /> ,document.getElementById('example'));




</script>
</body>
</html>
```


**组件之间通信实例** 

```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>React组件通信</title>
    <script type="text/javascript" src="react.js"></script>
    <script type="text/javascript" src="react-dom.js"></script>
    <script type="text/javascript" src="browser.min.js"></script>
</head>
<body>
<div id="example"></div>
<script type="text/babel">
    var Parent=React.createClass(
            {
                click:function(){
                    this.refs.child.getDOMNode().style.color="red";
                },
                render:function(){
                    return (
                            <div onClick={this.click} >Parent is :
                            <Child name={this.props.name}  ref="child"></Child>
            </div>

                    );

                }
            }
    );

    var Child=React.createClass({

        render:function(){
            return <span> {this.props.name} </span>;
        }

    });

    ReactDOM.render(<Parent name="React语法基础" /> ,document.getElementById('example'));




</script>
</body>
</html>
```


### React Native实战之调试与打包发布


http://localhost:8081/index.android.bundle?platform=android；当应用启动运行的时候，会自动拉取这
个bundle文件，该文件里存放的是应用的全部逻辑代码，在目录中并不存在这个文件，事实上，这个
地址只是一个请求地址，而非真正的静态资源文件，是通过包服务器packager通过动态分析
index.android.js中的依赖，并对其进行合并得到的，而且该服务允许代码实时渲染。

1.生成一个签名密钥                
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

最后它会生成一个叫做my-release-key.keystore的密钥库文件

2.找到路径/android/app/src/main，并在该目录下新建assets文件夹        

3.在工程目录下将index.android.bundle下载并保存到assets资源文件夹中            
  
curl -k "http://localhost:8081/index.android.bundle" > android/app/src/main/assets/index.android.bundle

这句命令是重点，如果assets目录中不存在该文件，则打包的apk在执行时显示空白。   

Protocol 'http not supported or disabled in libcurl           
Windows下安装使用curl命令:http://jingyan.baidu.com/article/a681b0dec4c67a3b1943467c.html          

4.添加gradle的android keystore配置           
打包的apk在未签名的情况下,在手机中（非root）是不允许安装的            
在build.gradle文件中            

```
//签名
  signingConfigs{
     release { 
     storeFile file("D://my-release-key.keystore")
      storePassword "286840jjx" 
      keyAlias "my-key-alias" 
      keyPassword "286840jjx" 
      } 
      } 

    //此外还要在BuildTypes中添加以下内容
    signingConfig signingConfigs.release
```


5.启用Proguard代码混淆来缩小APK文件的大小           

Proguard是一个Java字节码混淆压缩工具，它可以移除掉React Native Java（和它的依赖库中）中没
有被使用到的部分，最终有效的减少APK的大小。
重要：启用Proguard之后，你必须再次全面地测试你的应用。Proguard有时候需要为你引入的每个原
生库做一些额外的配置。参见app/proguard-rules.pro文件。  

def enableProguardInReleaseBuilds = true


6.在/android/目录中执行gradle assembleRelease命令，打包后的文件在
android/app/build/outputs/apk目录中，例如app-release.apk。如果打包碰到问题可以先执行 gradle
clean 清理一下。 

安装gradle工具（版本与android\gradle\wrapper下的一致），并配置环境变量，配置GRADLEHOME
到你的gradle根目录当中，然后把%GRADLEHOME%/bin（linux或mac的是$GRADLE_HOME/bin）加
到PATH的环境变量。              

配置完成之后，运行gradle -v，检查一下是否安装无误       

7.将apk发布到各大应用市场（BUILD SUCCESSFUL）      


   