---
layout: post
title: Retrofit与RXJava整合
date: 2016-6-11
categories: blog
tags: [网络编程]
description: Retrofit与RXJava整合
---


Retrofit 除了提供了传统的 Callback 形式的 API，还有 RxJava 版本的 Observable 形式 API。下面我用对比的方式来介绍 Retrofit 的 RxJava 版 API 和传统版本的区别。

以获取一个 User 对象的接口作为例子。使用Retrofit 的传统 API，你可以用这样的方式来定义请求：

```
@GET("/user")
public void getUser(@Query("userId") String userId, Callback<User> callback);
```


在程序的构建过程中， Retrofit 会把自动把方法实现并生成代码，然后开发者就可以利用下面的方法来获取特定用户并处理响应：

```
getUser(userId, new Callback<User>() {
    @Override
    public void success(User user) {
        userView.setUser(user);
    }

    @Override
    public void failure(RetrofitError error) {
        // Error handling
        ...
    }
};
```

而使用 RxJava 形式的 API，定义同样的请求是这样的：

```
@GET("/user")
public Observable<User> getUser(@Query("userId") String userId);
```

使用的时候是这样的：

```
getUser(userId)
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<User>() {
        @Override
        public void onNext(User user) {
            userView.setUser(user);
        }

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable error) {
            // Error handling
            ...
        }
    });
```

看到区别了吗？

当 RxJava 形式的时候，Retrofit 把请求封装进 Observable ，在请求结束后调用 onNext() 或在请求失败后调用 onError()。

对比来看， Callback 形式和 Observable 形式长得不太一样，但本质都差不多，而且在细节上 Observable 形式似乎还比 Callback 形式要差点。那 Retrofit 为什么还要提供 RxJava 的支持呢？

因为它好用啊！从这个例子看不出来是因为这只是最简单的情况。而一旦情景复杂起来， Callback 形式马上就会开始让人头疼。比如：

假设这么一种情况：你的程序取到的 User 并不应该直接显示，而是需要先与数据库中的数据进行比对和修正后再显示。使用 Callback 方式大概可以这么写：

```
getUser(userId, new Callback<User>() {
    @Override
    public void success(User user) {
        processUser(user); // 尝试修正 User 数据
        userView.setUser(user);
    }

    @Override
    public void failure(RetrofitError error) {
        // Error handling
        ...
    }
};
```

有问题吗？

很简便，但不要这样做。为什么？因为这样做会影响性能。数据库的操作很重，一次读写操作花费 10~20ms 是很常见的，这样的耗时很容易造成界面的卡顿。所以通常情况下，如果可以的话一定要避免在主线程中处理数据库。所以为了提升性能，这段代码可以优化一下：

```
getUser(userId, new Callback<User>() {
    @Override
    public void success(User user) {
        new Thread() {
            @Override
            public void run() {
                processUser(user); // 尝试修正 User 数据
                runOnUiThread(new Runnable() { // 切回 UI 线程
                    @Override
                    public void run() {
                        userView.setUser(user);
                    }
                });
            }).start();
    }

    @Override
    public void failure(RetrofitError error) {
        // Error handling
        ...
    }
};
```

性能问题解决，但……这代码实在是太乱了，迷之缩进啊！杂乱的代码往往不仅仅是美观问题，因为代码越乱往往就越难读懂，而如果项目中充斥着杂乱的代码，无疑会降低代码的可读性，造成团队开发效率的降低和出错率的升高。

这时候，如果用 RxJava 的形式，就好办多了。 RxJava 形式的代码是这样的：

```
getUser(userId)
    .doOnNext(new Action1<User>() {
        @Override
        public void call(User user) {
            processUser(user);
        })
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<User>() {
        @Override
        public void onNext(User user) {
            userView.setUser(user);
        }

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable error) {
            // Error handling
            ...
        }
    });
```

**其中** 


doOnNext()的执行在onNext()之前，对数据进行相关处理。doOnNext在observeOn所指定的线程中工作的         
OnNext和doOnNext都是请求成功后调用。当请求成功后会先调用doOnNext中保存用户信息的方法，然后才去执行OnNext中的方法。若请求失败，则不会调用doOnNext和OnNext中的方法。

**参考链接**

[RxJava操作符doOnNext - 享受技术带来的快乐！ - 博客频道 - CSDN.NET](http://blog.csdn.net/jdsjlzx/article/details/51499066)

[Rxjava中的doOnNext的作用和在哪里执行 - u010746364的博客 - 博客频道 - CSDN.NET](http://blog.csdn.net/u010746364/article/details/51438696)



后台代码和前台代码全都写在一条链中，明显清晰了很多。

再举一个例子：假设 /user 接口并不能直接访问，而需要填入一个在线获取的 token ，代码应该怎么写？

Callback 方式，可以使用嵌套的 Callback：

```
@GET("/token")
public void getToken(Callback<String> callback);

@GET("/user")
public void getUser(@Query("token") String token, @Query("userId") String userId, Callback<User> callback);

...

getToken(new Callback<String>() {
    @Override
    public void success(String token) {
        getUser(token, userId, new Callback<User>() {
            @Override
            public void success(User user) {
                userView.setUser(user);
            }

            @Override
            public void failure(RetrofitError error) {
                // Error handling
                ...
            }
        };
    }

    @Override
    public void failure(RetrofitError error) {
        // Error handling
        ...
    }
});
```

倒是没有什么性能问题，可是迷之缩进毁一生，你懂我也懂，做过大项目的人应该更懂。

而使用 RxJava 的话，代码是这样的：

```
@GET("/token")
public Observable<String> getToken();

@GET("/user")
public Observable<User> getUser(@Query("token") String token, @Query("userId") String userId);

...

getToken()
    .flatMap(new Func1<String, Observable<User>>() {
        @Override
        public Observable<User> onNext(String token) {
            return getUser(token, userId);
        })
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<User>() {
        @Override
        public void onNext(User user) {
            userView.setUser(user);
        }

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable error) {
            // Error handling
            ...
        }
    });
```

用一个 flatMap() 就搞定了逻辑，依然是一条链。看着就很爽，是吧？




### RxJava配合Retrofit2.0使用
新的Retrofit2.0简直就是设计模式的教科书典范，同时对Rx的支持也更加友好，本例子为查询ip获取地理信息，并过滤掉失败信息

```
//使用Rxjava配合Retrofit解析json数据，注意这里全是电脑运行的，没有分开线程订阅
public static void main(String[] args) throws Exception{
OkHttpClient client = new OkHttpClient();
client.interceptors().add(new LoggingInterceptor());//log for okhttp

Retrofit retrofit = new Retrofit.Builder().baseUrl(IPService.END).client(client)
    .addConverterFactory(GsonConverterFactory.create())//对Response进行adapter转换
    .addCallAdapterFactory(RxJavaCallAdapterFactory.create())//对转换后的数据进行再包装
    .build();

retrofit.create(IPService.class)//动态代理生成class
    //直接操作json数据，这里可不是一个好的习惯，真正应该是DTO对象的
    .getIPInfo("58.19.239.11")
    .filter(jsonObject -> jsonObject.get("code").getAsInt()==0)
    //转换数据类型
    .map(jsonObject1 -> jsonObject1.get("data"))
    //输出结果
    .subscribe(System.out::println);
}

//retrofit定义的接口
interface IPService {
    String END = "http://ip.taobao.com";
    //建议写成dto对象，博主只是为了演示filter就把这里JsonObject了
    @GET("/service/getIpInfo.php") Observable<JsonObject> getIPInfo(@Query("ip") String ip);
}

/**
* Retrofit2.0已经把网络部分剥离了，所以需要自己实现Log
*/
static class LoggingInterceptor implements Interceptor {
@Override public Response intercept(Chain chain) throws IOException {
  Request request = chain.request();

  long t1 = System.nanoTime();
  System.out.println(
      String.format("Sending request %s on %s%n%s", request.url(), chain.connection(),
          request.headers()));

  Response response = chain.proceed(request);

  long t2 = System.nanoTime();
  System.out.println(
      String.format("Received response for %s in %.1fms%n%s", response.request().url(),
          (t2 - t1) / 1e6d, response.headers()));

  return response;
}
```




**源代码**

[rengwuxian RxJava Samples](https://github.com/rengwuxian/RxJavaSamples)


**参考链接**               
[【Android】RxJava + Retrofit完成网络请求 - 简书](http://www.jianshu.com/p/1fb294ec7e3b)

[给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083#toc_14)

[函数式编程RxJava操作实例 - 简书](http://www.jianshu.com/p/40050ded099c)
