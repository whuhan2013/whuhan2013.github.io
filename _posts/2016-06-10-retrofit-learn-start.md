---
layout: post
title: Retrofit学习入门
date: 2016-6-10
categories: blog
tags: [网络编程]
description: Retrofit学习入门
---


### Retrofit的使用 

1. 设置权限与添加依赖  
2. 定义请求接口
3. 通过创建一个retrofit生成一个接口的实现类(动态代理)
4. 调用接口请求数据



### 设置权限与添加依赖

权限：首先确保在AndroidManifest.xml中请求了网络权限 ：


```
<uses-permission android:name="android.permission.INTERNET" />
```

（2）Studio用户,在app/build.gradle文件中添加如下代码：

```
dependencies {
    compile'com.squareup.retrofit2:retrofit:2.0.2'
    compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta3'
}
```

### 定义请求接口


```
public interface GankioService {
    @GET("data/福利/10/{page}")
    Call<GirlData> getGirls(@Path("page") int page);

    @GET("data/Android/10/{page}")
    Call<AndroidData> getAndroidData(@Path("page") int page);
}
```

请求的URL可以在函数中使用替换块和参数进行动态更新，替换块是{ and }包围的字母数字组成的字符串，相应的参数必须使用相同的字符串被@Path进行注释



### 然后创建一个retrofit


```

    public static GankioService buildGankioService() {
        if (mGankioService == null) {
            Retrofit retfGank = new Retrofit.Builder()
                    .baseUrl("http://gank.io/api/")
                    .addConverterFactory(GsonConverterFactory.create())
                    .build();
            mGankioService = retfGank.create(GankioService.class);
        }
        return mGankioService;
    }
```


retfGank.create(GankioService.class)用到了动态代理 

![](http://upload-images.jianshu.io/upload_images/2001089-6fc2a39a883f6485.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


大家一看就发现了 create方法返回了一个动态代理对象，那么动态代理是什么呢 ：
---在运行时， 动态代理类 实现了一个或者一组接口，目的是，其中任何一个接口的实例的方法调用将会被指派到统一的另一个接口的方法中。
InvocationHandler 中覆写的 invoke() 方法，在进行原本的方法调用之前或者之后，可以做点事情。
所以当我们调用 httpStores.getUser(hashMap);的时候其实是走了动态代理的invoke方法，在这里Retrofit巧妙的理由注解把接口转换成了一个HTTP请求



### 调用接口请求数据

```
 private void loadFromInternet(int page) {
        start = page;
        final Call<GirlData> girlDataCall = mService.getGirls(page);
        girlDataCall.enqueue(new Callback<GirlData>() {
            @Override
            public void onResponse(Call<GirlData> call, Response<GirlData> response) {
                mGirlData = response.body();

                List<Girl> newGirls = mGirlData.getGirls();
                // check if newGirls should be added into local girls
                if (checkShouldAdded(newGirls)){
                    addGirlsToDB(newGirls);
                }else{
                    shouldLoadFromInternet = false;
                }
                mView.showMore(newGirls);
                mView.finishRefresh();
            }

            @Override
            public void onFailure(Call<GirlData> call, Throwable t) {
                mView.finishRefresh();
                mView.showSnackBar();
            }
        });
    }
```


**其中GirlData是由接口返回的json数据格式决定的**

```
public class GirlData {


    /**
     * error : false
     * results : [{"_id":"56eb5db867765933d9b0a8fc","_ns":"ganhuo","createdAt":"2016-03-18T09:45:28.259Z","desc":"3.18","publishedAt":"2016-03-18T12:18:39.928Z","source":"chrome","type":"福利","url":"http://ww1.sinaimg.cn/large/7a8aed7bjw1f20ruz456sj20go0p0wi3.jpg","used":true,"who":"张涵宇"},{"_id":"56e8d0bb67765933d8be90be","_ns":"ganhuo","createdAt":"2016-03-16T11:19:23.692Z","desc":"3.16","publishedAt":"2016-03-17T11:14:16.306Z","source":"chrome","type":"福利","url":"http://ww4.sinaimg.cn/large/7a8aed7bjw1f1yjc38i9oj20hs0qoq6k.jpg","used":true,"who":"张涵宇"},{"_id":"56e8ce3967765933d8be90bd","_ns":"ganhuo","createdAt":"2016-03-16T11:08:41.957Z","desc":"3.16","publishedAt":"2016-03-16T11:24:01.505Z","source":"chrome","type":"福利","url":"http://ww3.sinaimg.cn/large/610dc034gw1f1yj0vc3ntj20e60jc0ua.jpg","used":true,"who":"代码家"},{"_id":"56e764116776592d80511280","_ns":"ganhuo","createdAt":"2016-03-15T09:23:29.580Z","desc":"3.15","publishedAt":"2016-03-15T11:45:57.350Z","source":"chrome","type":"福利","url":"http://ww4.sinaimg.cn/large/7a8aed7bjw1f1xad7meu2j20dw0ku0vj.jpg","used":true,"who":"张涵宇"},{"_id":"56e619a46776591744cf05c0","_ns":"ganhuo","createdAt":"2016-03-14T09:53:40.126Z","desc":"3.14","publishedAt":"2016-03-14T11:55:19.66Z","source":"chrome","type":"福利","url":"http://ww1.sinaimg.cn/large/7a8aed7bjw1f1w5m7c9knj20go0p0ae4.jpg","used":true,"who":"张涵宇"},{"_id":"56e220ca67765966681b3a23","_ns":"ganhuo","createdAt":"2016-03-11T09:35:06.879Z","desc":"3.11--一周年快乐！！！","publishedAt":"2016-03-11T12:37:20.4Z","source":"chrome","type":"福利","url":"http://ww4.sinaimg.cn/large/7a8aed7bjw1f1so7l2u60j20zk1cy7g9.jpg","used":true,"who":"张涵宇"},{"_id":"56e0f0e86776596669cc2511","_ns":"ganhuo","createdAt":"2016-03-10T11:58:32.298Z","desc":"3.10","publishedAt":"2016-03-10T12:54:31.68Z","source":"chrome","type":"福利","url":"http://ww4.sinaimg.cn/large/7a8aed7bjw1f1rmqzruylj20hs0qon14.jpg","used":true,"who":"张涵宇"},{"_id":"56df891167765947765e2ad1","_ns":"ganhuo","createdAt":"2016-03-09T10:23:13.778Z","desc":"3.9","publishedAt":"2016-03-09T12:06:26.401Z","source":"chrome","type":"福利","url":"http://ww2.sinaimg.cn/large/7a8aed7bjw1f1qed6rs61j20ss0zkgrt.jpg","used":true,"who":"张涵宇"},{"_id":"56de2b1b6776592b6192bf46","_ns":"ganhuo","createdAt":"2016-03-08T09:30:03.578Z","desc":"3.8","publishedAt":"2016-03-08T12:55:59.161Z","source":"chrome","type":"福利","url":"http://ww3.sinaimg.cn/large/7a8aed7bjw1f1p77v97xpj20k00zkgpw.jpg","used":true,"who":"张涵宇"},{"_id":"56dd06b56776592b6246e979","_ns":"ganhuo","createdAt":"2016-03-07T12:42:29.664Z","desc":"3.7","publishedAt":"2016-03-07T12:49:24.470Z","source":"chrome","type":"福利","url":"http://ww1.sinaimg.cn/large/7a8aed7bjw1f1o75j517xj20u018iqnf.jpg","used":true,"who":"张涵宇"}]
     */

    private boolean error;


    @SerializedName("results")
    private List<Girl> girls;

    public boolean isError() {
        return error;
    }

    public void setError(boolean error) {
        this.error = error;
    }

    public List<Girl> getGirls() {
        return girls;
    }

    public void setGirls(List<Girl> girls) {
        this.girls = girls;
    }


}
```


是不是很简单而且很优雅
当然Retrofit的内部实现更优雅使用了很多设计模式这里推荐一位大神的文章

[Retrofit分析-经典设计模式案例 - 简书](http://www.jianshu.com/p/fb8d21978e38)


下面在说一下Okhttp的拦截器Interceptor 这真的是一个用起来非常爽的东西实现起来也非常简单
上代码

```
public class OkhttpInterceptor implements Interceptor {
@Override
public Response intercept(Interceptor.Chain chain) throws IOException {
//取到当前请求的Requset
Request oldRequest = chain.request();
//取到请求的URL ，对你的URL进行修改，比如拼接一个UID什么的
//也可以使用addQueryParameter拼接参数
String url = oldRequest.url().toString()+"?="+uid;
Request newRequest = oldRequest.newBuilder()
.method(oldRequest.method(), oldRequest.body())
.url(url)
.build();
return chain.proceed(newRequest); }
}
当然你也可以增加请求参数
HttpUrl.Builder urlBuilder = oldRequest.url()newBuilder()
.scheme(oldRequest.url().scheme()) .addQueryParameter("key", "value")
只需要把url里的参数改为urlBuilder就行了
```


当然具体的业务需求也会遇到一些问题比如当我们的后台用的是https的时候，需要用到自签名证书，而OKhttp3已经没有了setCertificates设置自签名证书时怎么办，没关系我们还有万能的反射呢，下面我们就利用反射把https给过滤掉


```
SSLContext sc = null;
try {
sc = SSLContext.getInstance("SSL");
sc.init(null, new TrustManager[]{new X509TrustManager() {
@Override
public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) throws java.security.cert.CertificateException { }
@Override
public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) throws java.security.cert.CertificateException { }
@Override
public java.security.cert.X509Certificate[] getAcceptedIssuers() {
return null; } }
},
new SecureRandom());
} catch (Exception e) {
e.printStackTrace();
}
HostnameVerifier hv1 = new HostnameVerifier() {
@Override
public boolean verify(String hostname, SSLSession session) {
return true;
}};
String workerClassName = "okhttp3.OkHttpClient";
try {
sClient = new OkHttpClient.Builder().build();
Class workerClass = Class.forName(workerClassName);
Field hostnameVerifier = workerClass.getDeclaredField("hostnameVerifier"); hostnameVerifier.setAccessible(true);
hostnameVerifier.set(sClient, hv1);
Field sslSocketFactory = workerClass.getDeclaredField("sslSocketFactory"); sslSocketFactory.setAccessible(true);
sslSocketFactory.set(sClient, sc.getSocketFactory());
} catch (Exception e) {
e.printStackTrace();
}
```


这个反射只是用在忽略https上，现在国内主流还是http，一般是用不上的，拦截器的话要debug看看清楚，你可以在http执行请求之前做一些操作，比如加一些请求参数或者是判断什么的。


### 参考链接

[30分钟上手最火android网络请求框架Retrofit - 简书](http://www.jianshu.com/p/eb5d03085926)

[Retrofit2.0使用 - OPEN 开发经验库](http://www.open-open.com/lib/view/open1453552147323.html)

[快速Android开发系列网络篇之Retrofit - AngelDevil - 博客园](http://www.cnblogs.com/angeldevil/p/3757335.html)

[Retrofit分析-经典设计模式案例 - 简书](http://www.jianshu.com/p/fb8d21978e38)

[Retrofit 解析 JSON 数据 - 简书](http://www.jianshu.com/p/87c36d8dabce)


**Retrofit2.0**  

[我在stackoverflow回答的关于Retrofit2.0的相关问题](http://stackoverflow.com/questions/32311334/retrofit-2-0-beta1/32323588#32323588)  

[使用RxJava与Retrofit2.0使用的实例：Retrofit 2.0 RxJava Sample](http://www.jianshu.com/p/40050ded099c)

[JW大神的文章](https://realm.io/cn/news/droidcon-jake-wharton-simple-http-retrofit-2/)

[吴小龙Android Retrofit 2.0使用](http://wuxiaolong.me/2016/01/15/retrofit/)


