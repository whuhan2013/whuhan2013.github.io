---
layout: post
title: Android之查看网络图片和网页HTML
date: 2016-3-29
categories: blog
tags: [网络编程]
description: Android之查看网络图片和网页HTML
---

### 网络编程是Android应用中很重要的一部分，本文主要讲述了利用HttpURLConnection获取网络图片和HTML的方法。

### 获取网络图片  

```
public class MainActivity extends Activity implements OnClickListener {

  private static final String TAG = "MainActivity";
  protected static final int ERROR = 1;
  private EditText etUrl;
  private ImageView ivIcon;
  private final int SUCCESS = 0;
  
  private Handler handler = new Handler() {

    /**
     * 接收消息
     */
    @Override
    public void handleMessage(Message msg) {
      super.handleMessage(msg);
      
      Log.i(TAG, "what = " + msg.what);
      if(msg.what == SUCCESS) { // 当前是访问网络, 去显示图片
        ivIcon.setImageBitmap((Bitmap) msg.obj);    // 设置imageView显示的图片
      } else if(msg.what == ERROR) {
        Toast.makeText(MainActivity.this, "抓去失败", 0).show();
      }
    }
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    ivIcon = (ImageView) findViewById(R.id.iv_icon);
    etUrl = (EditText) findViewById(R.id.et_url);
    
    findViewById(R.id.btn_submit).setOnClickListener(this);
  }

  @Override
  public void onClick(View v) {
    final String url = etUrl.getText().toString();
    
    new Thread(new Runnable() {

      @Override
      public void run() {
        Bitmap bitmap = getImageFromNet(url);

//        ivIcon.setImageBitmap(bitmap);    // 设置imageView显示的图片
        if(bitmap != null) {
          Message msg = new Message();
          msg.what = SUCCESS;
          msg.obj = bitmap;
          handler.sendMessage(msg);
        } else {
          Message msg = new Message();
          msg.what = ERROR;
          handler.sendMessage(msg);
        }
      }}).start();
    
  }

  /**
   * 根据url连接取网络抓去图片返回
   * @param url
   * @return url对应的图片
   */
  private Bitmap getImageFromNet(String url) {
    HttpURLConnection conn = null;
    try {
      URL mURL = new URL(url);  // 创建一个url对象
      
      // 得到http的连接对象
      conn = (HttpURLConnection) mURL.openConnection();
      
      conn.setRequestMethod("GET");   // 设置请求方法为Get
      conn.setConnectTimeout(10000);    // 设置连接服务器的超时时间, 如果超过10秒钟, 没有连接成功, 会抛异常
      conn.setReadTimeout(5000);    // 设置读取数据时超时时间, 如果超过5秒, 抛异常
      
      conn.connect();   // 开始链接
      
      int responseCode = conn.getResponseCode(); // 得到服务器的响应码
      if(responseCode == 200) {
        // 访问成功
        InputStream is = conn.getInputStream(); // 获得服务器返回的流数据
        
        Bitmap bitmap = BitmapFactory.decodeStream(is); // 根据 流数据 创建一个bitmap位图对象
        
        return bitmap;
      } else {
        Log.i(TAG, "访问失败: responseCode = " + responseCode);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      if(conn != null) {
        conn.disconnect();    // 断开连接
      }
    }
    return null;
  }
}
```


### 不能子线程中改变主线程页面，故需要使用Handler

### 上面的方法较为烦琐，使用github上的开源库，android-smart-image-view可以有效的实现相同的功能，同时简化操作，使用方法是将开源库src文件夹下的内容复制一份到工程中，同时在布局文件中，使用全类名使用自定义控件SmartImageView即可.

### android-smart-image-view实现  

```
public class MainActivity2 extends Activity implements OnClickListener {

  private EditText etUrl;
  private SmartImageView mImageView;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main2);
    
    etUrl = (EditText) findViewById(R.id.et_url);
    mImageView = (SmartImageView) findViewById(R.id.iv_icon);
    
    findViewById(R.id.btn_submit).setOnClickListener(this);
  }

  @Override
  public void onClick(View v) {
    
    // 1. 取出url, 抓取图片
    String url = etUrl.getText().toString();
    
    mImageView.setImageUrl(url);
  }
}
```

### 查看网页HTML实现  

```
public class MainActivity extends Activity {

  private static final String TAG = "MainActivity";
  private static final int SUCCESS = 0;
  protected static final int ERROR = 1;
  private EditText etUrl;
  private TextView tvHtml;
  
  private Handler handler = new Handler() {

    @Override
    public void handleMessage(Message msg) {
      super.handleMessage(msg);
      switch (msg.what) {
      case SUCCESS:
         tvHtml.setText((String) msg.obj);
        break;
      case ERROR:
        Toast.makeText(MainActivity.this, "访问失败", 0).show();
        break;
      default:
        break;
      }
    }
    
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    etUrl = (EditText) findViewById(R.id.et_url);
    tvHtml = (TextView) findViewById(R.id.tv_html);
    
  }

  public void getHtml(View v) {
    final String url = etUrl.getText().toString();
    
    new Thread(new Runnable() {
      
      @Override
      public void run() {
        // 请求网络
        String html = getHtmlFromInternet(url);
        
        if(!TextUtils.isEmpty(html)) {
          // 更新textview的显示了
          Message msg = new Message();
          msg.what = SUCCESS;
          msg.obj = html;
          handler.sendMessage(msg);
        } else {
          Message msg = new Message();
          msg.what = ERROR;
          handler.sendMessage(msg);
        }
      }
    }).start();
  }

  /**
   * 根据给定的url访问网络, 抓去html代码
   * @param url
   * @return
   */
  protected String getHtmlFromInternet(String url) {
    
    try {
      URL mURL = new URL(url);
      HttpURLConnection conn = (HttpURLConnection) mURL.openConnection();
      
      conn.setRequestMethod("GET");
      conn.setConnectTimeout(10000);
      conn.setReadTimeout(5000);
      
//      conn.connect();
      
      int responseCode = conn.getResponseCode();
      
      if(responseCode == 200) {
        InputStream is = conn.getInputStream();
        String html = getStringFromInputStream(is);
        return html;
      } else {
        Log.i(TAG, "访问失败: " + responseCode);
      }
    } catch (Exception e) {
      e.printStackTrace();
    }
    return null;
  }
  
  /**
   * 根据流返回一个字符串信息
   * @param is
   * @return
   * @throws IOException 
   */
  private String getStringFromInputStream(InputStream is) throws IOException {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    byte[] buffer = new byte[1024];
    int len = -1;
    
    while((len = is.read(buffer)) != -1) {
      baos.write(buffer, 0, len);
    }
    is.close();
    
    String html = baos.toString();  // 把流中的数据转换成字符串, 采用的编码是: utf-8
    
    String charset = "utf-8";
    if(html.contains("gbk") || html.contains("gb2312")
        || html.contains("GBK") || html.contains("GB2312")) {   // 如果包含gbk, gb2312编码, 就采用gbk编码进行对字符串编码
      charset = "gbk";
    }
    
    html = new String(baos.toByteArray(), charset); // 对原有的字节数组进行使用处理后的编码名称进行编码
    baos.close();
    return html;
  }
}
```  

### 使用这种方法HTML有时会产生乱码，解决方法如上

### 完成










