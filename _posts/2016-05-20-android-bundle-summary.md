---
layout: post
title: Android之Bundle类
date: 2016-5-20
categories: blog
tags: [android]
description: Android之Bundle类
---   

### API文档说明
 
 1.介绍
 
用于不同Activity之间的数据传递
 
### 1.重要方法
 
- clear()：清除此Bundle映射中的所有保存的数据。
 
- clone()：克隆当前Bundle
 
- containsKey(String key)：返回指定key的值
 
- getString(String key)：返回指定key的字符
 
- hasFileDescriptors()：指示是否包含任何捆绑打包文件描述符
 
- isEmpty()：如果这个捆绑映射为空，则返回true
 
- putString(String key, String value):插入一个给定key的字符串值
 
- readFromParcel(Parcel parcel)：读取这个parcel的内容
 
- remove(String key)：移除指定key的值
 
- writeToParcel(Parcel parcel, int flags)：写入这个parcel的内容


### 官方文档

[http://developer.android.com/reference/android/os/Bundle.html](http://developer.android.com/reference/android/os/Bundle.html)


### 实例

```
public class BundleDemo extends Activity {
 private EditText etName;
 Button btn;
 
 /*
  * (non-Javadoc)
  * 
  * @see android.app.Activity#onCreate(android.os.Bundle)
  */
 @Override
 protected void onCreate(Bundle savedInstanceState) {
  // TODO Auto-generated method stub
  super.onCreate(savedInstanceState);
 
  setContentView(R.layout.bundle);
 
  etName = (EditText) findViewById(R.id.etname);
  btn = (Button) findViewById(R.id.btn);
  btn.setOnClickListener(new OnClickListener() {
 
   @Override
   public void onClick(View v) {
    String info = etName.getText().toString();
    Bundle bundle = new Bundle();
 
　　//保存输入的信息
    bundle.putString("name", info);
    Intent intent=new Intent(BundleDemo.this,BundleDemo1.class);
   intent.putExtras(bundle);
   finish();
   startActivity(intent);
   }
  });
 
 }
 
}
 
 
 
public class BundleDemo1 extends Activity {
private TextView etName;
 /* (non-Javadoc)
  * @see android.app.Activity#onCreate(android.os.Bundle)
  */
 @Override
 protected void onCreate(Bundle savedInstanceState) {
  // TODO Auto-generated method stub
  super.onCreate(savedInstanceState);
  
  setContentView(R.layout.b1);
  
  etName=(TextView)findViewById(R.id.txtname);
  Bundle b=getIntent().getExtras();
  //获取Bundle的信息
  String info=b.getString("name");
  etName.setText("您的姓名："+info);
 }
 
}
```


### 与SharedPreferences的区别
 
　　SharedPreferences是简单的存储持久化的设置，就像用户每次打开应用程序时的主页，它只是一些简单的键值对来操作。它将数据保存在一个xml文件中
 
　　Bundle是将数据传递到另一个上下文中或保存或回复你自己状态的数据存储方式。它的数据不是持久化状态。



### 参考链接

[Android之Bundle传递数据详解与实例及Bundle与SharedPreferences的区别 - ForrestWoo - 博客园](http://www.cnblogs.com/salam/archive/2010/10/27/1862730.html)

[Android Bundle类 - randyjiawenjie的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/randyjiawenjie/article/details/6651437)


