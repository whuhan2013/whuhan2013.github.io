---
layout: post
title: Android缓存学习入门(二)
date: 2016-6-16
categories: blog
tags: [android]
description: Android缓存学习入门
---


**本文主要包括以下内容**  

1. 内存缓存策略   
2. 文件缓存策略  


### 内存缓存策略    

当有一个图片要去从网络下载的时候，我们并不会直接去从网络下载，因为在这个时代，用户的流量是宝贵的，耗流量的应用是不会得到用户的青睐的。那我们该怎么办呢？这样，我们会先从内存缓存中去查找是否有该图片，如果没有就去文件缓存中查找是否有该图片，如果还没有，我们就从网络下载图片。本博文的侧重点是如何做内存缓存，内存缓存的查找策略是：先从强引用缓存中查找，如果没有再从软引用缓存中查找，如果在软引用缓存中找到了，就把它移入强引用缓存；如果强引用缓存满了，就会根据Lru算法把某些图片移入软引用缓存，如果软引用缓存也满了，最早的软引用就会被删除。这里，我有必要说明下几个概念：强引用、软引用、弱引用、Lru。 

- 强引用：就是直接引用一个对象，一般的对象引用均是强引用
- 软引用：引用一个对象，当内存不足并且除了我们的引用之外没有其他地方引用此对象的情况 下，该对象会被gc回收
- 弱引用：引用一个对象，当除了我们的引用之外没有其他地方引用此对象的情况下，只要gc被调用，它就会被回收（请注意它和软引用的区别）
- Lru：Least Recently Used 近期最少使用算法，是一种页面置换算法，其思想是在缓存的页面数目固定的情况下，那些最近使用次数最少的页面将被移出，对于我们的内存缓存来说，强引用缓存大小固定为4M，如果当缓存的图片大于4M的时候，有些图片就会被从强引用缓存中删除，哪些图片会被删除呢，就是那些近期使用次数最少的图片。


**注意**  

In the past, a popular memory cache implementation was a SoftReference or WeakReference bitmap cache, however this is not recommended. Starting from Android 2.3 (API Level 9) the garbage collector is more aggressive with collecting soft/weak references which makes them fairly ineffective. In addition, prior to Android 3.0 (API Level 11), the backing data of a bitmap was stored in native memory which is not released in a predictable manner, potentially causing an application to briefly exceed its memory limits and crash.


在android2.3之后，软引用与弱引用已经不可靠了，所以慎用。  


```
public class ImageMemoryCache {
    /**
     * 从内存读取数据速度是最快的，为了更大限度使用内存，这里使用了两层缓存。
     *  强引用缓存不会轻易被回收，用来保存常用数据，不常用的转入软引用缓存。
     */
    private static final String TAG = "ImageMemoryCache";

    private static LruCache<String, Bitmap> mLruCache; // 强引用缓存

    private static LinkedHashMap<String, SoftReference<Bitmap>> mSoftCache; // 软引用缓存

    private static final int LRU_CACHE_SIZE = 4 * 1024 * 1024; // 强引用缓存容量：4MB

    private static final int SOFT_CACHE_NUM = 20; // 软引用缓存个数

    // 在这里分别初始化强引用缓存和弱引用缓存
    public ImageMemoryCache() {
        mLruCache = new LruCache<String, Bitmap>(LRU_CACHE_SIZE) {
            @Override
            // sizeOf返回为单个hashmap value的大小
            protected int sizeOf(String key, Bitmap value) {
                if (value != null)
                    return value.getRowBytes() * value.getHeight();
                else
                    return 0;
            }

            @Override
            protected void entryRemoved(boolean evicted, String key,
                    Bitmap oldValue, Bitmap newValue) {
                if (oldValue != null) {
                    // 强引用缓存容量满的时候，会根据LRU算法把最近没有被使用的图片转入此软引用缓存
                    Logger.d(TAG, "LruCache is full,move to SoftRefernceCache");
                    mSoftCache.put(key, new SoftReference<Bitmap>(oldValue));
                }
            }
        };

        mSoftCache = new LinkedHashMap<String, SoftReference<Bitmap>>(
                SOFT_CACHE_NUM, 0.75f, true) {
            private static final long serialVersionUID = 1L;

            /**
             * 当软引用数量大于20的时候，最旧的软引用将会被从链式哈希表中移出
             */
            @Override
            protected boolean removeEldestEntry(
                    Entry<String, SoftReference<Bitmap>> eldest) {
                if (size() > SOFT_CACHE_NUM) {
                    Logger.d(TAG, "should remove the eldest from SoftReference");
                    return true;
                }
                return false;
            }
        };
    }

    /**
     * 从缓存中获取图片
     */
    public Bitmap getBitmapFromMemory(String url) {
        Bitmap bitmap;

        // 先从强引用缓存中获取
        synchronized (mLruCache) {
            bitmap = mLruCache.get(url);
            if (bitmap != null) {
                // 如果找到的话，把元素移到LinkedHashMap的最前面，从而保证在LRU算法中是最后被删除
                mLruCache.remove(url);
                mLruCache.put(url, bitmap);
                Logger.d(TAG, "get bmp from LruCache,url=" + url);
                return bitmap;
            }
        }

        // 如果强引用缓存中找不到，到软引用缓存中找，找到后就把它从软引用中移到强引用缓存中
        synchronized (mSoftCache) {
            SoftReference<Bitmap> bitmapReference = mSoftCache.get(url);
            if (bitmapReference != null) {
                bitmap = bitmapReference.get();
                if (bitmap != null) {
                    // 将图片移回LruCache
                    mLruCache.put(url, bitmap);
                    mSoftCache.remove(url);
                    Logger.d(TAG, "get bmp from SoftReferenceCache, url=" + url);
                    return bitmap;
                } else {
                    mSoftCache.remove(url);
                }
            }
        }
        return null;
    }

    /**
     * 添加图片到缓存
     */
    public void addBitmapToMemory(String url, Bitmap bitmap) {
        if (bitmap != null) {
            synchronized (mLruCache) {
                mLruCache.put(url, bitmap);
            }
        }
    }

    public void clearCache() {
        mSoftCache.clear();
    }
}
```


### 文件缓存策略 


当一张图片从网络下载成功以后，这个图片会被加入内存缓存和文件缓存，对于文件缓存来说，这张图片将被以url的哈希值加cach后缀名的形式存储在SD卡上，这样，当下一次再需要同一个url的图片的时候，就不需要从网络下载了，而是直接通过url来进行查找。同时一张图片被访问时，它的最后修改时间将被更新，这样的意义在于：当SD卡空间不足的时候，将会按照最后修改时间来删除40%缓存的图片，确切来说，那些修改时间比较早的图片将会被删除。


```
public class ImageFileCache
{
    private static final String TAG = "ImageFileCache";
    
    //图片缓存目录
    private static final String IMGCACHDIR = "/sdcard/ImgCach";
    
    //保存的cache文件宽展名
    private static final String CACHETAIL = ".cach";
                                                            
    private static final int MB = 1024*1024;
    
    private static final int CACHE_SIZE = 1;
    
    //当SD卡剩余空间小于10M的时候会清理缓存
    private static final int FREE_SD_SPACE_NEEDED_TO_CACHE = 10;
                                                                
    public ImageFileCache() 
    {
        //清理部分文件缓存
        removeCache(IMGCACHDIR);            
    }
                                                                
    /** 
     * 从缓存中获取图片 
     */
    public Bitmap getImageFromFile(final String url) 
    {    
        final String path = IMGCACHDIR + "/" + convertUrlToFileName(url);
        File file = new File(path);
        if (file != null && file.exists()) 
        {
            Bitmap bmp = BitmapFactory.decodeFile(path);
            if (bmp == null) 
            {
                file.delete();
            } 
            else 
            {
                updateFileTime(path);
                Logger.d(TAG, "get bmp from FileCache,url=" + url);
                return bmp;
            }
        }
        return null;
    }
                                                                
    /**
     * 将图片存入文件缓存 
     */
    public void saveBitmapToFile(Bitmap bm, String url) 
    {
        if (bm == null) {
            return;
        }
        //判断sdcard上的空间
        if (FREE_SD_SPACE_NEEDED_TO_CACHE > SdCardFreeSpace()) 
        {
            //SD空间不足
            return;
        }
        
        String filename = convertUrlToFileName(url);
        File dirFile = new File(IMGCACHDIR);
        if (!dirFile.exists())
            dirFile.mkdirs();
        File file = new File(IMGCACHDIR +"/" + filename);
        try 
        {
            file.createNewFile();
            OutputStream outStream = new FileOutputStream(file);
            bm.compress(Bitmap.CompressFormat.JPEG, 100, outStream);
            outStream.flush();
            outStream.close();
        } 
        catch (FileNotFoundException e) 
        {
            Logger.d(TAG, "FileNotFoundException");
        } 
        catch (IOException e) 
        {
            Logger.d(TAG, "IOException");
        }
    } 
                                                                
    /**
     * 计算存储目录下的文件大小，
     * 当文件总大小大于规定的CACHE_SIZE或者sdcard剩余空间小于FREE_SD_SPACE_NEEDED_TO_CACHE的规定
     * 那么删除40%最近没有被使用的文件
     */
    private boolean removeCache(String dirPath) 
    {
        File dir = new File(dirPath);
        File[] files = dir.listFiles();
        
        if (files == null) 
        {
            return true;
        }
        
        if (!android.os.Environment.getExternalStorageState().equals(android.os.Environment.MEDIA_MOUNTED))
        {
            return false;
        }
                                                            
        int dirSize = 0;
        for (int i = 0; i < files.length; i++) 
        {
            if (files[i].getName().contains(CACHETAIL)) 
            {
                dirSize += files[i].length();
            }
        }
                                                            
        if (dirSize > CACHE_SIZE * MB || FREE_SD_SPACE_NEEDED_TO_CACHE > SdCardFreeSpace()) 
        {
            int removeFactor = (int) (0.4 * files.length);
            Arrays.sort(files, new FileLastModifSort());
            for (int i = 0; i < removeFactor; i++) 
            {
                if (files[i].getName().contains(CACHETAIL)) 
                {
                    files[i].delete();
                }
            }
        }
                                                            
        if (SdCardFreeSpace() <= CACHE_SIZE) 
        {
            return false;
        }
                                                                    
        return true;
    }
                                                                
    /**
     * 修改文件的最后修改时间
     */
    public void updateFileTime(String path) 
    {
        File file = new File(path);
        long newModifiedTime = System.currentTimeMillis();
        file.setLastModified(newModifiedTime);
    }
                                                                
    /** 
     * 计算SD卡上的剩余空间 
     */
    private int SdCardFreeSpace()
    {
        StatFs stat = new StatFs(Environment.getExternalStorageDirectory().getPath());
        double sdFreeMB = ((double)stat.getAvailableBlocks() * (double) stat.getBlockSize()) / MB;
        return (int) sdFreeMB;
    } 
                                                                
    /** 
     * 将url转成文件名 
     */
    private String convertUrlToFileName(String url)
    {
        return url.hashCode() + CACHETAIL;
    }
                                                                
    /**
     * 根据文件的最后修改时间进行排序
     */
    private class FileLastModifSort implements Comparator<File> 
    {
        public int compare(File file0, File file1) 
        {
            if (file0.lastModified() > file1.lastModified())
            {
                return 1;
            } 
            else if (file0.lastModified() == file1.lastModified()) 
            {
                return 0;
            } 
            else 
            {
                return -1;
            }
        }
    }

}
```


**References**   

[android中图片的三级cache策略（内存、文件、网络）之二：内存缓存策略](http://blog.csdn.net/singwhatiwanna/article/details/17566439)

[android中图片的三级cache策略（内存、文件、网络）之三：文件缓存策略](http://blog.csdn.net/singwhatiwanna/article/details/17588159)














