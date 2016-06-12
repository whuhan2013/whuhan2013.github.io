---
layout: post
title: Volley学习总结
date: 2016-6-12
categories: blog
tags: [网络编程]
description: Volley学习总结
---


### 本文主要包括以下内容   


1. volly基本操作(String与Json类型)   
2. volly图片操作  
3. 自定义volly   
4. volly源码分析      


Volley简单易用，在性能方面也进行了大幅度的调整，它的设计目标就是非常适合去进行数据量不大，但通信频繁的网络操作，而对于大数据量的网络操作，比如说下载文件等，Volley的表现就会非常糟糕。


**在Android studio中导入volley**  

[Android Studio 中引入Volley - 简书](http://www.jianshu.com/p/b8c1a9f6a8fa)


### volly基本操作

1. 创建一个RequestQueue对象。
2. 创建一个StringRequest对象。
3. 将StringRequest对象添加到RequestQueue里面


```
        RequestQueue mQueue = Volley.newRequestQueue(getApplicationContext());
        StringRequest stringRequest = new StringRequest("http://www.baidu.com",
                new Response.Listener<String>() {
                    @Override
                    public void onResponse(String response) {
                        Log.d("volly", response);
                    }
                }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("volly", error.getMessage(), error);
            }
        });


        mQueue.add(stringRequest);
```



不过大家都知道，HTTP的请求类型通常有两种，GET和POST，刚才我们使用的明显是一个GET请求，那么如果想要发出一条POST请求应该怎么做呢？StringRequest中还提供了另外一种四个参数的构造函数，其中第一个参数就是指定请求类型的，我们可以使用如下方式进行指定：

```
StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener);  
```

可是这只是指定了HTTP请求方式是POST，那么我们要提交给服务器的参数又该怎么设置呢？很遗憾，StringRequest中并没有提供设置POST参数的方法，但是当发出POST请求的时候，Volley会尝试调用StringRequest的父类——Request中的getParams()方法来获取POST参数，那么解决方法自然也就有了，我们只需要在StringRequest的匿名类中重写getParams()方法，在这里设置POST参数就可以了，代码如下所示：

```
StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener) {  
    @Override  
    protected Map<String, String> getParams() throws AuthFailureError {  
        Map<String, String> map = new HashMap<String, String>();  
        map.put("params1", "value1");  
        map.put("params2", "value2");  
        return map;  
    }  
};  
```

**JsonRequest的用法**   


```
RequestQueue mQueue = Volley.newRequestQueue(getApplicationContext());
JsonObjectRequest jsonObjectRequest = new JsonObjectRequest("http://m.weather.com.cn/data/101010100.html", null,
        new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                Log.d("TAG", response.toString());
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.e("TAG", error.getMessage(), error);
            }
        });

mQueue.add(jsonObjectRequest);  
```



### volly图片操作


**ImageRequest的用法**   


前面我们已经学习过了StringRequest和JsonRequest的用法，并且总结出了它们的用法都是非常类似的，基本就是进行以下三步操作即可：   

1. 创建一个RequestQueue对象。           
2. 创建一个Request对象。                    
3. 将Request对象添加到RequestQueue里面。        

其中，StringRequest和JsonRequest都是继承自Request的，所以它们的用法才会如此类似。那么不用多说，今天我们要学习的ImageRequest，相信你从名字上就已经猜出来了，它也是继承自Request的，因此它的用法也是基本相同的.

```
RequestQueue mQueue = Volley.newRequestQueue(context);  

ImageRequest imageRequest = new ImageRequest(  
        "http://developer.android.com/images/home/aw_dac.png",  
        new Response.Listener<Bitmap>() {  
            @Override  
            public void onResponse(Bitmap response) {  
                imageView.setImageBitmap(response);  
            }  
        }, 0, 0, Config.RGB_565, new Response.ErrorListener() {  
            @Override  
            public void onErrorResponse(VolleyError error) {  
                imageView.setImageResource(R.drawable.default_image);  
            }  
        });  

mQueue.add(imageRequest);  

```
可以看到，ImageRequest的构造函数接收六个参数，第一个参数就是图片的URL地址，这个没什么需要解释的。第二个参数是图片请求成功的回调，这里我们把返回的Bitmap参数设置到ImageView中。第三第四个参数分别用于指定允许图片最大的宽度和高度，如果指定的网络图片的宽度或高度大于这里的最大值，则会对图片进行压缩，指定成0的话就表示不管图片有多大，都不会进行压缩。第五个参数用于指定图片的颜色属性，Bitmap.Config下的几个常量都可以在这里使用，其中ARGB_8888可以展示最好的颜色属性，每个图片像素占据4个字节的大小，而RGB_565则表示每个图片像素占据2个字节大小。第六个参数是图片请求失败的回调，这里我们当请求失败时在ImageView中显示一张默认图片。
最后将这个ImageRequest对象添加到RequestQueue里就可以了.



**ImageLoader的用法**     


1. 创建一个RequestQueue对象。            
2. 创建一个ImageLoader对象。            
3. 获取一个ImageListener对象。                   
4. 调用ImageLoader的get()方法加载网络上的图片。                 


```
ImageLoader imageLoader = new ImageLoader(mQueue, new ImageCache() {  
    @Override  
    public void putBitmap(String url, Bitmap bitmap) {  
    }  
  
    @Override  
    public Bitmap getBitmap(String url) {  
        return null;  
    }  
});  
可以看到，ImageLoader的构造函数接收两个参数，第一个参数就是RequestQueue对象，第二个参数是一个ImageCache对象，这里我们先new出一个空的ImageCache的实现即可。


ImageListener listener = ImageLoader.getImageListener(imageView,  
        R.drawable.default_image, R.drawable.failed_image);  

我们通过调用ImageLoader的getImageListener()方法能够获取到一个ImageListener对象，getImageListener()方法接收三个参数，第一个参数指定用于显示图片的ImageView控件，第二个参数指定加载图片的过程中显示的图片，第三个参数指定加载图片失败的情况下显示的图片。
最后，调用ImageLoader的get()方法来加载图片，代码如下所示：

imageLoader.get("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg", listener);       
get()方法接收两个参数，第一个参数就是图片的URL地址，第二个参数则是刚刚获取到的ImageListener对象。当然，如果你想对图片的大小进行限制，也可以使用get()方法的重载，指定图片允许的最大宽度和高度，如下所示：

imageLoader.get("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg",  
                listener, 200, 200);  
```



虽然现在我们已经掌握了ImageLoader的用法，但是刚才介绍的ImageLoader的优点却还没有使用到。为什么呢？因为这里创建的ImageCache对象是一个空的实现，完全没能起到图片缓存的作用。其实写一个ImageCache也非常简单，但是如果想要写一个性能非常好的ImageCache，最好就要借助Android提供的LruCache功能了，如果你对LruCache还不了解，可以参考[Android高效加载大图、多图解决方案，有效避免程序OOM](http://blog.csdn.net/guolin_blog/article/details/9316683)



** NetworkImageView的用法**  


除了以上两种方式之外，Volley还提供了第三种方式来加载网络图片，即使用NetworkImageView。不同于以上两种方式，NetworkImageView是一个自定义控制，它是继承自ImageView的，具备ImageView控件的所有功能，并且在原生的基础之上加入了加载网络图片的功能。NetworkImageView控件的用法要比前两种方式更加简单，大致可以分为以下五步：

1. 创建一个RequestQueue对象。          
2. 创建一个ImageLoader对象。                     
3. 在布局文件中添加一个NetworkImageView控件。        
4. 在代码中获取该控件的实例。         
5. 设置要加载的图片地址。         

其中，第一第二步和ImageLoader的用法是完全一样的，因此这里我们就从第三步开始学习了。首先修改布局文件中的代码，在里面加入NetworkImageView控件，如下所示：  


```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >

    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Send Request" />
    
    <com.android.volley.toolbox.NetworkImageView 
        android:id="@+id/network_image_view"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_gravity="center_horizontal"
        />

</LinearLayout>
```



接着在Activity获取到这个控件的实例


```
networkImageView = (NetworkImageView) findViewById(R.id.network_image_view);
networkImageView.setDefaultImageResId(R.drawable.default_image);  
networkImageView.setErrorImageResId(R.drawable.failed_image);  
networkImageView.setImageUrl("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg",  
                imageLoader);  
```


### 自定义自定义volly

**参见**  [Android Volley完全解析(三)，定制自己的Request - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/17612763)


### volly源码分析

**参见**   [Android Volley完全解析(四)，带你从源码的角度理解Volley - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/17656437)



### 参考链接

[Android Volley完全解析(一)，初识Volley的基本用法 - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/17482095)

[Android Volley完全解析(二)，使用Volley加载网络图片 - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/17482165)








