---
layout: post
title: AsyncTask学习
date: 2016-6-20
categories: blog
tags: [android]
description: AsyncTask学习
---

1、对于耗时的操作，我们的一般方法是开启“子线程”。如果需要更新UI，则需要使用handler
 
2、如果耗时的操作太多，那么我们需要开启太多的子线程，这就会给系统带来巨大的负担，随之也会带来性能方面的问题。在这种情况下我们就可以考虑使用类AsyncTask来异步执行任务，不需要子线程和handler，就可以完成异步操作和刷新UI。
 
3、AsyncTask：对线程间的通讯做了包装，是后台线程和UI线程可以简易通讯：后台线程执行异步任务，将result告知UI线程。
 
4、使用方法：共分为两步，自定义AsyncTask，在耗时的地方调用自定义的AsyncTask。可以参照以下代码示例。
 
- step1：继承AsyncTask<Params,Progress,Result>
	+ Params:输入参数。对应的是调用自定义的AsyncTask的类中调用excute()方法中传递的参数。如果不需要传递参数，则直接设为Void即可。
	+ Progress：子线程执行的百分比
	+ Result：返回值类型。和doInBackground（）方法的返回值类型保持一致。
 
- step2：实现以下几个方法：执行时机和作用看示例代码，以下对返回值类型和参数进行说明
	+ onPreExecute()：无返回值类型。不传参数
	+ doInBackground(Params... params)：返回值类型和Result保持一致。参数：若无就传递Void；若有，就可用Params
	+ publishProgress(Params... params)：在执行此方法的时候会直接调用onProgressUpdate(Params... values)
	+ onProgressUpdate(Params... values)：无返回值类型。参数：若无就传递Void；若有，就可用Progress
	+ onPostExecute(Result  result) ：无返回值类型。参数：和Result保持一致。
 
- step3：在调用自定义的AsyncTask类中生成对象；
	+ 执行  ：对象.excute(Params... params);
 
**小注：**
 
1) Task的实例必须在UI thread中创建
 
 
2) execute方法必须在UI thread中调用
 
3) 不要手动的调用onPreExecute(), onPostExecute(Result)，doInBackground=\'#\'" onProgressUpdate(Progress...)这几个方法
 
4) 该task只能被执行一次，否则多次调用时将会出现异常
 
 
 
示例代码：


```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    >
<TextView    
       android:layout_width="fill_parent"   
       android:layout_height="wrap_content"   
       android:text="Hello , Welcome to Andy's Blog!"/>  
    <Button  
       android:id="@+id/download"  
       android:layout_width="fill_parent"  
       android:layout_height="wrap_content"  
       android:text="Download"/>  
    <TextView    
       android:id="@+id/tv"  
       android:layout_width="fill_parent"   
       android:layout_height="wrap_content"   
       android:text="当前进度显示"/>  
    <ProgressBar  
       android:id="@+id/pb"  
       android:layout_width="fill_parent"  
       android:layout_height="wrap_content"  
       style="?android:attr/progressBarStyleHorizontal"/>  
</LinearLayout>
```


```
package sn.demo;

import android.content.Context;
import android.os.AsyncTask;
import android.util.Log;
import android.widget.ProgressBar;
import android.widget.TextView;

public class DownloadTask extends AsyncTask<Integer, Integer, String> {
    //后面尖括号内分别是参数（线程休息时间），进度(publishProgress用到)，返回值 类型  
    
    private Context mContext=null;
    private ProgressBar mProgressBar=null;
    private TextView mTextView=null;
    public DownloadTask(Context context,ProgressBar pb,TextView tv){
        this.mContext=context;
        this.mProgressBar=pb;
        this.mTextView=tv;
    }
    /*
     * 第一个执行的方法
     * 执行时机：在执行实际的后台操作前，被UI 线程调用
     * 作用：可以在该方法中做一些准备工作，如在界面上显示一个进度条，或者一些控件的实例化，这个方法可以不用实现。
     * @see android.os.AsyncTask#onPreExecute()
     */
    @Override
    protected void onPreExecute() {
        // TODO Auto-generated method stub
        Log.d("sn", "00000");
        super.onPreExecute();
    }
    
    /*
     * 执行时机：在onPreExecute 方法执行后马上执行，该方法运行在后台线程中
     * 作用：主要负责执行那些很耗时的后台处理工作。可以调用 publishProgress方法来更新实时的任务进度。该方法是抽象方法，子类必须实现。
     * @see android.os.AsyncTask#doInBackground(Params[])
     */
    @Override
    protected String doInBackground(Integer... params) {
        // TODO Auto-generated method stub
        Log.d("sn", "1111111");
        for(int i=0;i<=100;i++){
           
            publishProgress(i);
            try {
                Thread.sleep(params[0]);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        return "执行完毕";
    }

    /*
     * 执行时机：这个函数在doInBackground调用publishProgress时被调用后，UI 线程将调用这个方法.虽然此方法只有一个参数,但此参数是一个数组，可以用values[i]来调用
     * 作用：在界面上展示任务的进展情况，例如通过一个进度条进行展示。此实例中，该方法会被执行100次
     * @see android.os.AsyncTask#onProgressUpdate(Progress[])
     */
    @Override
    protected void onProgressUpdate(Integer... values) {
        // TODO Auto-generated method stub
        Log.d("sn", "2222222222");
         mProgressBar.setProgress(values[0]);
        mTextView.setText(values[0]+"%");
        super.onProgressUpdate(values);
    }
    
    /*
     * 执行时机：在doInBackground 执行完成后，将被UI 线程调用
     * 作用：后台的计算结果将通过该方法传递到UI 线程，并且在界面上展示给用户
     * result:上面doInBackground执行后的返回值，所以这里是"执行完毕"  
     * @see android.os.AsyncTask#onPostExecute(java.lang.Object)
     */
    @Override
    protected void onPostExecute(String result) {
        // TODO Auto-generated method stub
        Log.d("sn", "3333333333");
        
        super.onPostExecute(result);
    }




}
```

```
package sn.demo;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.ProgressBar;
import android.widget.TextView;

public class AsyncTaskDemoActivity extends Activity {
    /** Called when the activity is first created. */
    private Button download;
    private TextView tv;
    private ProgressBar pb;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        
        initView();
    }
    private void initView() {
        // TODO Auto-generated method stub
        tv=(TextView)findViewById(R.id.tv);
        pb=(ProgressBar)findViewById(R.id.pb);
        download=(Button)findViewById(R.id.download);
        download.setOnClickListener(new OnClickListener() {
            
            @Override
            public void onClick(View v) {
                // TODO Auto-generated method stub
                DownloadTask dt=new DownloadTask(AsyncTaskDemoActivity.this,pb,tv);
                dt.execute(100);
            }
        });
    }
}
```


**参考链接** 

[五十、AsyncTask的使用方法和理解 - gentle_girl - 博客园](http://www.cnblogs.com/suinuaner/archive/2013/04/11/android_fifty.html)

[android AsyncTask介绍 - Devin Zhang - 博客园](http://www.cnblogs.com/devinzhang/archive/2012/02/13/2350070.html)


### Android面试系列之AsyncTask

**面试题：在项目中使用AsyncTask会有什么问题吗?**   

那我们考查AsyncTask会问些什么呢？得先问问会不会用吧，看看知不知道有onProgressUpdate方法。

**其次问一下是怎么理解AsyncTask的机制，有没有看过它的源代码？** 

这个问题主要看对方是否对Android的东西有好奇心，会主动去看AsyncTask的源码，或者能大体地讲清楚AsyncTask的原理。一般有好奇心的同学都比较善长学习，善长学习的人都能比较快融入团队。

```
AnsycTask执行任务时，内部会创建一个进程作用域的线程池来管理要运行的任务，也就就是说当你调用了AsyncTask.execute()后，AsyncTask会把任务交给线程池，由线程池来管理创建Thread和运行Therad。最后和UI打交道就交给Handler去处理了。
```

我们在实际的项目中，还需要关注一些问题：

```
线程池可以同时执行多少个TASK？
```

Android 3.0之前（1.6之前的版本不再关注）规定线程池的核心线程数为5个（corePoolSize），线程池总大小为128（maximumPoolSize），还有一个缓冲队列（sWorkQueue，缓冲队列可以放10个任务），当我们尝试去添加第139个任务时，程序就会崩溃。当线程池中的数量大于corePoolSize，缓冲队列已满，并且线程池中的数量小于maximumPoolSize，将会创建新的线程来处理被添加的任务。如下图会出现第16个Task比第6－15个Task先执行的情况。

**多个AsyncTask任务是串行还是并行？**  

从Android 1.6到2.3(Gingerbread) AsyncTask是并行的，即上面我们提到的有5个核心线程的线程池（ThreadPoolExecutor）负责调度任务。从Android 3.0开始，Android团队又把AsyncTask改成了串行，默认的Executor被指定为SERIAL_EXECUTOR。

```
    /**
     * An {@link Executor} that executes tasks one at a time in serial
     * order.  This serialization is global to a particular process.
     */
    public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
```

从它的说明也可以看出是串行的。如需要并行，可以通过设置executeOnExecutor(Executor)来实现多个AsyncTask并行。

```
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
```

**AsyncTask容易引发的Activity内存泄露**

如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对创建了AsyncTask的Activity的引用。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将继续在内存里保留这个引用，导致Activity无法被回收，引起内存泄露。

当然，最后少不了问一句：“你在项目中，会用什么方案来替换AsyncTask呢？”

**小结**   

感觉对AsyncTask的使用有点“成也萧何败萧何”的味道，它简单的解决了UI和后台线程交互的问题，但如果忽视它的限制（缺陷）和各版本不一致的线程池方式，可能会达不到预想的效果。最后发现没有使用过AsyncTask会被鄙视，如果你在实际项目中使用了AsyncTask也会被鄙视。不过，学习它对理解Android的机制和线程池的使用还是很的意义的，所以强烈建议大家读一下它的源码。


**参考链接** 

[Android面试系列之AsyncTask](http://mp.weixin.qq.com/s?__biz=MjM5NDkxMTgyNw==&mid=2653057600&idx=1&sn=006f4592cb3286b2b58bde484520b649&scene=0)