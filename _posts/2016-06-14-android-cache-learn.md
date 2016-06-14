---
layout: post
title: Android缓存学习入门
date: 2016-6-14
categories: blog
tags: [android]
description: Android缓存学习入门
---


**本文主要包括以下内容** 

1. 利用LruCache实现内存缓存    
2. 利用DiskLruCache实现磁盘缓存    
3. LruCache与DiskLruCache结合实例      
4. 利用了缓存机制的瀑布流实例   


### 内存缓存的实现  

```
public class PhotoWallAdapter extends ArrayAdapter<String> implements OnScrollListener {

    /**
     * 记录所有正在下载或等待下载的任务。
     */
    private Set<BitmapWorkerTask> taskCollection;

    /**
     * 图片缓存技术的核心类，用于缓存所有下载好的图片，在程序内存达到设定值时会将最少最近使用的图片移除掉。
     */
    private LruCache<String, Bitmap> mMemoryCache;

    /**
     * GridView的实例
     */
    private GridView mPhotoWall;

    /**
     * 第一张可见图片的下标
     */
    private int mFirstVisibleItem;

    /**
     * 一屏有多少张图片可见
     */
    private int mVisibleItemCount;

    /**
     * 记录是否刚打开程序，用于解决进入程序不滚动屏幕，不会下载图片的问题。
     */
    private boolean isFirstEnter = true;

    public PhotoWallAdapter(Context context, int textViewResourceId, String[] objects,
            GridView photoWall) {
        super(context, textViewResourceId, objects);
        mPhotoWall = photoWall;
        taskCollection = new HashSet<BitmapWorkerTask>();
        // 获取应用程序最大可用内存
        int maxMemory = (int) Runtime.getRuntime().maxMemory();
        int cacheSize = maxMemory / 8;
        // 设置图片缓存大小为程序最大可用内存的1/8
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            @Override
            protected int sizeOf(String key, Bitmap bitmap) {
                return bitmap.getByteCount();
            }
        };
        mPhotoWall.setOnScrollListener(this);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        final String url = getItem(position);
        View view;
        if (convertView == null) {
            view = LayoutInflater.from(getContext()).inflate(R.layout.photo_layout, null);
        } else {
            view = convertView;
        }
        final ImageView photo = (ImageView) view.findViewById(R.id.photo);
        // 给ImageView设置一个Tag，保证异步加载图片时不会乱序
        photo.setTag(url);
        setImageView(url, photo);
        return view;
    }

    /**
     * 给ImageView设置图片。首先从LruCache中取出图片的缓存，设置到ImageView上。如果LruCache中没有该图片的缓存，
     * 就给ImageView设置一张默认图片。
     * 
     * @param imageUrl
     *            图片的URL地址，用于作为LruCache的键。
     * @param imageView
     *            用于显示图片的控件。
     */
    private void setImageView(String imageUrl, ImageView imageView) {
        Bitmap bitmap = getBitmapFromMemoryCache(imageUrl);
        if (bitmap != null) {
            imageView.setImageBitmap(bitmap);
        } else {
            imageView.setImageResource(R.drawable.empty_photo);
        }
    }

    /**
     * 将一张图片存储到LruCache中。
     * 
     * @param key
     *            LruCache的键，这里传入图片的URL地址。
     * @param bitmap
     *            LruCache的键，这里传入从网络上下载的Bitmap对象。
     */
    public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
        if (getBitmapFromMemoryCache(key) == null) {
            mMemoryCache.put(key, bitmap);
        }
    }

    /**
     * 从LruCache中获取一张图片，如果不存在就返回null。
     * 
     * @param key
     *            LruCache的键，这里传入图片的URL地址。
     * @return 对应传入键的Bitmap对象，或者null。
     */
    public Bitmap getBitmapFromMemoryCache(String key) {
        return mMemoryCache.get(key);
    }

    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        // 仅当GridView静止时才去下载图片，GridView滑动时取消所有正在下载的任务
        if (scrollState == SCROLL_STATE_IDLE) {
            loadBitmaps(mFirstVisibleItem, mVisibleItemCount);
        } else {
            cancelAllTasks();
        }
    }

    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount,
            int totalItemCount) {
        mFirstVisibleItem = firstVisibleItem;
        mVisibleItemCount = visibleItemCount;
        // 下载的任务应该由onScrollStateChanged里调用，但首次进入程序时onScrollStateChanged并不会调用，
        // 因此在这里为首次进入程序开启下载任务。
        if (isFirstEnter && visibleItemCount > 0) {
            loadBitmaps(firstVisibleItem, visibleItemCount);
            isFirstEnter = false;
        }
    }

    /**
     * 加载Bitmap对象。此方法会在LruCache中检查所有屏幕中可见的ImageView的Bitmap对象，
     * 如果发现任何一个ImageView的Bitmap对象不在缓存中，就会开启异步线程去下载图片。
     * 
     * @param firstVisibleItem
     *            第一个可见的ImageView的下标
     * @param visibleItemCount
     *            屏幕中总共可见的元素数
     */
    private void loadBitmaps(int firstVisibleItem, int visibleItemCount) {
        try {
            for (int i = firstVisibleItem; i < firstVisibleItem + visibleItemCount; i++) {
                String imageUrl = Images.imageThumbUrls[i];
                Bitmap bitmap = getBitmapFromMemoryCache(imageUrl);
                if (bitmap == null) {
                    BitmapWorkerTask task = new BitmapWorkerTask();
                    taskCollection.add(task);
                    task.execute(imageUrl);
                } else {
                    ImageView imageView = (ImageView) mPhotoWall.findViewWithTag(imageUrl);
                    if (imageView != null && bitmap != null) {
                        imageView.setImageBitmap(bitmap);
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 取消所有正在下载或等待下载的任务。
     */
    public void cancelAllTasks() {
        if (taskCollection != null) {
            for (BitmapWorkerTask task : taskCollection) {
                task.cancel(false);
            }
        }
    }

    /**
     * 异步下载图片的任务。
     * 
     * @author guolin
     */
    class BitmapWorkerTask extends AsyncTask<String, Void, Bitmap> {

        /**
         * 图片的URL地址
         */
        private String imageUrl;

        @Override
        protected Bitmap doInBackground(String... params) {
            imageUrl = params[0];
            // 在后台开始下载图片
            Bitmap bitmap = downloadBitmap(params[0]);
            if (bitmap != null) {
                // 图片下载完成后缓存到LrcCache中
                addBitmapToMemoryCache(params[0], bitmap);
            }
            return bitmap;
        }

        @Override
        protected void onPostExecute(Bitmap bitmap) {
            super.onPostExecute(bitmap);
            // 根据Tag找到相应的ImageView控件，将下载好的图片显示出来。
            ImageView imageView = (ImageView) mPhotoWall.findViewWithTag(imageUrl);
            if (imageView != null && bitmap != null) {
                imageView.setImageBitmap(bitmap);
            }
            taskCollection.remove(this);
        }

        /**
         * 建立HTTP请求，并获取Bitmap对象。
         * 
         * @param imageUrl
         *            图片的URL地址
         * @return 解析后的Bitmap对象
         */
        private Bitmap downloadBitmap(String imageUrl) {
            Bitmap bitmap = null;
            HttpURLConnection con = null;
            try {
                URL url = new URL(imageUrl);
                con = (HttpURLConnection) url.openConnection();
                con.setConnectTimeout(5 * 1000);
                con.setReadTimeout(10 * 1000);
                bitmap = BitmapFactory.decodeStream(con.getInputStream());
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (con != null) {
                    con.disconnect();
                }
            }
            return bitmap;
        }

    }

}
```

其中getView方法中的getItem方法是获取当前位置的数据的  

[android中Baseadapter的 getItem 跟 getItemId 的作用](http://www.csdn123.com/html/exception/667/667664_667666_667660.htm)


PhotoWallAdapter是整个照片墙程序中最关键的一个类了，这里我来重点给大家讲解一下。首先在PhotoWallAdapter的构造函数中，我们初始化了LruCache类，并设置了最大缓存容量为程序最大可用内存的1/8，接下来又为GridView注册了一个滚动监听器。然后在getView()方法中，我们为每个ImageView设置了一个唯一的Tag，这个Tag的作用是为了后面能够准确地找回这个ImageView，不然异步加载图片会出现乱序的情况。之后调用了setImageView()方法为ImageView设置一张图片，这个方法首先会从LruCache缓存中查找是否已经缓存了这张图片，如果成功找到则将缓存中的图片显示在ImageView上，否则就显示一张默认的空图片。
看了半天，那到底是在哪里下载图片的呢？这是在GridView的滚动监听器中进行的，在onScrollStateChanged()方法中，我们对GridView的滚动状态进行了判断，如果当前GridView是静止的，则调用loadBitmaps()方法去下载图片，如果GridView正在滚动，则取消掉所有下载任务，这样可以保证GridView滚动的流畅性。在loadBitmaps()方法中，我们为屏幕上所有可见的GridView子元素开启了一个线程去执行下载任务，下载成功后将图片存储到LruCache当中，然后通过Tag找到相应的ImageView控件，把下载好的图片显示出来。
由于我们使用了LruCache来缓存图片，所以不需要担心内存溢出的情况，当LruCache中存储图片的总大小达到容量上限的时候，会自动把最近最少使用的图片从缓存中移除。



### 硬盘缓存实现  

**主要包括以下几步**  

1、 下载DiskLruCache的源码。下载好了源码之后，只需要在项目中新建一个libcore.io包，然后将DiskLruCache.java文件复制到这个包中即可。


2、 打开缓存   

```
DiskLruCache mDiskLruCache = null;
try {
    File cacheDir = getDiskCacheDir(context, "bitmap");
    if (!cacheDir.exists()) {
        cacheDir.mkdirs();
    }
    mDiskLruCache = DiskLruCache.open(cacheDir, getAppVersion(context), 1, 10 * 1024 * 1024);
} catch (IOException e) {
    e.printStackTrace();
}
```


3、写入缓存  

```
new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";
            String key = hashKeyForDisk(imageUrl);
            DiskLruCache.Editor editor = mDiskLruCache.edit(key);
            if (editor != null) {
                OutputStream outputStream = editor.newOutputStream(0);
                if (downloadUrlToStream(imageUrl, outputStream)) {
                    editor.commit();
                } else {
                    editor.abort();
                }
            }
            mDiskLruCache.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}).start();




private boolean downloadUrlToStream(String urlString, OutputStream outputStream) {
    HttpURLConnection urlConnection = null;
    BufferedOutputStream out = null;
    BufferedInputStream in = null;
    try {
        final URL url = new URL(urlString);
        urlConnection = (HttpURLConnection) url.openConnection();
        in = new BufferedInputStream(urlConnection.getInputStream(), 8 * 1024);
        out = new BufferedOutputStream(outputStream, 8 * 1024);
        int b;
        while ((b = in.read()) != -1) {
            out.write(b);
        }
        return true;
    } catch (final IOException e) {
        e.printStackTrace();
    } finally {
        if (urlConnection != null) {
            urlConnection.disconnect();
        }
        try {
            if (out != null) {
                out.close();
            }
            if (in != null) {
                in.close();
            }
        } catch (final IOException e) {
            e.printStackTrace();
        }
    }
    return false;
}

```


4、读取缓存

```
try {
    String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";
    String key = hashKeyForDisk(imageUrl);
    DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);
    if (snapShot != null) {
        InputStream is = snapShot.getInputStream(0);
        Bitmap bitmap = BitmapFactory.decodeStream(is);
        mImage.setImageBitmap(bitmap);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```


5. 移除缓存

```
try {
    String imageUrl = "http://img.my.csdn.net/uploads/201309/01/1378037235_7476.jpg";  
    String key = hashKeyForDisk(imageUrl);  
    mDiskLruCache.remove(key);
} catch (IOException e) {
    e.printStackTrace();
}
```


### LruCache与DiskLruCache结合实例


```
public class PhotoWallAdapter extends ArrayAdapter<String> {

    /**
     * 记录所有正在下载或等待下载的任务。
     */
    private Set<BitmapWorkerTask> taskCollection;

    /**
     * 图片缓存技术的核心类，用于缓存所有下载好的图片，在程序内存达到设定值时会将最少最近使用的图片移除掉。
     */
    private LruCache<String, Bitmap> mMemoryCache;

    /**
     * 图片硬盘缓存核心类。
     */
    private DiskLruCache mDiskLruCache;

    /**
     * GridView的实例
     */
    private GridView mPhotoWall;

    /**
     * 记录每个子项的高度。
     */
    private int mItemHeight = 0;

    public PhotoWallAdapter(Context context, int textViewResourceId, String[] objects,
            GridView photoWall) {
        super(context, textViewResourceId, objects);
        mPhotoWall = photoWall;
        taskCollection = new HashSet<BitmapWorkerTask>();
        // 获取应用程序最大可用内存
        int maxMemory = (int) Runtime.getRuntime().maxMemory();
        int cacheSize = maxMemory / 8;
        // 设置图片缓存大小为程序最大可用内存的1/8
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            @Override
            protected int sizeOf(String key, Bitmap bitmap) {
                return bitmap.getByteCount();
            }
        };
        try {
            // 获取图片缓存路径
            File cacheDir = getDiskCacheDir(context, "thumb");
            if (!cacheDir.exists()) {
                cacheDir.mkdirs();
            }
            // 创建DiskLruCache实例，初始化缓存数据
            mDiskLruCache = DiskLruCache
                    .open(cacheDir, getAppVersion(context), 1, 10 * 1024 * 1024);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        final String url = getItem(position);
        View view;
        if (convertView == null) {
            view = LayoutInflater.from(getContext()).inflate(R.layout.photo_layout, null);
        } else {
            view = convertView;
        }
        final ImageView imageView = (ImageView) view.findViewById(R.id.photo);
        if (imageView.getLayoutParams().height != mItemHeight) {
            imageView.getLayoutParams().height = mItemHeight;
        }
        // 给ImageView设置一个Tag，保证异步加载图片时不会乱序
        imageView.setTag(url);
        imageView.setImageResource(R.drawable.empty_photo);
        loadBitmaps(imageView, url);
        return view;
    }

    /**
     * 将一张图片存储到LruCache中。
     * 
     * @param key
     *            LruCache的键，这里传入图片的URL地址。
     * @param bitmap
     *            LruCache的键，这里传入从网络上下载的Bitmap对象。
     */
    public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
        if (getBitmapFromMemoryCache(key) == null) {
            mMemoryCache.put(key, bitmap);
        }
    }

    /**
     * 从LruCache中获取一张图片，如果不存在就返回null。
     * 
     * @param key
     *            LruCache的键，这里传入图片的URL地址。
     * @return 对应传入键的Bitmap对象，或者null。
     */
    public Bitmap getBitmapFromMemoryCache(String key) {
        return mMemoryCache.get(key);
    }

    /**
     * 加载Bitmap对象。此方法会在LruCache中检查所有屏幕中可见的ImageView的Bitmap对象，
     * 如果发现任何一个ImageView的Bitmap对象不在缓存中，就会开启异步线程去下载图片。
     */
    public void loadBitmaps(ImageView imageView, String imageUrl) {
        try {
            Bitmap bitmap = getBitmapFromMemoryCache(imageUrl);
            if (bitmap == null) {
                BitmapWorkerTask task = new BitmapWorkerTask();
                taskCollection.add(task);
                task.execute(imageUrl);
            } else {
                if (imageView != null && bitmap != null) {
                    imageView.setImageBitmap(bitmap);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 取消所有正在下载或等待下载的任务。
     */
    public void cancelAllTasks() {
        if (taskCollection != null) {
            for (BitmapWorkerTask task : taskCollection) {
                task.cancel(false);
            }
        }
    }

    /**
     * 根据传入的uniqueName获取硬盘缓存的路径地址。
     */
    public File getDiskCacheDir(Context context, String uniqueName) {
        String cachePath;
        if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
                || !Environment.isExternalStorageRemovable()) {
            cachePath = context.getExternalCacheDir().getPath();
        } else {
            cachePath = context.getCacheDir().getPath();
        }
        return new File(cachePath + File.separator + uniqueName);
    }

    /**
     * 获取当前应用程序的版本号。
     */
    public int getAppVersion(Context context) {
        try {
            PackageInfo info = context.getPackageManager().getPackageInfo(context.getPackageName(),
                    0);
            return info.versionCode;
        } catch (NameNotFoundException e) {
            e.printStackTrace();
        }
        return 1;
    }

    /**
     * 设置item子项的高度。
     */
    public void setItemHeight(int height) {
        if (height == mItemHeight) {
            return;
        }
        mItemHeight = height;
        notifyDataSetChanged();
    }

    /**
     * 使用MD5算法对传入的key进行加密并返回。
     */
    public String hashKeyForDisk(String key) {
        String cacheKey;
        try {
            final MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(key.getBytes());
            cacheKey = bytesToHexString(mDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            cacheKey = String.valueOf(key.hashCode());
        }
        return cacheKey;
    }
    
    /**
     * 将缓存记录同步到journal文件中。
     */
    public void fluchCache() {
        if (mDiskLruCache != null) {
            try {
                mDiskLruCache.flush();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private String bytesToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }

    /**
     * 异步下载图片的任务。
     * 
     * @author guolin
     */
    class BitmapWorkerTask extends AsyncTask<String, Void, Bitmap> {

        /**
         * 图片的URL地址
         */
        private String imageUrl;

        @Override
        protected Bitmap doInBackground(String... params) {
            imageUrl = params[0];
            FileDescriptor fileDescriptor = null;
            FileInputStream fileInputStream = null;
            Snapshot snapShot = null;
            try {
                // 生成图片URL对应的key
                final String key = hashKeyForDisk(imageUrl);
                // 查找key对应的缓存
                snapShot = mDiskLruCache.get(key);
                if (snapShot == null) {
                    // 如果没有找到对应的缓存，则准备从网络上请求数据，并写入缓存
                    DiskLruCache.Editor editor = mDiskLruCache.edit(key);
                    if (editor != null) {
                        OutputStream outputStream = editor.newOutputStream(0);
                        if (downloadUrlToStream(imageUrl, outputStream)) {
                            editor.commit();
                        } else {
                            editor.abort();
                        }
                    }
                    // 缓存被写入后，再次查找key对应的缓存
                    snapShot = mDiskLruCache.get(key);
                }
                if (snapShot != null) {
                    fileInputStream = (FileInputStream) snapShot.getInputStream(0);
                    fileDescriptor = fileInputStream.getFD();
                }
                // 将缓存数据解析成Bitmap对象
                Bitmap bitmap = null;
                if (fileDescriptor != null) {
                    bitmap = BitmapFactory.decodeFileDescriptor(fileDescriptor);
                }
                if (bitmap != null) {
                    // 将Bitmap对象添加到内存缓存当中
                    addBitmapToMemoryCache(params[0], bitmap);
                }
                return bitmap;
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (fileDescriptor == null && fileInputStream != null) {
                    try {
                        fileInputStream.close();
                    } catch (IOException e) {
                    }
                }
            }
            return null;
        }

        @Override
        protected void onPostExecute(Bitmap bitmap) {
            super.onPostExecute(bitmap);
            // 根据Tag找到相应的ImageView控件，将下载好的图片显示出来。
            ImageView imageView = (ImageView) mPhotoWall.findViewWithTag(imageUrl);
            if (imageView != null && bitmap != null) {
                imageView.setImageBitmap(bitmap);
            }
            taskCollection.remove(this);
        }

        /**
         * 建立HTTP请求，并获取Bitmap对象。
         * 
         * @param imageUrl
         *            图片的URL地址
         * @return 解析后的Bitmap对象
         */
        private boolean downloadUrlToStream(String urlString, OutputStream outputStream) {
            HttpURLConnection urlConnection = null;
            BufferedOutputStream out = null;
            BufferedInputStream in = null;
            try {
                final URL url = new URL(urlString);
                urlConnection = (HttpURLConnection) url.openConnection();
                in = new BufferedInputStream(urlConnection.getInputStream(), 8 * 1024);
                out = new BufferedOutputStream(outputStream, 8 * 1024);
                int b;
                while ((b = in.read()) != -1) {
                    out.write(b);
                }
                return true;
            } catch (final IOException e) {
                e.printStackTrace();
            } finally {
                if (urlConnection != null) {
                    urlConnection.disconnect();
                }
                try {
                    if (out != null) {
                        out.close();
                    }
                    if (in != null) {
                        in.close();
                    }
                } catch (final IOException e) {
                    e.printStackTrace();
                }
            }
            return false;
        }

    }

}
```




代码有点长，我们一点点进行分析。首先在PhotoWallAdapter的构造函数中，我们初始化了LruCache类，并设置了内存缓存容量为程序最大可用内存的1/8，紧接着调用了DiskLruCache的open()方法来创建实例，并设置了硬盘缓存容量为10M，这样我们就把LruCache和DiskLruCache的初始化工作完成了。
接着在getView()方法中，我们为每个ImageView设置了一个唯一的Tag，这个Tag的作用是为了后面能够准确地找回这个ImageView，不然异步加载图片会出现乱序的情况。然后在getView()方法的最后调用了loadBitmaps()方法，加载图片的具体逻辑也就是在这里执行的了。          

进入到loadBitmaps()方法中可以看到，实现是调用了getBitmapFromMemoryCache()方法来从内存中获取缓存，如果获取到了则直接调用ImageView的setImageBitmap()方法将图片显示到界面上。如果内存中没有获取到，则开启一个BitmapWorkerTask任务来去异步加载图片。
那么在BitmapWorkerTask的doInBackground()方法中，我们就灵活运用了上篇文章中学习的DiskLruCache的各种用法。首先根据图片的URL生成对应的MD5 key，然后调用DiskLruCache的get()方法来获取硬盘缓存，如果没有获取到的话则从网络上请求图片并写入硬盘缓存，接着将Bitmap对象解析出来并添加到内存缓存当中，最后将这个Bitmap对象显示到界面上，这样一个完整的流程就执行完了。
那么我们再来分析一下上述流程，每次加载图片的时候都优先去内存缓存当中读取，当读取不到的时候则回去硬盘缓存中读取，而如果硬盘缓存仍然读取不到的话，就从网络上请求原始数据。不管是从硬盘缓存还是从网络获取，读取到了数据之后都应该添加到内存缓存当中，这样的话我们下次再去读取图片的时候就能迅速从内存当中读取到，而如果该图片从内存中被移除了的话，那就重复再执行一遍上述流程就可以了。
这样我们就把LruCache和DiskLruCache完美结合到一起了。


**效果如下** 

![](http://img.blog.csdn.net/20140812151529218)

### 瀑布流效果 

![](http://img.blog.csdn.net/20130904201515703)


### 源码下载 

[内存缓存](http://download.csdn.net/detail/sinyu890807/5848269)

[内存与硬盘结合](http://download.csdn.net/detail/sinyu890807/7754669)

[瀑布流](http://download.csdn.net/detail/sinyu890807/6218757)


### 参考链接

[Android照片墙应用实现，再多的图片也不怕崩溃 - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/9526203)

[Android DiskLruCache完全解析，硬盘缓存的最佳方案 - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/28863651)

[Android照片墙完整版，完美结合LruCache和DiskLruCache - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/34093441)

[Android瀑布流照片墙实现，体验不规则排列的美感 - 郭霖的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/guolin_blog/article/details/10470797)


















