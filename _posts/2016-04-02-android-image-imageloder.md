---
layout: post
title: ImageLoader实现图片异步加载
date: 2016-4-2
categories: blog
tags: [android]
description: ImageLoader实现图片异步加载
---
### ImageLoader是一个广泛使用的图片库,在向网络请求图片时，使用imageView和smartView常会产生outofmemory错误，这时ImageLoader可以起到很大的作用，主要有如下功能   

### 一、功能特性：

1. 多线程异步加载和显示图片（图片来源于网络、sd卡、assets文件夹，drawable文件夹（不能加载9patch），新增加载视频缩略图）    

```
   "http://site.com/image.png" // from Web
  "file:///mnt/sdcard/image.png" // from SD card
  "file:///mnt/sdcard/video.mp4" // from SD card (video thumbnail)
  "content://media/external/images/media/13" // from content provider
  "content://media/external/video/media/13" // from content provider (video thumbnail)
  "assets://image.png" // from assets
  "drawable://" + R.drawable.img // from drawables (non-9patch images)
```  

2. 支持通过“listener”监视加载的过程，可以暂停加载图片，在经常使用的ListView、GridView中，可以设置滑动时暂停加载，停止滑动时加载图片（便于节约流量，在一些优化中可以使用）      

3. 缓存图片至内存时，可以更加高效的工作        
4.  高度可定制化（可以根据自己的需求进行各种配置，如：线程池，图片下载器，内存缓存策略等）      

5. 支持图片的内存缓存，SD卡（文件）缓存    

6. 在网络速度较慢时，还可以对图片进行加载并设置下载监听     


### 二 配置详解  

1. 载jar包放在libs文件夹中    
2.  AndroidManifest.xml     

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET" />
```

3.在应用中配置ImageLoaderConfiguration参数（只能配置一次，如多次配置，则默认第一次的配置参数）  
有两种配置方法

a、使用默认设置

```
public class CustomApplication extends Application{
  
  @Override
  public void onCreate() {
    // TODO Auto-generated method stub
    super.onCreate();
    ImageLoader.getInstance().init(ImageLoaderConfiguration.createDefault(CustomApplication.this));
  }

}
```   

b、使用自定义配置  

```
public class ImageLoaderApplication extends Application {
  public void onCreate() {
    super.onCreate();
    initImageLoader(getApplicationContext());
  }

  public static void initImageLoader(Context context) {
    //缓存文件的目录
    File cacheDir = StorageUtils.getOwnCacheDirectory(context, "imageloader/Cache"); 
    ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(context)
        .memoryCacheExtraOptions(480, 800) // max width, max height，即保存的每个缓存文件的最大长宽 
        .threadPoolSize(3) //线程池内加载的数量
        .threadPriority(Thread.NORM_PRIORITY - 2)
        .denyCacheImageMultipleSizesInMemory()
        .diskCacheFileNameGenerator(new Md5FileNameGenerator()) //将保存的时候的URI名称用MD5 加密
        .memoryCache(new UsingFreqLimitedMemoryCache(2 * 1024 * 1024)) // You can pass your own memory cache implementation/你可以通过自己的内存缓存实现
        .memoryCacheSize(2 * 1024 * 1024) // 内存缓存的最大值
        .diskCacheSize(50 * 1024 * 1024)  // 50 Mb sd卡(本地)缓存的最大值
        .tasksProcessingOrder(QueueProcessingType.LIFO)
        // 由原先的discCache -> diskCache
        .diskCache(new UnlimitedDiscCache(cacheDir))//自定义缓存路径  
        .imageDownloader(new BaseImageDownloader(context, 5 * 1000, 30 * 1000)) // connectTimeout (5 s), readTimeout (30 s)超时时间  
        .writeDebugLogs() // Remove for release app
        .build();
    //全局初始化此配置  
    ImageLoader.getInstance().init(config);
  }
}
```  

### 注意需要在manifest文件中声明Application  

```
 android:name="com.zj.more.CustomApplication"
```

4、图片显示操作   

### 显示图片

```
1、  ImageLoader.getInstance().displayImage(uri, imageView);
  2、  ImageLoader.getInstance().displayImage(uri, imageView, options);
  3、  ImageLoader.getInstance().displayImage(uri, imageView, listener);
  4、  ImageLoader.getInstance().displayImage(uri, imageView, options, listener);
  5、  ImageLoader.getInstance().displayImage(uri, imageView, options, listener, progressListener);
```
参数解析：
imageUrl   图片的URL地址   
imageView  显示图片的ImageView控件     
options    DisplayImageOptions配置信息    
listener   图片下载情况的监听     
progressListener  图片下载进度的监听    

1)方法1:最简单的方式，我们只需要定义要显示的图片的URL和要显示图片的ImageView。这种情况下，图片的显示选项会使用默认的配置

2)方法2:加载自定义配置的一个图片

3)方法3:加载带监听的一个图片

4)方法4:加载自定义配置且带监听的一个图片

5) 方法5:加载自定义配置且带监听和进度条的一个图片 

```
ImageLoader.getInstance().displayImage(uri, imageView, options,
        new ImageLoadingListener() {
          @Override
          public void onLoadingStarted(String arg0, View arg1) {
            //开始加载
          }
          @Override
          public void onLoadingFailed(String arg0, View arg1,
              FailReason arg2) {
            //加载失败
          }
          @Override
          public void onLoadingComplete(String arg0, View arg1,
              Bitmap arg2) {
            //加载成功
          }
          @Override
          public void onLoadingCancelled(String arg0, View arg1) {
            //加载取消
          }
        }, new ImageLoadingProgressListener() {
          @Override
          public void onProgressUpdate(String imageUri, View view,
              int current, int total) {
            //加载进度
          }
        });
```   

### 实例  

a、在BaseActivity中得到单例的imageLoader

```
public abstract class BaseActivity extends Activity {
  
  protected ImageLoader imageLoader;

    /**
     * 初始化布局资源文件
     */
    public abstract int initResource();

    /**
     * 初始化组件
     */
    public abstract void initComponent();

    /**
     * 初始化数据
     */
    public abstract void initData();

    /**
     * 添加监听
     */
    public abstract void addListener();


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(initResource());
        imageLoader = ImageLoader.getInstance();
        initComponent();
        initData();
        addListener();
    }

}
```   

b、listView实现加载图片  

```
public class ImageListActivity extends BaseActivity {

  private ListView mListImageLv;
  private DisplayImageOptions options; // 设置图片显示相关参数
  private String[] imageUrls; // 图片路径

  @Override
  public int initResource() {
    return R.layout.activity_list;
  }

  @Override
  public void initComponent() {
    mListImageLv = (ListView) findViewById(R.id.lv_image);
  }

  @Override
  public void initData() {
    Bundle bundle = getIntent().getExtras();
    imageUrls = bundle.getStringArray(Constants.IMAGES);

    // 使用DisplayImageOptions.Builder()创建DisplayImageOptions
    options = new DisplayImageOptions.Builder()
        .showImageOnLoading(R.drawable.ic_stub) // 设置图片下载期间显示的图片
        .showImageForEmptyUri(R.drawable.ic_empty) // 设置图片Uri为空或是错误的时候显示的图片
        .showImageOnFail(R.drawable.ic_error) // 设置图片加载或解码过程中发生错误显示的图片
        .cacheInMemory(true) // 设置下载的图片是否缓存在内存中
        .cacheOnDisk(true) // 设置下载的图片是否缓存在SD卡中
        .displayer(new RoundedBitmapDisplayer(20)) // 设置成圆角图片
        .build(); // 构建完成

    mListImageLv.setAdapter(new ItemListAdapter());
  }

  @Override
  public void addListener() {

  }

  class ItemListAdapter extends BaseAdapter {

    @Override
    public int getCount() {
      return imageUrls.length;
    }

    @Override
    public Object getItem(int position) {
      return imageUrls[position];
    }

    @Override
    public long getItemId(int position) {
      return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
      ViewHolder viewHolder = null;
      if (convertView == null) {
        convertView = getLayoutInflater().inflate(R.layout.item_list,
            null);
        viewHolder = new ViewHolder();
        viewHolder.image = (ImageView) convertView
            .findViewById(R.id.iv_image);
        viewHolder.text = (TextView) convertView
            .findViewById(R.id.tv_introduce);
        convertView.setTag(viewHolder);
      } else {
        viewHolder = (ViewHolder) convertView.getTag();
      }

      /**
       * imageUrl 图片的Url地址 imageView 承载图片的ImageView控件 options
       * DisplayImageOptions配置文件
       */
      imageLoader.displayImage(imageUrls[position],
          viewHolder.image, options);
      viewHolder.text.setText("Item " + (position + 1)); // TextView设置文本

      return convertView;
    }

    public class ViewHolder {
      public ImageView image;
      public TextView text;
    }

  }

}
```  

其中imageUrls 由以下MainActivity中的内容发送而来

```
// 点击进入ListView展示界面  
    public void onImageListClick(View view) {  
       Intent intent = new Intent(this, ImageListActivity.class);  
       intent.putExtra(Constants.IMAGES, Constants.images);  
       startActivity(intent);  
    }  
```

运行效果  
![这里写图片描述](http://img.blog.csdn.net/20160402110145154)

c、GrideView实现加载图片   

暂时空缺      


### 三、提示和技巧    

1. 只有在你需要让Image的尺寸比当前设备的尺寸大的时候，你才需要配置maxImageWidthForMemoryCach(...)和      

maxImageHeightForMemoryCache(...)这两个参数，比如放大图片的时候。其他情况下，不需要做这些配置，因为默认的配置会根据屏幕尺寸以最节约内存的方式处理Bitmap。         

2. 在设置中配置线程池的大小是非常明智的。一个大的线程池会允许多条线程同时工作，但是也会显著的影响到UI线程的速度。但是可以通过设置一个较低的优先级来解决：当ImageLoader在使用的时候，可以降低它的优先级，这样UI线程会更加流畅。在使用List的时候，UI 线程经常会不太流畅，所以在你的程序中最好设置threadPoolSize(...)和threadPriority(...)这两个参数来优化你的应用。

3. memoryCache(...)和memoryCacheSize(...)这两个参数会互相覆盖，所以在ImageLoaderConfiguration中使用一个就好了

4. diskCacheSize(...)、diskCache(...)和diskCacheFileCount(...)这三个参数会互相覆盖，只使用一个

注：不要使用discCacheSize(...)、discCache(...)和discCacheFileCount(...)这三个参数已经弃用

5. 如果你的程序中使用displayImage（）方法时传入的参数经常是一样的，那么一个合理的解决方法是，把这些选项

配置在ImageLoader的设置中作为默认的选项（通过调用defaultDisplayImageOptions(...)方法）。之后调用

displayImage(...)方法的时候就不必再指定这些选项了，如果这些选项没有明确的指定给

defaultDisplayImageOptions(...)方法，那调用的时候将会调用UIL的默认设置。



### 四、注意事项

1. 如果你经常出现oom，你可以尝试:

  1)禁用在内存中缓存cacheInMemory(false)，如果oom仍然发生那么似乎你的应用程序有内存泄漏，使用MemoryAnalyzer来检测它。否则尝试以下步骤(尝试所有或几个)

  2)减少配置的线程池的大小(.threadPoolSize(...))，建议1~5

  3)在显示选项中使用 .bitmapConfig(Bitmap.Config.RGB_565) . RGB_565模式消耗的内存比ARGB_8888模式少两倍.

  4)配置中使用.diskCacheExtraOptions(480, 320, null)

  5)配置中使用 .memoryCache(newWeakMemoryCache()) 或者完全禁用在内存中缓存(don't call .cacheInMemory()).

  6)在显示选项中使用.imageScaleType(ImageScaleType.EXACTLY) 或 .imageScaleType(ImageScaleType.IN_SAMPLE_INT)

  7)避免使用 RoundedBitmapDisplayer. 调用的时候它使用ARGB-8888模式创建了一个新的Bitmap对象来显示，对于内存缓存模式 (ImageLoaderConfiguration.memoryCache(...)) 你可以使用已经实现好的方法.

2. ImageLoader是根据ImageView的height，width确定图片的宽高

3. 一定要对ImageLoaderConfiguration进行初始化,否则会报错

4. 开启缓存后默认会缓存到外置SD卡如下地址(/sdcard/Android/data/[package_name]/cache).如果外置SD卡不存在，会缓存到手机. 缓存到Sd卡需要在AndroidManifest.xml文件中进行如下配置

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
``` 

5. 内存缓存模式可以使用以下已实现的方法 (ImageLoaderConfiguration.memoryCache(...))

1)缓存只使用强引用

LruMemoryCache (缓存大小超过指定值时，删除最近最少使用的bitmap)  --默认情况下使用

2)缓存使用弱引用和强引用

```
UsingFreqLimitedMemoryCache (缓存大小超过指定值时,删除最少使的bitmap)
LRULimitedMemoryCache (缓存大小超过指定值时,删除最近最少使用的<span style="font-family: 'Helvetica Neue', Helvetica, 'Segoe UI', Arial, freesans, sans-serif;">bitmap) --默认值</span>
FIFOLimitedMemoryCache (缓存大小超过指定值时,按先进先出规则删除的<span style="font-family: 'Helvetica Neue', Helvetica, 'Segoe UI', Arial, freesans, sans-serif;">bitmap)</span>
LargestLimitedMemoryCache (缓存大小超过指定值时,删除最大的bitmap)
LimitedAgeMemoryCache (缓存对象超过定义的时间后删除)
```
3)缓存使用弱引用

```
WeakMemoryCache（没有限制缓存）
```  

6. 本地缓存模式可以使用以下已实现的方法 (ImageLoaderConfiguration.diskCache(...)) 

UnlimitedDiskCache   不限制缓存大小（默认）  
TotalSizeLimitedDiskCache (设置总缓存大小，超过时删除最久之前的缓存)  
FileCountLimitedDiskCache (设置总缓存文件数量，当到达警戒值时，删除最久之前的缓存。如果文件的大小都一样的时候，可以使用该模式)     
LimitedAgeDiskCache (不限制缓存大小，但是设置缓存时间，到期后删除)           

### 注意，使用ImageLoader之间一定要初始化，最简单的方式就是在Application中加入  

```
ImageLoader.getInstance().init(ImageLoaderConfiguration.createDefault(CustomApplication.this));
```
### 参考链接:  
[Android开源框架Universal-Image-Loader详解 - android](http://www.mincoder.com/article/3800.shtml)  

[Android UI-开源框架ImageLoader的完美例子 - 巫_1曲待续 - 博客频道 - CSDN.NET](http://blog.csdn.net/wwj_748/article/details/10079311)

### 完成













