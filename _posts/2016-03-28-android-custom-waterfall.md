---
layout: post
title: 自定义控件之瀑布流与水波纹实现
date: 2016-3-28
categories: blog
tags: [android]
description: 自定义控件之瀑布流与水波纹实现
---

### 本文主要讲述了利用android自定义控件实现瀑布流与水波纹效果

### 首先为实现效果，应了解touch事件在android中的传递机制
![这里写图片描述](http://img.blog.csdn.net/20160328204531512)

 ![这里写图片描述](http://img.blog.csdn.net/20160328204550137)

### 在执行touch事件时

1. 首先执行dispatchTouchEvent方法，执行事件分发。   

2.  再执行onInterceptTouchEvent方法，判断是否中断事件，返回true时中断，执行自己的onTouchEvnet方法.

3. 最后执行onTouchEvent方法，处理事件

### 瀑布流实现

### 首先定义三个listView竖起排列，并且配置适配器

```
public class MainActivity extends Activity {

  private ListView lv1;
  private ListView lv2;
  private ListView lv3;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    lv1 = (ListView) findViewById(R.id.lv1);
    lv2 = (ListView) findViewById(R.id.lv2);
    lv3 = (ListView) findViewById(R.id.lv3);

    try {
      lv1.setAdapter(new MyAdapter1());
      lv2.setAdapter(new MyAdapter1());
      lv3.setAdapter(new MyAdapter1());
    } catch (Exception e) {
      e.printStackTrace();
    }

  }

  private int ids[] = new int[] { R.drawable.default1, R.drawable.girl1,
      R.drawable.girl2, R.drawable.girl3 };

  class MyAdapter1 extends BaseAdapter {

    @Override
    public int getCount() {
      return 3000;
    }

    @Override
    public Object getItem(int position) {
      return position;
    }

    @Override
    public long getItemId(int position) {
      return 0;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {

      ImageView iv = (ImageView) View.inflate(getApplicationContext(),
          R.layout.lv_item, null);
      int resId = (int) (Math.random() * 4);
      iv.setImageResource(ids[resId]);
      return iv;
    }
  }
}

```

### 布局文件的配置  

```
<com.example.pinterestlistview.MyLinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/mll"
    tools:context=".MainActivity" >

    <ListView
        android:id="@+id/lv2"
        android:scrollbars="none"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1" />

    <ListView
        android:id="@+id/lv1"
        android:scrollbars="none"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1" />

    <ListView
        android:id="@+id/lv3"
        android:scrollbars="none"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1" />

</com.example.pinterestlistview.MyLinearLayout>    
```     

### listView布局文件，注意要设置图片adjustViewBounds=true,并且图片应该放大drawable文件夹下，而不是drawable-hdpi下，不然的话会参差不齐

```
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/iv"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:adjustViewBounds="true"
    android:src="@drawable/lvfood1" />
```

### 自定义类截取事件分发

```
public class MyLinearLayout extends LinearLayout {


  public MyLinearLayout(Context context, AttributeSet attrs) {
    super(context, attrs);
  }

  @Override
  public boolean onInterceptTouchEvent(MotionEvent ev) {
    return true;
  }

  @Override
  public boolean dispatchTouchEvent(MotionEvent ev) {
    return super.dispatchTouchEvent(ev);
  }
  
  @Override
  public boolean onTouchEvent(MotionEvent event) {
    
    int width=getWidth()/getChildCount();
    int height = getHeight();
    int count=getChildCount();
    
    float eventX = event.getX();
    
    if (eventX<width){  // 滑动左边的 listView
      event.setLocation(width/2, event.getY());
      getChildAt(0).dispatchTouchEvent(event);
      return true;
      
    } else if (eventX > width && eventX < 2 * width) { //滑动中间的 listView  
      float eventY = event.getY();
      if (eventY < height / 2) {
        event.setLocation(width / 2, event.getY());
        for (int i = 0; i < count; i++) {
          View child = getChildAt(i);
          try {
            child.dispatchTouchEvent(event);
          } catch (Exception e) {
            e.printStackTrace();
          }
          
        }
        return true;
      } else if (eventY > height / 2) {
        event.setLocation(width / 2, event.getY());
        try {
          getChildAt(1).dispatchTouchEvent(event);
        } catch (Exception e) {
          e.printStackTrace();
        }
        return true;
      }
    }else if (eventX>2*width){
      event.setLocation(width/2, event.getY());
      getChildAt(2).dispatchTouchEvent(event);
      return true;
    }
    
    return true;
  }
  
}
```   

### 完成，运行效果如下   

![如下](http://img.blog.csdn.net/20160328205445813)

### 水波纹效果实现  

### 要实现水波纹效果，只需用一个自定义控件填充整个Activity即可

```
  <com.zj.wave.MyWave
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        />
```

### 自定义控件实现

```
/**
 * 水波纹效果
 * @author leo
 *
 */
public class MyRingWave extends View{

  /**
   * 二个相临波浪中心点的最小距离
   */
  private static final int DIS_SOLP = 13;

  protected boolean isRunning = false;

  private ArrayList<Wave> wList;

  public MyRingWave(Context context, AttributeSet attrs) {
    super(context, attrs);
    wList = new ArrayList<MyRingWave.Wave>();
  }
  
  private Handler handler = new Handler(){
    public void handleMessage(android.os.Message msg) {

      //刷新数据
      flushData();
      //刷新页面
      invalidate();
      //循环动画
      if (isRunning) {
        handler.sendEmptyMessageDelayed(0, 50);
      }

    };
  };
  
  @Override
  protected void onDraw(Canvas canvas) {
    for (int i = 0; i < wList.size(); i++) {
      Wave wave = wList.get(i);
      canvas.drawCircle(wave.cx, wave.cy, wave.r, wave.p);
    }
  }
  
  @Override
  public boolean onTouchEvent(MotionEvent event) {
    super.onTouchEvent(event);
    
    switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:
    case MotionEvent.ACTION_MOVE:
      
      int x = (int) event.getX();
      int y = (int) event.getY();
      
      addPoint(x,y);
      
      break;

    default:
      break;
    }
    
    return true; 
    
  }
  
  /**
   * 添加新的波浪中心点
   * @param x
   * @param y
   */
  private void addPoint(int x, int y) {
    if(wList.size() == 0){
      addPoint2List(x,y);
      /*
       * 第一次启动动画
       */
      isRunning = true;
      handler.sendEmptyMessage(0);
    }else{
      Wave w = wList.get(wList.size()-1);
      
      if(Math.abs(w.cx - x)>DIS_SOLP || Math.abs(w.cy-y)>DIS_SOLP){
        addPoint2List(x,y);
      }
      
    };
    
  }

  /**
   * 添加新的波浪
   * @param x
   * @param y
   */
  private void addPoint2List(int x, int y) {
    Wave w = new Wave();
    w.cx = x;
    w.cy=y;
    Paint pa=new Paint();
    pa.setColor(colors[(int)(Math.random()*4)]);
    pa.setAntiAlias(true);
    pa.setStyle(Style.STROKE);

    w.p = pa;
    
    wList.add(w);
  }

  private int [] colors = new int[]{Color.BLUE,Color.RED,Color.YELLOW,Color.GREEN};
  /**
   * 刷新数据
   */
  private void flushData() {
    
    for (int i = 0; i < wList.size(); i++) {
      
      Wave w = wList.get(i);
      
      //如果透明度为 0 从集合中删除
      int alpha = w.p.getAlpha();
      if(alpha == 0){
        wList.remove(i);  //删除i 以后，i的值应该再减1 否则会漏掉一个对象，不过，在此处影响不大，效果上看不出来。
        continue;
      }
      
      alpha-=5;
      if(alpha<5){
        alpha =0;
      }
      //降低透明度
      w.p.setAlpha(alpha);
      
      //扩大半径
      w.r = w.r+3;
      //设置半径厚度
      w.p.setStrokeWidth(w.r/3);
    }
    
    /*
     * 如果集合被清空，就停止刷新动画
     */
    if(wList.size() == 0){
      isRunning = false;
    }
  }

  /**
   * 定义一个波浪
   * @author leo
   */
  private class Wave {
    //圆心
    int cx;
    int cy;
    
    //画笔
    Paint p;
    //半径
    int r;
  }
}
```

### 完成，效果如下 

![如下](http://img.blog.csdn.net/20160328205745798)










