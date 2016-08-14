---
layout: post
title: Picasso源码解析
date: 2016-8-14
categories: blog
tags: [源码解析]
description: 源码解析
---

Picasso是Square公司开源的一个Android平台上的图片加载框架,也是大名鼎鼎的JakeWharton的代表作品之一.对于图片加载和缓存框架,优秀的开源作品有不少。比如:Android-Universal-Image-Loader,Glide,fresco等等.我自己有在项目中使用过的有Picasso,U-I-L,Glide.对于一般的应用上面这些图片加载和缓存框架都是能满足使用的,Trinea有一篇关于这些框架的对比文章Android[三大图片缓存原理、特性对比](http://www.trinea.cn/android/android-image-cache-compare/)有兴趣了解的可以去看看,本文我们主要讲Picasso的使用方法和源码分析.


### 使用方法   

```
//加载一张图片
Picasso.with(this).load("url").placeholder(R.mipmap.ic_default).into(imageView);

//加载一张图片并设置一个回调接口
Picasso.with(this).load("url").placeholder(R.mipmap.ic_default).into(imageView, new Callback() {
    @Override
    public void onSuccess() {

    }

    @Override
    public void onError() {

    }
});

//预加载一张图片
Picasso.with(this).load("url").fetch();

//同步加载一张图片,注意只能在子线程中调用并且Bitmap不会被缓存到内存里.
new Thread() {
    @Override
    public void run() {
        try {
            final Bitmap bitmap = Picasso.with(getApplicationContext()).load("url").get();
            mHandler.post(new Runnable() {
                @Override
                public void run() {
                    imageView.setImageBitmap(bitmap);
                }
            });
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}.start();

//加载一张图片并自适应imageView的大小,如果imageView设置了wrap_content,会显示不出来.直到该ImageView的
//LayoutParams被设置而且调用了该View的ViewTreeObserver.OnPreDrawListener回调接口后才会显示.
Picasso.with(this).load("url").priority(Picasso.Priority.HIGH).fit().into(imageView);

//加载一张图片并按照指定尺寸以centerCrop()的形式缩放.
Picasso.with(this).load("url").resize(200,200).centerCrop().into(imageView);

//加载一张图片并按照指定尺寸以centerInside()的形式缩放.并设置加载的优先级为高.注意centerInside()或centerCrop()
//只能同时使用一种,而且必须指定resize()或者resizeDimen();
Picasso.with(this).load("url").resize(400,400).centerInside().priority(Picasso.Priority.HIGH).into(imageView);

//加载一张图片旋转并且添加一个Transformation,可以对图片进行各种变化处理,例如圆形头像.
Picasso.with(this).load("url").rotate(10).transform(new Transformation() {
    @Override
    public Bitmap transform(Bitmap source) {
        //处理Bitmap
        return null;
    }

    @Override
    public String key() {
        return null;
    }
}).into(imageView);

//加载一张图片并设置tag,可以通过tag来暂定或者继续加载,可以用于当ListView滚动是暂定加载.停止滚动恢复加载.
Picasso.with(this).load("url").tag(mContext).into(imageView);
Picasso.with(this).pauseTag(mContext);
Picasso.with(this).resumeTag(mContxt);
```


上面我们介绍了Picasso的大部分常用的使用方法.此外Picasso内部还具有监控功能.可以检测内存数据,缓存命中率等等.而且还会根据网络变化优化并发线程.下面我们就来进行Picasso的源码分析。这里说明一下,Picasso的源码分析目前在网络上已经有几篇比较好的分析文章,参考了以下文章   

[RowandJJ的Picasso源码学习](http://blog.csdn.net/chdjj/article/details/49964901)               
[闭门造车的Picasso源码分析系列](http://blog.happyhls.me/category/android/picasso/)


### 类关系图     
![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/android/p1.png)

从类图上我们可以看出Picasso的核心类主要包括:Picasso,RequestCreator,Action,Dispatcher,Request,RequestHandler,BitmapHunter等等.一张图片加载可以分为以下几步:

创建->入队->执行->解码->变换->批处理->完成->分发->显示(可选)           

下面就让我们来通过Picasso的调用流程来具体分析它的具体实现.      


### 源码分析

#### Picasso.with()方法的实现    

按照我们的惯例,我们从Picasso的调用流程开始分析,我们就从加载一张图片开始看起:

```
Picasso.with(this).load(url).into(imageView);
```

让我们先来看看Picasso.with()做了什么:

```
      public static Picasso with(Context context) {
        if (singleton == null) {
          synchronized (Picasso.class) {
            if (singleton == null) {
              singleton = new Builder(context).build();
            }
          }
        }
        return singleton;
      }
```

维护一个Picasso的单例,如果还未实例化就通过new Builder(context).build()创建一个singleton并返回,我们继续看Builder类的实现:


```
public static class Builder {

        public Builder(Context context) {
          if (context == null) {
            throw new IllegalArgumentException("Context must not be null.");
          }
          this.context = context.getApplicationContext();
        }

        /** Create the {@link Picasso} instance. */
        public Picasso build() {
          Context context = this.context;

          if (downloader == null) {
            //创建默认下载器
            downloader = Utils.createDefaultDownloader(context);
          }
          if (cache == null) {
            //创建Lru内存缓存
            cache = new LruCache(context);
          }
          if (service == null) {
            //创建线程池,默认有3个执行线程,会根据网络状况自动切换线程数
            service = new PicassoExecutorService();
          }
          if (transformer == null) {
            //创建默认的transformer,并无实际作用
            transformer = RequestTransformer.IDENTITY;
          }
          //创建stats用于统计缓存,以及缓存命中率,下载数量等等
          Stats stats = new Stats(cache);
          //创建dispatcher对象用于任务的调度
          Dispatcher dispatcher = new Dispatcher(context, service, HANDLER, downloader, cache, stats);

          return new Picasso(context, dispatcher, cache, listener, transformer, requestHandlers, stats,
              defaultBitmapConfig, indicatorsEnabled, loggingEnabled);
        }
      }
```


上面的代码里省去了部分配置的方法,当我们使用Picasso默认配置的时候(当然也可以自定义),最后会调用build()方法并配置好我们需要的各种对象,最后实例化一个Picasso对象并返回。最后在Picasso的构造方法里除了对这些对象的赋值以及创建一些新的对象,例如清理线程等等.最重要的是初始化了requestHandlers,下面是代码片段:


```
   List<RequestHandler> allRequestHandlers =
        new ArrayList<RequestHandler>(builtInHandlers + extraCount);

    // ResourceRequestHandler needs to be the first in the list to avoid
    // forcing other RequestHandlers to perform null checks on request.uri
    // to cover the (request.resourceId != 0) case.
    allRequestHandlers.add(new ResourceRequestHandler(context));
    if (extraRequestHandlers != null) {
      allRequestHandlers.addAll(extraRequestHandlers);
    }
    allRequestHandlers.add(new ContactsPhotoRequestHandler(context));
    allRequestHandlers.add(new MediaStoreRequestHandler(context));
    allRequestHandlers.add(new ContentStreamRequestHandler(context));
    allRequestHandlers.add(new AssetRequestHandler(context));
    allRequestHandlers.add(new FileRequestHandler(context));
    allRequestHandlers.add(new NetworkRequestHandler(dispatcher.downloader, stats));
    requestHandlers = Collections.unmodifiableList(allRequestHandlers);
```

可以看到除了添加我们可以自定义的extraRequestHandlers,另外添加了7个RequestHandler分别用来处理加载不同来源的资源,可能是Resource里的,也可能是File也可能是来源于网络的资源.这里使用了一个ArrayList来存放这些RequestHandler现在先不用了解这么做是为什么,下面我们会分析,到这我们就了解了Picasso.with()做了什么,接下来我们去看看load()方法.


#### load(),centerInside(),等方法的实现

在Picasso的load()方法里我们可以传入String,Uri或者File对象,但是其最终都是返回一个RequestCreator对象,如下所示:

```
  public RequestCreator load(Uri uri) {
        return new RequestCreator(this, uri, 0);
    }
```

再来看看RequestCreator的构造方法:

```
  RequestCreator(Picasso picasso, Uri uri, int resourceId) {
    if (picasso.shutdown) {
      throw new IllegalStateException(
          "Picasso instance already shut down. Cannot submit new requests.");
    }
    this.picasso = picasso;
    this.data = new Request.Builder(uri, resourceId, picasso.defaultBitmapConfig);
  }
```

首先是持有一个Picasso的对象,然后构建一个Request的Builder对象,将我们需要加载的图片的信息都保存在data里,在我们通过.centerCrop()或者.transform()等等方法的时候实际上也就是改变data内的对应的变量标识,再到处理的阶段根据这些参数来进行对应的操作,所以在我们调用into()方法之前,所有的操作都是在设定我们需要处理的参数,真正的操作都是有into()方法引起的。



#### into()方法的实现    

从上文中我们知道在我们调用了load()方法之后会返回一个RequestCreator对象,所以.into(imageView)方法必然是在RequestCreator里:     
 
```
public void into(ImageView target) {
    //传入空的callback
    into(target, null);
  }

  public void into(ImageView target, Callback callback) {
    long started = System.nanoTime();
    //检查调用是否在主线程
    checkMain();

    if (target == null) {
      throw new IllegalArgumentException("Target must not be null.");
    }
    //如果没有设置需要加载的uri,或者resourceId
    if (!data.hasImage()) {
      picasso.cancelRequest(target);
      //如果设置占位图片,直接加载并返回
      if (setPlaceholder) {
        setPlaceholder(target, getPlaceholderDrawable());
      }
      return;
    }
    //如果是延时加载,也就是选择了fit()模式
    if (deferred) {
      //fit()模式是适应target的宽高加载,所以并不能手动设置resize,如果设置就抛出异常
      if (data.hasSize()) {
        throw new IllegalStateException("Fit cannot be used with resize.");
      }
      int width = target.getWidth();
      int height = target.getHeight();
      //如果目标ImageView的宽或高现在为0
      if (width == 0 || height == 0) {
        //先设置占位符
        if (setPlaceholder) {
          setPlaceholder(target, getPlaceholderDrawable());
        }
        //监听ImageView的ViewTreeObserver.OnPreDrawListener接口,一旦ImageView
        //的宽高被赋值,就按照ImageView的宽高继续加载.
        picasso.defer(target, new DeferredRequestCreator(this, target, callback));
        return;
      }
      //如果ImageView有宽高就设置设置
      data.resize(width, height);
    }

    //构建Request
    Request request = createRequest(started);
    //构建requestKey
    String requestKey = createKey(request);

    //根据memoryPolicy来决定是否可以从内存里读取
    if (shouldReadFromMemoryCache(memoryPolicy)) {
      //通过LruCache来读取内存里的缓存图片
      Bitmap bitmap = picasso.quickMemoryCacheCheck(requestKey);
      //如果读取到
      if (bitmap != null) {
        //取消target的request
        picasso.cancelRequest(target);
        //设置图片
        setBitmap(target, picasso.context, bitmap, MEMORY, noFade, picasso.indicatorsEnabled);
        if (picasso.loggingEnabled) {
          log(OWNER_MAIN, VERB_COMPLETED, request.plainId(), "from " + MEMORY);
        }
        //如果设置了回调接口就回调接口的方法.
        if (callback != null) {
          callback.onSuccess();
        }
        return;
      }
    }
    //如果缓存里没读到,先根据是否设置了占位图并设置占位
    if (setPlaceholder) {
      setPlaceholder(target, getPlaceholderDrawable());
    }
    //构建一个Action对象,由于我们是往ImageView里加载图片,所以这里创建的是一个ImageViewAction对象.
    Action action =
        new ImageViewAction(picasso, target, request, memoryPolicy, networkPolicy, errorResId,
            errorDrawable, requestKey, tag, callback, noFade);

    //将Action对象入列提交
    picasso.enqueueAndSubmit(action);
  }
```

整个流程看下来应该是比较清晰的,最后是创建了一个ImageViewAction对象并通过picasso提交,这里简要说明一下ImageViewAction,实际上Picasso会根据我们调用的不同方式来实例化不同的Action对象,当我们需要往ImageView里加载图片的时候会创建ImageViewAction对象,如果是往实现了Target接口的对象里加载图片是则会创建TargetAction对象,这些Action类的实现类不仅保存了这次加载需要的所有信息,还提供了加载完成后的回调方法.也是由子类实现并用来完成不同的调用的。然后让我们继续去看picasso.enqueueAndSubmit(action)方法:


```
 void enqueueAndSubmit(Action action) {
    Object target = action.getTarget();
    //取消这个target已经有的action.
    if (target != null && targetToAction.get(target) != action) {
      // This will also check we are on the main thread.
      cancelExistingRequest(target);
      targetToAction.put(target, action);
    }
    //提交action
    submit(action);
  }
  //调用dispatcher来派发action
  void submit(Action action) {
    dispatcher.dispatchSubmit(action);
  }
```

很简单,最后是转到了dispatcher类来处理,那我们就来看看dispatcher.dispatchSubmit(action)方法:

```
void dispatchSubmit(Action action) {
    handler.sendMessage(handler.obtainMessage(REQUEST_SUBMIT, action));
  }
```

看到通过一个handler对象发送了一个REQUEST_SUBMIT的消息,那么这个handler是存在与哪个线程的呢?

```
Dispatcher(Context context, ExecutorService service, Handler mainThreadHandler,
      Downloader downloader, Cache cache, Stats stats) {
    this.dispatcherThread = new DispatcherThread();
    this.dispatcherThread.start();    
    this.handler = new DispatcherHandler(dispatcherThread.getLooper(), this);
    this.mainThreadHandler = mainThreadHandler;
  }

    static class DispatcherThread extends HandlerThread {
    DispatcherThread() {
      super(Utils.THREAD_PREFIX + DISPATCHER_THREAD_NAME, THREAD_PRIORITY_BACKGROUND);
    }
  }
```

上面是Dispatcher的构造方法(省略了部分代码),可以看到先是创建了一个HandlerThread对象,然后创建了一个DispatcherHandler对象,这个handler就是刚刚用来发送REQUEST_SUBMIT消息的handler,这里我们就明白了原来是通过Dispatcher类里的一个子线程里的handler不断的派发我们的消息,这里是用来派发我们的REQUEST_SUBMIT消息,而且最终是调用了 dispatcher.performSubmit(action);方法:

```
void performSubmit(Action action) {
    performSubmit(action, true);
  }

  void performSubmit(Action action, boolean dismissFailed) {
    //是否该tag的请求被暂停
    if (pausedTags.contains(action.getTag())) {
      pausedActions.put(action.getTarget(), action);
      if (action.getPicasso().loggingEnabled) {
        log(OWNER_DISPATCHER, VERB_PAUSED, action.request.logId(),
            "because tag '" + action.getTag() + "' is paused");
      }
      return;
    }
    //通过action的key来在hunterMap查找是否有相同的hunter,这个key里保存的是我们
    //的uri或者resourceId和一些参数,如果都是一样就将这些action合并到一个
    //BitmapHunter里去.
    BitmapHunter hunter = hunterMap.get(action.getKey());
    if (hunter != null) {
      hunter.attach(action);
      return;
    }

    if (service.isShutdown()) {
      if (action.getPicasso().loggingEnabled) {
        log(OWNER_DISPATCHER, VERB_IGNORED, action.request.logId(), "because shut down");
      }
      return;
    }

    //创建BitmapHunter对象
    hunter = forRequest(action.getPicasso(), this, cache, stats, action);
    //通过service执行hunter并返回一个future对象
    hunter.future = service.submit(hunter);
    //将hunter添加到hunterMap中
    hunterMap.put(action.getKey(), hunter);
    if (dismissFailed) {
      failedActions.remove(action.getTarget());
    }

    if (action.getPicasso().loggingEnabled) {
      log(OWNER_DISPATCHER, VERB_ENQUEUED, action.request.logId());
    }
  }
```

注释很详细,这里我们再分析一下forRequest()是如何实现的:

```
static BitmapHunter forRequest(Picasso picasso, Dispatcher dispatcher, Cache cache, Stats stats,
      Action action) {
    Request request = action.getRequest();
    List<RequestHandler> requestHandlers = picasso.getRequestHandlers();

    //从requestHandlers中检测哪个RequestHandler可以处理这个request,如果找到就创建
    //BitmapHunter并返回.
    for (int i = 0, count = requestHandlers.size(); i < count; i++) {
      RequestHandler requestHandler = requestHandlers.get(i);
      if (requestHandler.canHandleRequest(request)) {
        return new BitmapHunter(picasso, dispatcher, cache, stats, action, requestHandler);
      }
    }

    return new BitmapHunter(picasso, dispatcher, cache, stats, action, ERRORING_HANDLER);
  }
```


这里就体现出来了责任链模式,通过依次调用requestHandlers里RequestHandler的canHandleRequest()方法来确定这个request能被哪个RequestHandler执行,找到对应的RequestHandler后就创建BitmapHunter对象并返回.再回到performSubmit()方法里,通过service.submit(hunter);执行了hunter,hunter实现了Runnable接口,所以run()方法就会被执行,所以我们继续看看BitmapHunter里run()方法的实现:


```
  @Override public void run() {
    try {
      //更新当前线程的名字
      updateThreadName(data);

      if (picasso.loggingEnabled) {
        log(OWNER_HUNTER, VERB_EXECUTING, getLogIdsForHunter(this));
      }

      //调用hunt()方法并返回Bitmap类型的result对象.
      result = hunt();

      //如果为空,调用dispatcher发送失败的消息,
      //如果不为空则发送完成的消息
      if (result == null) {
        dispatcher.dispatchFailed(this);
      } else {
        dispatcher.dispatchComplete(this);
      }
    //通过不同的异常来进行对应的处理
    } catch (Downloader.ResponseException e) {
      if (!e.localCacheOnly || e.responseCode != 504) {
        exception = e;
      }
      dispatcher.dispatchFailed(this);
    } catch (NetworkRequestHandler.ContentLengthException e) {
      exception = e;
      dispatcher.dispatchRetry(this);
    } catch (IOException e) {
      exception = e;
      dispatcher.dispatchRetry(this);
    } catch (OutOfMemoryError e) {
      StringWriter writer = new StringWriter();
      stats.createSnapshot().dump(new PrintWriter(writer));
      exception = new RuntimeException(writer.toString(), e);
      dispatcher.dispatchFailed(this);
    } catch (Exception e) {
      exception = e;
      dispatcher.dispatchFailed(this);
    } finally {
      Thread.currentThread().setName(Utils.THREAD_IDLE_NAME);
    }
  }

  Bitmap hunt() throws IOException {
    Bitmap bitmap = null;
    //是否可以从内存中读取
    if (shouldReadFromMemoryCache(memoryPolicy)) {
      bitmap = cache.get(key);
      if (bitmap != null) {
        //统计缓存命中率
        stats.dispatchCacheHit();
        loadedFrom = MEMORY;
        if (picasso.loggingEnabled) {
          log(OWNER_HUNTER, VERB_DECODED, data.logId(), "from cache");
        }
        return bitmap;
      }
    }

    //如果未设置networkPolicy并且retryCount为0,则将networkPolicy设置为
    //NetworkPolicy.OFFLINE
    data.networkPolicy = retryCount == 0 ? NetworkPolicy.OFFLINE.index : networkPolicy;
    //通过对应的requestHandler来获取result
    RequestHandler.Result result = requestHandler.load(data, networkPolicy);
    if (result != null) {
      loadedFrom = result.getLoadedFrom();
      exifOrientation = result.getExifOrientation();
      bitmap = result.getBitmap();

      // If there was no Bitmap then we need to decode it from the stream.
      if (bitmap == null) {
        InputStream is = result.getStream();
        try {
          bitmap = decodeStream(is, data);
        } finally {
          Utils.closeQuietly(is);
        }
      }
    }

    if (bitmap != null) {
      if (picasso.loggingEnabled) {
        log(OWNER_HUNTER, VERB_DECODED, data.logId());
      }
      stats.dispatchBitmapDecoded(bitmap);
      //处理Transformation
      if (data.needsTransformation() || exifOrientation != 0) {
        synchronized (DECODE_LOCK) {
          if (data.needsMatrixTransform() || exifOrientation != 0) {
            bitmap = transformResult(data, bitmap, exifOrientation);
            if (picasso.loggingEnabled) {
              log(OWNER_HUNTER, VERB_TRANSFORMED, data.logId());
            }
          }
          if (data.hasCustomTransformations()) {
            bitmap = applyCustomTransformations(data.transformations, bitmap);
            if (picasso.loggingEnabled) {
              log(OWNER_HUNTER, VERB_TRANSFORMED, data.logId(), "from custom transformations");
            }
          }
        }
        if (bitmap != null) {
          stats.dispatchBitmapTransformed(bitmap);
        }
      }
    }
    //返回bitmap
    return bitmap;
  }
```


在run()方法里调用了hunt()方法来获取result然后通知了dispatcher来处理结果,并在try-catch里通知dispatcher来处理相应的异常,在hunt()方法里通过前面指定的requestHandler来获取相应的result,我们是从网络加载图片,自然是调用NetworkRequestHandler的load()方法来处理我们的request,这里我们就不再分析NetworkRequestHandler具体的细节.获取到result之后就获得我们的bitmap然后检测是否需要Transformation,这里使用了一个全局锁DECODE_LOCK来保证同一个时刻仅仅有一个图片正在处理。我们假设我们的请求被正确处理了,这样我们拿到我们的result然后调用了dispatcher.dispatchComplete(this);最终也是通过handler调用了dispatcher.performComplete()方法:


```
 void performComplete(BitmapHunter hunter) {
    //是否可以放入内存缓存里
    if (shouldWriteToMemoryCache(hunter.getMemoryPolicy())) {
      cache.set(hunter.getKey(), hunter.getResult());
    }
    //从hunterMap移除
    hunterMap.remove(hunter.getKey());
    //处理hunter
    batch(hunter);
    if (hunter.getPicasso().loggingEnabled) {
      log(OWNER_DISPATCHER, VERB_BATCHED, getLogIdsForHunter(hunter), "for completion");
    }
  }

  private void batch(BitmapHunter hunter) {
    if (hunter.isCancelled()) {
      return;
    }
    batch.add(hunter);
    if (!handler.hasMessages(HUNTER_DELAY_NEXT_BATCH)) {
      handler.sendEmptyMessageDelayed(HUNTER_DELAY_NEXT_BATCH, BATCH_DELAY);
    }
  }

  void performBatchComplete() {
    List<BitmapHunter> copy = new ArrayList<BitmapHunter>(batch);
    batch.clear();
    mainThreadHandler.sendMessage(mainThreadHandler.obtainMessage(HUNTER_BATCH_COMPLETE, copy));
    logBatch(copy);
  }
```

首先是添加到内存缓存中去,然后在发送一个HUNTER_DELAY_NEXT_BATCH消息,实际上这个消息最后会触发performBatchComplete()方法,performBatchComplete()里则是通过mainThreadHandler将BitmapHunter的List发送到主线程处理,所以我们去看看mainThreadHandler的handleMessage()方法:


```
static final Handler HANDLER = new Handler(Looper.getMainLooper()) {
    @Override public void handleMessage(Message msg) {
      switch (msg.what) {
        case HUNTER_BATCH_COMPLETE: {
          @SuppressWarnings("unchecked") List<BitmapHunter> batch = (List<BitmapHunter>) msg.obj;
          //noinspection ForLoopReplaceableByForEach
          for (int i = 0, n = batch.size(); i < n; i++) {
            BitmapHunter hunter = batch.get(i);
            hunter.picasso.complete(hunter);
          }
          break;
        }
        default:
          throw new AssertionError("Unknown handler message received: " + msg.what);
      }
    }
  };
```

很简单,就是依次调用picasso.complete(hunter)方法:

```
 void complete(BitmapHunter hunter) {
    //获取单个Action
    Action single = hunter.getAction();
    //获取被添加进来的Action
    List<Action> joined = hunter.getActions();

    //是否有合并的Action
    boolean hasMultiple = joined != null && !joined.isEmpty();
    //是否需要派发
    boolean shouldDeliver = single != null || hasMultiple;

    if (!shouldDeliver) {
      return;
    }

    Uri uri = hunter.getData().uri;
    Exception exception = hunter.getException();
    Bitmap result = hunter.getResult();
    LoadedFrom from = hunter.getLoadedFrom();

    //派发Action
    if (single != null) {
      deliverAction(result, from, single);
    }

    //派发合并的Action
    if (hasMultiple) {
      //noinspection ForLoopReplaceableByForEach
      for (int i = 0, n = joined.size(); i < n; i++) {
        Action join = joined.get(i);
        deliverAction(result, from, join);
      }
    }

    if (listener != null && exception != null) {
      listener.onImageLoadFailed(this, uri, exception);
    }
  }

 private void deliverAction(Bitmap result, LoadedFrom from, Action action) {
    if (action.isCancelled()) {
      return;
    }
    if (!action.willReplay()) {
      targetToAction.remove(action.getTarget());
    }
    if (result != null) {
      if (from == null) {
        throw new AssertionError("LoadedFrom cannot be null.");
      }
      //回调action的complete()方法
      action.complete(result, from);
      if (loggingEnabled) {
        log(OWNER_MAIN, VERB_COMPLETED, action.request.logId(), "from " + from);
      }
    } else {
      //失败则回调error()方法
      action.error();
      if (loggingEnabled) {
        log(OWNER_MAIN, VERB_ERRORED, action.request.logId());
      }
    }
  }
```


可以看出最终是回调了action的complete()方法,从前文知道我们这里是ImageViewAction,所以我们去看看ImageViewAction的complete()的实现:


```
@Override public void complete(Bitmap result, Picasso.LoadedFrom from) {
    if (result == null) {
      throw new AssertionError(
          String.format("Attempted to complete action with no result!\n%s", this));
    }

    //得到target也就是ImageView
    ImageView target = this.target.get();
    if (target == null) {
      return;
    }

    Context context = picasso.context;
    boolean indicatorsEnabled = picasso.indicatorsEnabled;
    //通过PicassoDrawable来将bitmap设置到ImageView上
    PicassoDrawable.setBitmap(target, context, result, from, noFade, indicatorsEnabled);

    //回调callback接口
    if (callback != null) {
      callback.onSuccess();
    }
  }
```

很显然通过了PicassoDrawable.setBitmap()将我们的Bitmap设置到了我们的ImageView上,最后并回调callback接口,这里为什么会使用PicassoDrawabl来设置Bitmap呢?使用过Picasso的都知道,Picasso自带渐变的加载动画,所以这里就是处理渐变动画的地方,由于篇幅原因我们就不做具体分析了,感兴趣的同学可以自行研究,所以到这里我们的整个Picasso的调用流程的源码分析就结束了.


### 设计模式

#### 建造者模式

建造者模式是指:将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。建造者模式应该是我们都比较熟悉的一种模式,在创建AlertDialog的时候通过配置不同的参数就可以展示不同的AlertDialog,这也正是建造者模式的用途,通过不同的参数配置或者不同的执行顺序来构建不同的对象,在Picasso里当构建RequestCreator的时候正是使用了这种设计模式,我们通过可选的配置比如centerInside(),placeholder()等等来分别达到不同的使用方式,在这种使用场景下使用建造者模式是非常合适的.

#### 责任链模式

责任链模式是指:一个请求沿着一条“链”传递，直到该“链”上的某个处理者处理它为止。当我们有一个请求可以被多个处理者处理或处理者未明确指定时。可以选择使用责任链模式,在Picasso里当我们通过BitmapHunter的forRequest()方法构建一个BitmapHunter对象时,需要为Request指定一个对应的RequestHandler来进行处理Request.这里就使用了责任链模式,依次调用requestHandler.canHandleRequest(request)方法来判断是否该RequestHandler能处理该Request如果可以就构建一个RequestHandler对象返回.这里就是典型的责任链思想的体现。


**参考链接**  

[Picasso源代分析 - 简书](http://www.jianshu.com/p/3c36382bc1cd)



