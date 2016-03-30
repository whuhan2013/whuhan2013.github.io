---
layout: post
title: Android之Intent深入
date: 2016-3-30
categories: blog
tags: [android]
description: Android之Intent深入
---
### Android中的意图包含多种用法，本文主要包括以下内容  

1. 显式意图     
2.  隐匿意图     
3.  要求结果回传的意图    

显式意图  ：必须指定要激活的组件的完整包名和类名 （应用程序之间耦合在一起）         
一般激活自己应用的组件的时候 采用显示意图        
  
隐式意图： 只需要指定要动作和数据就可以 （ 好处应用程序之间没有耦合）           
激活别人写的应用  隐式意图， 不需要关心对方的包名和类名

### 显式意图   

```
//意图     开电视  打人  打酱油
    Intent intent = new Intent(this, CalcActivity.class);
    intent.putExtra("name", name);
    //显式意图
    //intent.setClassName(getPackageName(), "com.itheima.rpcalc.CalcActivity");//不指定动作，也不指定数据 直接指定要激活的组件
    startActivity(intent);
```    

### 显式意图接收   

```
public class CalcActivity extends Activity {
  private TextView tv_result;

  //当activity被创建的时候调用的方法
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_calc);
    tv_result = (TextView) findViewById(R.id.tv_result);
    Intent intent = getIntent();
    String name = intent.getStringExtra("name");
    byte[] result = name.getBytes();
    int number = 0;
    for(byte b :result){
       number += b&0xff;
    }
    int sorce = Math.abs(number)%100;
    tv_result.setText(name+"的人品："+sorce);
  }
}
```

### 隐式意图 

隐式意图只要设置action与data即可  

### 要实现隐式意图，首先要在Manifest文件中配置action,category,mimetype等

```
<activity android:name="com.itheima.intent2.SecondActivity" >
            <intent-filter>
                <action android:name="com.itheima.intent2.open2" />

                <category android:name="android.intent.category.DEFAULT" />

                <data android:mimeType="application/person" />
                <data android:scheme="jianren" />
            </intent-filter>
     
        </activity>

```   

### 隐式意图的实现   

```
public void click(View view) {
    // 打 action
    // 人 数据
    // 附件的数据 Category 类别
    Intent intent = new Intent();
    intent.setAction("com.itheima.intent2.open2");
    intent.addCategory(Intent.CATEGORY_DEFAULT);

    // URL:统一资源定位符 http https ftp rtsp: URI：统一资源标识符 url是uri的一个子集
    // intent.setData(Uri.parse("jianren:张三"));  setData与setType是对立的，不能同时使用，同时使用时要用setDataAndType
    // intent.setType("application/person");
    intent.setDataAndType(Uri.parse("jianren:张三"), "application/person");
    startActivity(intent);
  }
```    

### 使用隐式意图打开浏览器的一个例子   

### 浏览器的属性配置如下   

```
<intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />
                <data android:scheme="http" />
                <data android:scheme="https" />
                <data android:scheme="inline" />
                <data android:mimeType="text/html"/>
                <data android:mimeType="text/plain"/>
                <data android:mimeType="application/xhtml+xml"/>
                <data android:mimeType="application/vnd.wap.xhtml+xml"/>
            </intent-filter>
```    

### 利用隐式意图实现  

```
  public void click(View view){
//        <action android:name="android.intent.action.VIEW" />
//        <category android:name="android.intent.category.DEFAULT" />
//        <category android:name="android.intent.category.BROWSABLE" />
//        <data android:scheme="http" />
//        <data android:scheme="https" />
//        <data android:scheme="about" />
//        <data android:scheme="javascript" />
    Intent intent = new Intent();
    intent.setAction("android.intent.action.VIEW");
    intent.addCategory("android.intent.category.DEFAULT");
    intent.addCategory("android.intent.category.BROWSABLE");
    intent.setData(Uri.parse("http://www.baidu.com"));
    startActivity(intent);
  }
```


### 得到返回值的Intent实现,短信助手实例   

```
public class MainActivity extends Activity {
  private EditText et_content;
  private EditText et_number;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    et_content = (EditText) findViewById(R.id.et_content);
    et_number = (EditText) findViewById(R.id.et_number);
  }

  public void selectSms(View view) {
    Intent intent = new Intent(this, ListSmsActivity.class);
    // 开启一个新的界面，并且获取界面的返回值
    // startActivity(intent);
    // int requestCode 请求码
    startActivityForResult(intent, 0);
  }

  public void selectNumber(View view) {
    Intent intent = new Intent(this, ListNumberActivity.class);
    // 开启一个新的界面，并且获取界面的返回值
    // startActivity(intent);
    // int requestCode 请求码
    startActivityForResult(intent, 1);
  }

  @Override
  protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (data != null) {
      String smsinfo = data.getStringExtra("smsinfo");
      if (resultCode == 0) {
        et_content.setText(smsinfo);
      } else if (resultCode == 1) {
        et_number.setText(smsinfo);
      }
    }
    super.onActivityResult(requestCode, resultCode, data);
  }

  
  public void sendSms(View view){
    String content = et_content.getText().toString().trim();
    String number = et_number.getText().toString().trim();
    SmsManager.getDefault().sendTextMessage(number, null, content, null, null);
    Toast.makeText(this, "发送成功", 0).show();
  }
}
```   

### 编辑短信   

```
public class ListSmsActivity extends Activity {
  private ListView lv;
  private String[] objects = {
      "玫瑰香香，情人黏黏，情话甜甜，情歌绵绵；花灯灿灿，礼花点点，好运连连，好梦圆圆。情人节喜逢元宵节，喜鹊登枝蝴蝶成双鸳鸯成对双喜临门祝双节快乐，合家团团圆圆，甜甜蜜蜜，开开心心，幸幸福福",
      "情人节快到了，我精心挑选玫瑰花、百合花和满天星，扎成一束鲜花随短信送给你，火红的玫瑰代表热烈奔放，洁白的百合代表百年好合，小小的满天星代表幸福美好。愿你的情人节热烈奔放，你们的爱情百年好合，你们的生活幸福美满。预祝情人节快乐",
      "^o^满天星光，最爱你许过愿望的那一颗，鲜花绽放，最爱你摘下微笑的那一朵，曼妙旋律，最爱你感动落泪的那一段，亲爱的，情人节快乐，爱你。 ",
      "^o^宝贝，情人节到了，送你一束玫瑰，用真心塑料纸包扎，系上快乐彩带，喷点爱的香水，插一张真情卡片，写着：宝贝，愿我的爱能带给你一生的快乐！" };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_sms);
    lv = (ListView) findViewById(R.id.lv);
    lv.setAdapter(new ArrayAdapter<String>(this, R.layout.sms_item,
        R.id.tv_info, objects));
    
    lv.setOnItemClickListener(new OnItemClickListener() {

      @Override
      public void onItemClick(AdapterView<?> parent, View view,
          int position, long id) {
        String smsinfo = objects[position];
        Intent data = new Intent();
        data.putExtra("smsinfo", smsinfo);
        //设置数据。
        setResult(0, data);
        //关闭掉当前的activity，并且回传数据 onActivityResult().
        finish();
      }
    });
  }
}
```    

### 选择联系人    

```
public class ListNumberActivity extends Activity {
  private ListView lv;
  private String[] objects = {
      "1234","34324","5643543","32424" };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_sms);
    lv = (ListView) findViewById(R.id.lv);
    lv.setAdapter(new ArrayAdapter<String>(this, R.layout.sms_item,
        R.id.tv_info, objects));
    
    lv.setOnItemClickListener(new OnItemClickListener() {

      @Override
      public void onItemClick(AdapterView<?> parent, View view,
          int position, long id) {
        String smsinfo = objects[position];
        Intent data = new Intent();
        data.putExtra("smsinfo", smsinfo);
        //设置数据。
        setResult(1, data);
        //关闭掉当前的activity，并且回传数据 onActivityResult().
        finish();
      }
    });
  }
}
```   


### 完成












