---
layout: post
title: Android之WebView学习
date: 2016-8-13
categories: blog
tags: [android]
description: webView
---

### WebView常用方法       

#### WebSettings
在使用WebView前我们都要进行相关的配置，常见的操作如下：   

```
 WebSettings settings = mWebView.getSettings();
        settings.setJavaScriptEnabled(true);    //支持javascript
        settings.setUseWideViewPort(true);    //设置webview推荐使用的窗口，使html界面自适应屏幕
        settings.setLoadWithOverviewMode(true);     //缩放至屏幕的大小
        settings.setAllowFileAccess(true);      //设置可以访问文件
//        settings.setDefaultZoom(WebSettings.ZoomDensity.MEDIUM);    //设置中等像素密度，medium=160dpi
        settings.setSupportZoom(true);    //设置支持缩放
        settings.setLoadsImagesAutomatically(true);    //设置自动加载图片
//        settings.setBlockNetworkImage(true);    //设置网页在加载的时候暂时不加载图片
//        settings.setAppCachePath("");   //设置缓存路径
        settings.setCacheMode(WebSettings.LOAD_NO_CACHE);   //设置缓存模式
```


#### WebChromeClient
WebChromeClient主要是辅助WebView处理Javascript的对话框，网站图标，网站title，加载进度等。

```
mWebView.setWebChromeClient(new WebChromeClient() {
            @Override
            public void onReceivedTitle(WebView view, String title) {
                super.onReceivedTitle(view, title);
                mTitle.setText(title);
            }

            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                super.onProgressChanged(view, newProgress);
                if (newProgress == 100) {
                    mProgressBar.setVisibility(View.GONE);
                }
                mProgressBar.setProgress(newProgress);
            }

        });
```


#### WebViewClient
WebViewClient用来辅助WebView处理各种通知、请求事件的，例如在WebView中点击请求新的链接、页面加载开始、结束等：

```
mWebView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                return true;
            }

            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
            }
        });
```

#### 上\下一个页面

```
private void goBack(){
        if (mWebView.canGoBack()){
            mWebView.goBack();
        }else{
            finish();
        }
    }

private void goForward() {
        if (mWebView.canGoForward()) {
            mWebView.goForward();
        }
    }
```

就是通过WebView的goBack()、goForward()方法。

#### 下载

```
mWebView.setDownloadListener(new DownloadListener() {
            @Override
            public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype,
                                        long contentLength) {

            }
        });

```

### java与js交互      

首先编写一个简单的H5：

```
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>js test</title>
</head>

<script type="text/javascript">
      function click1(){
         window.client.showMessage("来自js的消息");
      }

      function click2(msg){
         alert("来自java代码的消息:" + msg)
      }

</script>
<body>
    <input id="one" type="button"  value="js调用java代码" onclick="click1()"
     style="width:300px; height:150px; font-size:35px"/>
</body>
</html>
```

内容很简单，两个函数click1、click2，一个按钮，点击按钮执行click1函数。       

将改H5文件放到assets目录，之后用WebView加载改H5：

```
mWebView.loadUrl("file:///android_asset/test.html");
```

接下来在Activity中编写一个特殊的类：

```
class JsOperation {
        @JavascriptInterface
        public void showMessage(String msg) {
            Toast.makeText(MainActivity.this, msg, Toast.LENGTH_SHORT).show();
        }
    }
```

然后注入到WebView中：

```
mWebView.addJavascriptInterface(new JsOperation(), "client");
```

注意和js中的这行代码对比下：

```
window.client.showMessage("来自js的消息");
```

其中client就是addJavascriptInterface()的第二个参数，当然这个参数可以自定义，但要保持一致。js中调用的showMessage()方法，就是我们JsOperation类中的方法。

点击H5中的按钮，可以看到一个Toast提示：

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/android/p1.gif)

到这里就完成了js对java代码的调用，接下来看如下通过java代码调用js。
其实很简单：

```
mWebView.loadUrl("javascript:click2" + "(" + 1008611 + ")");
```

这样就可以执行js中的click2函数了。我们通过模拟器返回键执行这行代码，可以看到一个提示框，即click2函数得到执行：

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/android/p2.gif)


### 模拟原生应用的页面跳转   

效果

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/android/p2.jpg)

首先可以在Activity的onCreate()中创建一个栈，并将当前Activity入栈：

```
if (mStack == null) {
            mStack = new Stack<>();
        }

 mStack.push(this);
```

在WebView中继续加载新页面时：

```
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
                Intent intent = new Intent(MainActivity.this, MainActivity.class);
                intent.putExtra("url", url);
                startActivity(intent);

                return true;
            }
```

我们重写了shouldOverrideUrlLoading()，重新启动当前Activity来加载新链接，则一个新的Activtiy会入栈，这样就有了跳转的效果。
在返回上一个界面时进行出栈操作：

```
private void popActivity() {
        if (mStack.size() > 0) {
            mStack.remove(this);
            this.finish();
        }
    }
```

**源码**  

[源码](http://download.csdn.net/detail/shehuan320_/9599706)

**参考链接**  

[Android 你应该知道的WebView - 简书](http://www.jianshu.com/p/d533d2d44814)