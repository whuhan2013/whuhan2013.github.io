---
layout: post
title: Android之常用知识点总结(一)
date: 2016-3-26
categories: blog
tags: [android]
description: 安卓。
---

### 本文主要包括安卓一些常用的知识点  

1.  android常用的四种响应按钮点击事件的方法  
2. android动态刷新界面  
3.  android常用的listView用法  
4.  android常用的handler的用法    

### android常用的四种响应按钮点击事件的方法有
 

1.内部类   

2.匿名内部类  

3.布局文件夹定义Onclick属性,并在activity中声明方法  

4.在主类中实现OncickListener接口,并在主类中实现未实现的方法  


1.内部类    

```
btnButton.setOnClickListener(new MyListener());  
  
class MyListener implements OnClickListener {  
  
        @Override  
        public void onClick(View v) {  
            System.out.println("内部类响应点击事件");  
        }  
  
    }  
```  
2.匿名内部类  

```
btnButton.setOnClickListener(new OnClickListener() {  
              
            @Override  
            public void onClick(View v) {  
                System.out.println("匿名内部类响应按钮点击事件");  
            }  
        });  
```
3.布局文件夹定义Onclick属性,并在activity中声明方法  
定义Onclick属性   

```
<Button  
    android:id="@+id/loginButton"  
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"  
    android:onClick="btnOnClick"  
    android:layout_alignParentRight="true"  
    android:text="登入" />  
声明 "btnOnClick"方法
[java] view plain copy 在CODE上查看代码片派生到我的代码片
public void btnOnClick(View v) {  
    System.out.println("定义属性响应按钮点击事件");  
}  
```  
4.在主类中实现OncickListener接口,并在主类中实现未实现的方法   

```
btnButton.setOnClickListener(this);  
```    

```   
public class MainActivity extends Activity implements OnClickListener  
```    

```   
@Override  
public void onClick(View v) {  
    // TODO Auto-generated method stub  
      
}  
```       

### 动态刷新界面实现  
第一步：定义一个LinearLayout作为将来加载的条目的容器   

```  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity" >

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="onClick"
        android:text="添加" />

    <ScrollView
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" >

        <LinearLayout
            android:id="@+id/ll"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:orientation="vertical" >
        </LinearLayout>
    </ScrollView>

</LinearLayout>
```  

### 第二步：定义textView并加入到容器中    

```  
public class MainActivity extends Activity {

  private LinearLayout llGroup;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    llGroup = (LinearLayout) findViewById(R.id.ll);
  }

  public void onClick(View view) {
    
    // 添加一个TextView向llGroup
    
    // 定义一个textview对象
    TextView tv = new TextView(this);
    tv.setText("张三　　　女　　　34");
    
    // 把textview对象添加到linearlayout中
    llGroup.addView(tv);
  }
}
```  

### listView实现    

listView是安卓中一种常用的控件，有以下三种实现方法   

1. simpeAdapter  

```
SimpleAdapter adapter = new SimpleAdapter(
        this, // 上下文
        data, // listView绑定的数据
        R.layout.listview_item, // listview的子条目的布局的id
        new String[]{"name", "icon"},   // data数据中的map集合里的key
        new int[]{R.id.tv_name, R.id.iv_icon}); // resource 中的id
    
    mListView.setAdapter(adapter);
```    

其中data是ArrayList类型的数据，里面存储了map类型的数据，有两个键name,incon  

```
List<Map<String, Object>> data =ArrayList<Map<String,Object>>();
Map<String, Object> map = new HashMap<String, Object>();

map = new HashMap<String, Object>();
    map.put("name", "张三5");
    map.put("icon", R.drawable.f007);
    data.add(map);
```   

2. arrayAdapter   

```  
ListView mListView = (ListView) findViewById(R.id.listview);
    String[] textArray = {"功能1","功能2","功能3","功能4","功能5","功能6","功能7","功能8"};
    
    /*
     * 定义数据适配器
     * android.R.layout.simple_list_item_1  Listview的子条目显示的布局的id
     * textArray 显示在ListView列表中的数据
     */
    ArrayAdapter<String> adapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, 
        textArray);
    
    mListView.setAdapter(adapter);
```   

3. 最常见的继承baseAdapter  
在oncreate方法中   

```  
 ListView mListView = (ListView) findViewById(R.id.listview);
        
        PersonDao dao = new PersonDao(this);
        personList = dao.queryAll();
        
        // 把view层对象ListView和控制器BaseAdapter关联起来
        mListView.setAdapter(new MyAdapter());
  ```       

  ### anapter实现   

  ```     
   /**
     * @author andong
     * 数据适配器
     */
    class MyAdapter extends BaseAdapter {

      private static final String TAG = "MyAdapter";

    /**
       * 定义ListView的数据的长度
       */
    @Override
    public int getCount() {
      return personList.size();
    }

    @Override
    public Object getItem(int position) {
      // TODO Auto-generated method stub
      return null;
    }

    @Override
    public long getItemId(int position) {
      // TODO Auto-generated method stub
      return 0;
    }

    /**
     * 此方法返回的是ListView的列表中某一行的View对象
     * position 当前返回的view的索引位置
     * convertView 缓存对象
     * parent 就是ListView对象
     */
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
      
      
      TextView tv = null;
      
      if(convertView != null) {   // 判断缓存对象是否为null,  不为null时已经缓存了对象
        Log.i(TAG, "getView: 复用缓存" + position);
        tv = (TextView) convertView;
      } else {  // 等于null, 说明第一次显示, 新创建
        Log.i(TAG, "getView: 新建" + position);
        tv = new TextView(MainActivity.this);
      }
      
      tv.setTextSize(25);
      
      Person person = personList.get(position); // 获得指定位置的数据, 进行对TextView的绑定
      
      tv.setText(person.toString());
      
      return tv;
    }
      
    }
    
```   
   
### 在listView中展示的控件也可以是自定义的    

```  
/**
     * 此方法返回的是ListView的列表中某一行的View对象
     * position 当前返回的view的索引位置
     * convertView 缓存对象
     * parent 就是ListView对象
     */
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
      View view = null;
      
      if(convertView == null) {
        // 布局填充器对象, 用于把xml布局转换成view对象
        LayoutInflater inflater = MainActivity2.this.getLayoutInflater();
        view = inflater.inflate(R.layout.listview_item, null);
      } else {
        view = convertView;
      }
      
      // 给view中的姓名和年龄赋值
      TextView tvName = (TextView) view.findViewById(R.id.tv_listview_item_name);
      TextView tvAge = (TextView) view.findViewById(R.id.tv_listview_item_age);
      
      Person person = personList.get(position);
      
      tvName.setText("姓名: " + person.getName());
      tvAge.setText("年龄: " + person.getAge());
      return view;
    }
```   

### handler实现  
1. 定义一个消息接收器   

```  
/**
       * 接收消息
       */
    @Override
    public void handleMessage(Message msg) {
      switch (msg.what) {
      case SUCCESS:   // 访问成功, 有数据
        // 给Listview列表绑定数据
        
        newInfoList = (List<NewInfo>) msg.obj;

        MyAdapter adapter = new MyAdapter();
        lvNews.setAdapter(adapter);
        break;
      case FAILED:  // 无数据
        Toast.makeText(MainActivity.this, "当前网络崩溃了.", 0).show();
        break;
      default:
        break;
      }
    }
    };
     
```    

2.  定义一个子线程发送消息    

```   
private void init() {
    lvNews = (ListView) findViewById(R.id.lv_news);

    // 抓取新闻数据
    new Thread(new Runnable() {
      @Override
      public void run() {
        // 获得新闻集合
        List<NewInfo> newInfoList = getNewsFromInternet();
        Message msg = new Message();
        if(newInfoList != null) {
          msg.what = SUCCESS;
          msg.obj = newInfoList;
        } else {
          msg.what = FAILED;
        }
        handler.sendMessage(msg);
      }
    }).start();
    
    
  } 
```   

### 完成













