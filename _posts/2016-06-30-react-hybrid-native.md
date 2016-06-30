---
layout: post
title: React Native原生混合与数据通信开发
date: 2016-6-30
categories: blog
tags: [React Native]
description: React Native原生混合与数据通信开发
---


**本文主要包括以下内容** 

1. React/JS界面调用原生界面  
2. React/JS界面调用原生界面 -等待原生返回数据  
3. 原生界面调用React/JS界面



### React/JS界面调用原生界面  

首先我们来看一下从React/JS界面进入原生界面，其实这一步是非常简单的。还记得我们上面一篇文章原生模块封装篇-基础篇，我们那时候讲解了如何封装一个Toast弹出提示的模块让React/JS进行使用。如果还记得我们在自定义继承ReactContextBaseJavaModule的类中定义了一个使用@ReactMethod注解的方法，该方法用于在React/JS前端来调用方法这个方法。在该方法中我们调用原生的Toast.show(xxx)的方法来弹出toast提醒。OK，这个原理搞清楚了之后，那么这边处理打开原生的界面就非常简单了。

在Android原生开发中，我们知道打开一个Activity一般有两种方法:显式和隐式。隐式方法一般通过AndroidManifest 配置文件中的Activity Intent-Filter中进行相关拦截配置。那么这边我们主要讲解的是显示启动Activity。

下面我们这边在创建继承ReactContextBaseJavaModule的IntentModule模块,具体代码如下:


```
package com.hunhedemo;
 
import android.app.Activity;
import android.content.Intent;
import android.text.TextUtils;
 
import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.JSApplicationIllegalArgumentException;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
 
/**
 * 当前类注释:
 * 项目名：android
 * 包名：com.hunhedemo
 * 作者：江清清 on 16/5/20 21:51
 * 邮箱：jiangqqlmj@163.com
 * QQ： 781931404
 * 公司：江苏中天科技软件技术有限公司
 */
public class IntentModule  extends ReactContextBaseJavaModule {
    public IntentModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
 
    @Override
    public String getName() {
        return "IntentModule";
    }
    /**
     * 从JS页面跳转到原生activity   同时也可以从JS传递相关数据到原生
     * @param name  需要打开的Activity的class
     * @param params
     */
    @ReactMethod
    public void startActivityFromJS(String name, String params){
        try{
            Activity currentActivity = getCurrentActivity();
            if(null!=currentActivity){
                Class toActivity = Class.forName(name);
                Intent intent = new Intent(currentActivity,toActivity);
                intent.putExtra("params", params);
                currentActivity.startActivity(intent);
            }
        }catch(Exception e){
            throw new JSApplicationIllegalArgumentException(
                    "不能打开Activity : "+e.getMessage());
        }
    }
}
```


看到以上的代码，我们会发现和之前封装Toast模块差不多，我们也在这边加入了一个@ReactMethod注解的startActivityFromJS方法，该用于在JS代码进行调用，从JS端传过来是两个参数，第一个参数为需要打开的Activity的class，第二个参数为传递过来的数据。至于为什么用class，那是因为原生代码这边用反射的形式更加方便了，其他方式都有局限性。这边我们根据传入的class，然后直接调用startActivity直接打开相应的Activity即可，甚至也可以携带一些参数值。具体调用方法方式如下:


```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
 
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  TouchableHighlight,
} from 'react-native';
var { NativeModules } = require('react-native');
class CustomButton extends Component {
  render() {
    return (
      <TouchableHighlight
        style={styles.button}
        underlayColor="#a5a5a5"
        onPress={this.props.onPress}>
        <Text style={styles.buttonText}>{this.props.text}</Text>
      </TouchableHighlight>
    );
  }
}
class hunheDemo extends Component {
   render() {
    return (
      <View>
        <Text style={styles.welcome}>
           JS跳转Activity界面
        </Text>
        <CustomButton
          text="点击跳转到Activity界面"
          onPress={()=>NativeModules.IntentModule.startActivityFromJS("com.hunhedemo.TwoActivity","我是从JS传过来的参数信息.456")}
        />
      </View>
    );
  }
}
const styles = StyleSheet.create({
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
   button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  },
});
 
AppRegistry.registerComponent('hunheDemo', () => hunheDemo);
```

请关注以上的主要两行代码:


```
var { NativeModules } = require('react-native');
 
NativeModules.IntentModule.startActivityFromJS("com.hunhedemo.TwoActivity","我是从JS传过来的参数信息.456")}
```


3. React/JS界面调用原生界面 -等待原生返回数据

经过上面的讲解，大家应该知道了启动原生界面怎么样搞了吧，还是非常简单的吧！下面我们来看另外一种情况。如果我们的React/JS界面打开了原生界面，同时获取到原生的返回数据呢？做过原生开发的童鞋肯定知道，原生Activity中回调的数据一般在onActivityResult中或者其他回调方法中的，但是如果需要返回给React/JS的数据是通过封装的原生模块方法中的Callback进行传输的。为了解决这个问题，我们这边创建一个阻塞的队列来实现，一旦有原生回调数据加入到队列中，那么数据就会从阻塞队列中取出来，再通过回调方法传入到React/JS界面中。

下面我们看一下重载了onActivityResult方法的MainActivity类中的写法:


```
package com.hunhedemo;
 
import android.content.Intent;
 
import com.facebook.react.ReactActivity;
import com.facebook.react.ReactPackage;
import com.facebook.react.shell.MainReactPackage;
 
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ArrayBlockingQueue;
 
public class MainActivity extends ReactActivity {
    //构建一个阻塞的单一数据的队列
    public static ArrayBlockingQueue<String> mQueue = new ArrayBlockingQueue<String>(1);
    /**
     * Returns the name of the main component registered from JavaScript.
     * This is used to schedule rendering of the component.
     */
    @Override
    protected String getMainComponentName() {
        return "hunheDemo";
    }
 
    /**
     * Returns whether dev mode should be enabled.
     * This enables e.g. the dev menu.
     */
    @Override
    protected boolean getUseDeveloperSupport() {
        return BuildConfig.DEBUG;
    }
 
    /**
     * A list of packages used by the app. If the app uses additional views
     * or modules besides the default ones, add more packages here.
     */
    @Override
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
            new MainReactPackage(),
                new IntentReactPackage()
        );
    }
 
    /**
     * 打开 带返回的Activity
     * @param requestCode
     * @param resultCode
     * @param data
     */
    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == RESULT_OK && requestCode == 200) {
            String result = data.getStringExtra("three_result");
            if (result != null && !result.equals("")) {
                mQueue.add(result);
            } else {
                mQueue.add("无数据啦");
            }
        } else {
            mQueue.add("没有回调...");
        }
    }
}
```


然后在IntentModule类中添加startActivityFromJSGetResult方法:


```
/**
     * 从JS页面跳转到Activity界面，并且等待从Activity返回的数据给JS
     * @param className
     * @param successBack
     * @param errorBack
     */
    @ReactMethod
    public void startActivityFromJSGetResult(String className,int requestCode,Callback successBack, Callback errorBack){
            try {
                Activity currentActivity=getCurrentActivity();
                if(currentActivity!=null) {
                    Class toActivity = Class.forName(className);
                    Intent intent = new Intent(currentActivity,toActivity);
                    currentActivity.startActivityForResult(intent,requestCode);
                    //进行回调数据
                    successBack.invoke(MainActivity.mQueue.take());
                }
            } catch (Exception e) {
                errorBack.invoke(e.getMessage());
                e.printStackTrace();
            }
 
    }
```


请注意上面的方法中，启动Activity是通过startActivityForResult()方法，这样打开的Activity有数据返回之后，才会调用之前的onActivityResult()方法，然后我们在这个方法中把回调的数据添加到阻塞队列中。

接下来我们看一下React/JS界面调用的方法:


```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
 
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  TouchableHighlight,
  ToastAndroid,
} from 'react-native';
var { NativeModules } = require('react-native');
class CustomButton extends Component {
  render() {
    return (
      <TouchableHighlight
        style={styles.button}
        underlayColor="#a5a5a5"
        onPress={this.props.onPress}>
        <Text style={styles.buttonText}>{this.props.text}</Text>
      </TouchableHighlight>
    );
  }
}
 
class hunheDemo extends Component {
   render() {
    return (
      <View>
        <Text style={styles.welcome}>
           React/JS与原生交互,数据通信实例
        </Text>
        <CustomButton
          text="点击跳转到Activity界面,并且等待数据返回..."
          onPress={()=>NativeModules.IntentModule.startActivityFromJSGetResult("com.hunhedemo.ThreeActivity",200,(msg) => {
                    ToastAndroid.show('JS界面:从Activity中传输过来的数据为:'+msg,ToastAndroid.SHORT);
                  },
                   (result) => {
                    ToastAndroid.show('JS界面:错误信息为:'+result,ToastAndroid.SHORT);
                  })} 
        />
      </View>
    );
  }
}
const styles = StyleSheet.create({
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
   button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  },
});
 
AppRegistry.registerComponent('hunheDemo', () => hunheDemo);
```


### 原生界面调用React/JS界面


到这边原生界面打开React/JS界面，还是非常简单的，直接启动配置了React Native的界面Activity即可，具体代码到时候大家看实例源代码即可了

打开React/JS界面是非常简单的，但是另外一个问题又出现了，我们如果想在打开React/JS界面同时，从原生Activity中传点数据过去呢？其实非常简单了，我们知道Activity之间打开传递数据会通过Intent的，那么我们就可以在承载React/JS界面的Activity中获取当前Intent中的数据，然后通过Callback方法回调即可。下面来一下实例源代码:

我们看一下IntentModule类中的dataToJS()方法:

```
package com.hunhedemo;
 
import android.app.Activity;
import android.content.Intent;
import android.text.TextUtils;
 
import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.JSApplicationIllegalArgumentException;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
 
/**
 * 当前类注释:
 * 项目名：android
 * 包名：com.hunhedemo
 * 作者：江清清 on 16/5/20 21:51
 * 邮箱：jiangqqlmj@163.com
 * QQ： 781931404
 * 公司：江苏中天科技软件技术有限公司
 */
public class IntentModule  extends ReactContextBaseJavaModule {
 
    public IntentModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
 
    @Override
    public String getName() {
        return "IntentModule";
    }
 
    /**
     * Activtiy跳转到JS页面，传输数据
     * @param successBack
     * @param errorBack
     */
    @ReactMethod
    public void dataToJS(Callback successBack, Callback errorBack){
        try{
            Activity currentActivity = getCurrentActivity();
            String result = currentActivity.getIntent().getStringExtra("data");
            if (TextUtils.isEmpty(result)){
                result = "没有数据";
            }
            successBack.invoke(result);
        }catch (Exception e){
            errorBack.invoke(e.getMessage());
        }
    }
   }
```


接着在React/JS端componentDidMount()方法中调用原生封装的方法去获取传过来的数据:


```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
 
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  TouchableHighlight,
  ToastAndroid,
} from 'react-native';
var { NativeModules } = require('react-native');
class CustomButton extends Component {
  render() {
    return (
      <TouchableHighlight
        style={styles.button}
        underlayColor="#a5a5a5"
        onPress={this.props.onPress}>
        <Text style={styles.buttonText}>{this.props.text}</Text>
      </TouchableHighlight>
    );
  }
}
 
class hunheDemo extends Component {
  //当组件挂载之后,去获取Activity传输过来的数据...
  componentDidMount(){
     //进行从Activity中获取数据传输到JS
     NativeModules.IntentModule.dataToJS((msg) => {
                    console.log(msg);
                    ToastAndroid.show('JS界面:从Activity中传输过来的数据为:'+msg,ToastAndroid.SHORT);
                  },
                   (result) => {
                    ToastAndroid.show('JS界面:错误信息为:'+result,ToastAndroid.SHORT);
                  })
 
  }
  render() {
    return (
      <View>
        <Text style={styles.welcome}>
           React/JS与原生交互,数据通信实例
        </Text>
      </View>
    );
  }
}
const styles = StyleSheet.create({
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
   button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  },
});
 
AppRegistry.registerComponent('hunheDemo', () => hunheDemo);
```


**source code**

[https://github.com/jiangqqlmj/hunheDemo](https://github.com/jiangqqlmj/hunheDemo)


**effect**

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/05/react_native_demo.gif)