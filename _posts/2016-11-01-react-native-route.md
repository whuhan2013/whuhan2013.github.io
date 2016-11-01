---
layout: post
title: 原生界面与ReactNative跳转
date: 2016-11-1
categories: blog
tags: [React Native]
description: 原生界面与ReactNative跳转
---

这里原生界面是指用布局文件实现或java代码实现view的Activity，React界面是指用ReactJS实现的界面的Activity。

从某种角度看，React只是充当了Android里的view层，因此原生界面与React界面的相互调用及数据传递同原生界面之间的互动基本是一致的。


#### React界面调用原生界面

1.只是启动原生界面

安卓中启动activity有隐式和显式，这里只考虑显示启动。显式启动需要指定目的activity的类，而我们虽然可以定制原生模块供JS调用，但并不能直接指定类或字节码为参数。一些特殊的情况下时可以用swtich的思路来解决，适用性更好的方法有没有呢，如何用允许的参数来得到目的类呢。这里就要要到反射了。

需要以下步骤

**写MyIntentModule**     

```
public class MyIntentModule extends ReactContextBaseJavaModule {
    public MyIntentModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }

    @Override
    public String getName() {
        return "MyIntentModule";  //注意此处不能返回null，否则无法建立桥接
    }

    /**
     * js页面跳转到activity 并传数据
     *
     *
     *
     */
    @ReactMethod
    public void startActivityByString(String activityName,String title){
        try {
            Activity currentActivity = getCurrentActivity();
            if (null != currentActivity) {

                Class aimActivity = Class.forName(activityName);
                Intent intent = new Intent(currentActivity,aimActivity);
                intent.putExtra("title", title);
                currentActivity.startActivity(intent);

            }
        } catch (Exception e) {
            throw new JSApplicationIllegalArgumentException(
                    "Could not open the activity : " + e.getMessage());
        }
    }
}
```

**完成module后要加入到package中**     

```
public class MyReactPackage implements ReactPackage {

    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        return Arrays.<NativeModule>asList(
                new MyIntentModule(reactContext)
        );
    }
    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
        return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
}
```

**要将上面的package加入到packagelist中，以前是加到mainactivity中，现在是放在MainApplication中**

```
public class MainApplication extends Application implements ReactApplication {

  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
    @Override
    protected boolean getUseDeveloperSupport() {
      return BuildConfig.DEBUG;
    }

    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
              new MyReactPackage()
      );
    }
  };

  @Override
  public ReactNativeHost getReactNativeHost() {
      return mReactNativeHost;
  }
}
```

**JS代码的调用** 

```
var { NativeModules } = require('react-native');

export default class AwesomeProject extends Component {
    onPressCallback = () => {
        
        NativeModules.MyIntentModule.startActivityByString("com.awesomeproject.SecondActivity","Ricardo")

    };
    render() {
        return (
          <View style = { styles.container } >
          <Text>Hello World</Text>
            <TouchableHighlight onPress = { this.onPressCallback }
              ><Text>Button</Text></TouchableHighlight>
              </View>
        );
    }
}
```

#### 原生界面调用React界面

1.只是启动React界面的话很简单，同原生界面间的启动一样，直接用startActivity即可。

2.在启动React界面时传递数据给React界面。

先来捋下思路，原生界面启动是没问题的，同原生一样；关键是React界面如何获取到传递过来的值：从前面的学习中我们知道原生模块是有很大自由度的，只要能得到React界面所在activity就可以顺着得到传递的来的intent，而JS端想得到数据就要利用回调函数了。

接下来是实现思路，从后往前来：首先先要按之前构建原生模块的步骤一步步来构建我们这次需要的功能

```
public class MyIntentModule extends ReactContextBaseJavaModule
@ReactMethod
public void getDataFromIntent(Callback successBack,Callback erroBack){
    try{
        Activity currentActivity = getCurrentActivity();
        String result = currentActivity.getIntent().getStringExtra("result");//会有对应数据放入
        if (TextUtils.isEmpty(result)){
            result = "No Data";
        }
        successBack.invoke(result);
    }catch (Exception e){
        erroBack.invoke(e.getMessage());
    }
}
public class MyReactPackage implements ReactPackage
@Override
public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
    return Arrays.<NativeModule>asList(
            new MyIntentModule(reactContext)
    );
}
public class MyReactActivity  extends ReactActivity
@Override
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
                new MainReactPackage(),
                new MyReactPackage()
        );
    }
```


在JS端配置相应代码


```
.......
class myreactactivity extends React.Component {
constructor(props) {
    super(props);   //这一句不能省略，照抄即可
    this.state = {
        TEXT: 'Input Text',//这里放你自己定义的state变量及初始值
    };
  }
  render() {
    return (
     <View style={styles.container}>
             <TextInput
                 style={{height: 40, borderColor: 'gray', borderWidth: 1,}}
                 onChangeText={(text) => this.setState({text})}
                 value={this.state.TEXT} />
     </View>
    )
  }
     componentDidMount(){   //这是React的生命周期函数，会在界面加载完成后执行一次
        React.NativeModules.MyIntentModule.getDataFromIntent(
            (successMsg) =>{
            this.setState({TEXT: successMsg,}); //状态改变的话重新绘制界面
             },
             (erroMsg) => {alert(erroMsg)}
        );
      }
}
.......
```

最后在原生界面将数据存放在intent里，然后用startActivity启动；

```
Intent intent =new Intent(this,MyReactActivity.class);
 intent.putExtra("result","haha");
 startActivity(intent,10);
```

**参考链接**        

[RN之Android:原生界面与React界面的相互调用及数据传递](https://github.com/ipk2015/RN-Resource-ipk/tree/master/react-native-docs)

[示例下载](http://download.csdn.net/download/mj924920789/9481621)


