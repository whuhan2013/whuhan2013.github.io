---
layout: post
title: Retrofit与RxJava封装实现
date: 2016-7-29
categories: blog
tags: [网络编程]
description: 封装
---


**实现功能**         
Retrofit+RxJava的网络请求工具封装，自己感觉还是比较全面的，包括OKHttp拦截实现缓存，打印请求log信息，添加头部等功能     
首先对我们一般app应用的请求进行进行解析，一般我们得请求是这样的：

{"code":0,"desc":"success","content":{} }

```
public class HttpResult<T> {

    public int code;
    public String desc;
    public T content;


    public boolean isSuccess() {
        return code == Constant.SUCCESS;
    }

    public boolean isEmpty() {
        return code == Constant.EMPTY;
    }

    public boolean isNoMore() {
        return code == Constant.NOMORE;
    }
}
```

接着，我们再写一个类，用于封装所有请求API：APIService

```
/**
 * Created by hjzhang on 2016/7/26.
 */
public interface APIService {
    @GET("GetDayDiscountList")
    Observable<HttpResult<ListDayDiscount>> getDayDiscount(@Query("param") String param);
}
```


然后，再写一个总的工具类，用来配置Retrofit和OKHttp，以及公用的一些Rx方法，注释详细，请look代码：

```
/**
 * Created by hjzhang on 2016/7/26.
 */
public class RetrofitHttpUtil {
    /**
     * 服务器地址
     */
    private static final String BASE_URL = "http://app.aishangh.com/Id_Index.asmx/";

    public APIService apiService;
    private static Retrofit retrofit = null;
    private static OkHttpClient okHttpClient = null;

    private Context mContext;

    //缓存设置0不缓存
    private boolean isUseCache;
    private int maxCacheTime = 60;
    public void setMaxCacheTime(int maxCacheTime) {
        this.maxCacheTime = maxCacheTime;
    }

    public void setUseCache(boolean useCache) {
        isUseCache = useCache;
    }

    public APIService getService() {
        if (apiService == null && retrofit != null) {
            apiService = retrofit.create(APIService.class);
        }
        return apiService;
    }

    public void init(Context context) {
        this.mContext = context;
        initOkHttp();
        initRetrofit();
        if (apiService == null) {
            apiService = retrofit.create(APIService.class);
        }
    }

    private void initOkHttp() {
        OkHttpClient.Builder builder = new OkHttpClient.Builder();
        //打印请求log日志
        if (BuildConfig.DEBUG) {
            HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor();
            loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
            builder.addInterceptor(loggingInterceptor);
        }

        // 缓存 http://www.jianshu.com/p/93153b34310e
        File cacheFile = new File(AppUtil.getCacheDir(mContext), "httpCache");
//        Log.d("OkHttp", "缓存目录---" + cacheFile.getAbsolutePath());
        Cache cache = new Cache(cacheFile, 1024 * 1024 * 50);
        Interceptor cacheInterceptor = new Interceptor() {
            @Override
            public Response intercept(Chain chain) throws IOException {
                Request request = chain.request();
                if (!AppUtil.isNetworkConnected(mContext)||isUseCache) {//如果网络不可用或者设置只用缓存
                    request = request.newBuilder()
                            .cacheControl(CacheControl.FORCE_CACHE)
                            .build();
//                    Log.d("OkHttp", "网络不可用请求拦截");
                } else if(AppUtil.isNetworkConnected(mContext)&&!isUseCache){//网络可用
                    request = request.newBuilder()
                            .cacheControl(CacheControl.FORCE_NETWORK)
                            .build();
//                    Log.d("OkHttp", "网络可用请求拦截");
                }
                Response response = chain.proceed(request);
                if (AppUtil.isNetworkConnected(mContext)) {//如果网络可用
//                    Log.d("OkHttp", "网络可用响应拦截");
                    response = response.newBuilder()
                            //覆盖服务器响应头的Cache-Control,用我们自己的,因为服务器响应回来的可能不支持缓存
                            .header("Cache-Control", "public,max-age="+maxCacheTime)
                            .removeHeader("Pragma")
                            .build();
                } else {
//                    Log.d("OkHttp","网络不可用响应拦截");
//                    int maxStale = 60 * 60 * 24 * 28; // tolerate 4-weeks stale
//                    response= response.newBuilder()
//                            .header("Cache-Control", "public, only-if-cached, max-stale=" + maxStale)
//                            .removeHeader("Pragma")
//                            .build();
                }
                return response;

            }
        };
        builder.cache(cache);
        builder.interceptors().add(cacheInterceptor);//添加本地缓存拦截器，用来拦截本地缓存
        builder.networkInterceptors().add(cacheInterceptor);//添加网络拦截器，用来拦截网络数据
        //设置头部
//        Interceptor headerInterceptor = new Interceptor() {
//            @Override
//            public Response intercept(Chain chain) throws IOException {
//                Request originalRequest = chain.request();
//                Request.Builder requestBuilder = originalRequest.newBuilder()
//                        .header("myhead", "myhead")
//                        .header("Content-Type", "application/json")
//                        .header("Accept", "application/json")
//                        .method(originalRequest.method(), originalRequest.body());
//                Request request = requestBuilder.build();
//                return chain.proceed(request);
//            }
//        };
//        builder.addInterceptor(headerInterceptor );
        //设置超时
        builder.connectTimeout(15, TimeUnit.SECONDS);
        builder.readTimeout(20, TimeUnit.SECONDS);
        builder.writeTimeout(20, TimeUnit.SECONDS);
        //错误重连
        builder.retryOnConnectionFailure(true);
        okHttpClient = builder.build();

    }

    private void initRetrofit() {
        retrofit = new Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(okHttpClient)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .build();
    }


    public <T> void toSubscribe(Observable<T> o, Subscriber<T> s) {
        o.subscribeOn(Schedulers.io())
                .unsubscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(s);
    }

    /**
     * 用来统一处理Http的resultCode,并将HttpResult的Data部分剥离出来返回给subscriber
     *
     * @param <T> Subscriber真正需要的数据类型，也就是Data部分的数据类型
     */
    public class HttpResultFunc<T> implements Func1<HttpResult<T>, T> {

        @Override
        public T call(HttpResult<T> httpResult) {
            if (!httpResult.isSuccess()) {
                throw new APIException(httpResult.code, httpResult.desc);
            }
            return httpResult.content;
        }
    }
}
```

toSubscribe方法是公用的RxJava线程切换方法，网络请求在Schedulers.io()线程，而结果处理切换到AndroidSchedulers.mainThread()线程，妥妥的来去自如。

HttpResultFunc方法是处理返回结果的，有个自定义异常处理，除了有正确数据以外，其余转移给APIException处理。

```
/**
 * 自定义异常
 * Created by hjzhang on 2016/7/26.
 */
public class APIException extends RuntimeException{
    public int code;
    public String message;

    public APIException(int code, String message) {
        this.code = code;
        this.message = message;
    }

    @Override
    public String getMessage() {
        return message;
    }

    public int getCode() {
        return code;
    }
}
```

其中，log拦截器,缓存拦截器代码请看源码吧。

再接着，写一个工厂类，去实际产生调用处理结果，继承RetrofitHttpUtil ,对应APIService：APIFactory

```
public class APIFactory extends RetrofitHttpUtil{

    public static APIFactory getInstance() {
        return SingletonHolder.INSTANCE;
    }
    private static class SingletonHolder {
        private static final APIFactory INSTANCE = new APIFactory();
    }

    public void getDayDiscount(Subscriber<ListDayDiscount> subscriber, String param){
        Observable observable = apiService.getDayDiscount(param)
                .map(new HttpResultFunc<ListDayDiscount>());
        toSubscribe(observable, subscriber);
    }
}
```

**源码**             
完整源码https://yunpan.cn/c6jwYLT8cmTWH 访问密码 cef0

**参考链接**        
[Retrofit+RxJava最佳封装使用 - 简书](http://www.jianshu.com/p/566912926427)


**其他**              

- 源码解析：[阅读 Okhttp3.0 以及 Retrofit2.0 相关技术博客总结 ](http://allenwu.itscoder.com/2016/05/18/android-okhttp-retrofit/#一.OKHttp)
- 文件下载：[基于Retrofit+Okio+RxBus实现文件下载（带进度）](http://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650820593&idx=1&sn=6d27d2fc323bd134b4e579de224964fc&scene=0&ptlang=2052&ADUIN=2868405029&ADSESSION=1467935831&ADTAG=CLIENT.QQ.5437_.0&ADPUBNO=26517#wechat_redirect)
- 自定义了CallBack[Retrofit--合理封装回调能让你的项目高逼格 - 掘金](http://gold.xitu.io/entry/5767883b7f5785005489d55d)
- [Retrofit 源码解读之离线缓存策略的实现 - 简书](http://www.jianshu.com/p/3a8d910cce38)      
- [OkHttp3源码分析[缓存策略] - 简书](http://www.jianshu.com/p/9cebbbd0eeab)