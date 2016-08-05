---
layout: post
title: Retrofit源码解析
date: 2016-8-5
categories: blog
tags: [源码解析]
description: 源码解析
---   

**retrofit如何为我们的接口实现实例**

我们使用retrofit需要去定义一个接口，然后可以通过调用retrofit.create(IUserBiz.class);方法，得到一个接口的实例，最后通过该实例执行我们的操作，那么retrofit如何实现我们指定接口的实例呢？

其实原理是：动态代理。但是不要被动态代理这几个词吓唬到，Java中已经提供了非常简单的API帮助我们来实现动态代理。


```
return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod serviceMethod = loadServiceMethod(method);
            OkHttpCall okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.callAdapter.adapt(okHttpCall);
          }
        });
```


#### retrofit整体实现流程

**Retrofit的构建**

这里依然是通过构造者模式进行构建retrofit对象，好在其内部的成员变量比较少，我们直接看build()方法。

```
 public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }

      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
      adapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
          callbackExecutor, validateEagerly);
    }
```


- baseUrl必须指定，这个是理所当然的；
- 然后可以看到如果不着急设置callFactory，则默认直接new OkHttpClient()，可见如果你需要对okhttpclient进行详细的设置，需要构建OkHttpClient对象，然后传入；
- 接下来是callbackExecutor，这个想一想大概是用来将回调传递到UI线程了，当然这里设计的比较巧妙，利用platform对象，对平台进行判断，判断主要是利用Class.forName("")进行查找，具体代码已经被放到文末，如果是Android平台，会自定义一个Executor对象，并且利用Looper.getMainLooper()实例化一个handler对象，在Executor内部通过handler.post(runnable)，ok，整理凭大脑应该能构思出来，暂不贴代码了。
- 接下来是adapterFactories，这个对象主要用于对Call进行转化，基本上不需要我们自己去自定义。
- 最后是converterFactories，该对象用于转化数据，例如将返回的responseBody转化为对象等；当然不仅仅是针对返回的数据，还能用于一般备注解的参数的转化例如@Body标识的对象做一些操作，后面遇到源码详细再描述。
ok，总体就这几个对象去构造retrofit，还算比较少的~