---
layout: post
title: Android之网络编程
date: 2016-3-29
categories: blog
tags: [网络编程]
description: Android之网络编程
---

### 本文主要包括三方面内容   

1. Httpurlconnection中doGet与doPost方法实现提交数据到服务器

2.  HttpClient中doGet与doPost方法实现提交数据到服务器

3.  android-async-http开源库方法实现提交数据到服务器  

### 首先是服务器端的实现  

```
public class LoginServlet extends HttpServlet {

  /**
   * The doGet method of the servlet. <br>
   *
   * This method is called when a form has its tag value method equals to get.
   * 
   * @param request the request send by the client to the server
   * @param response the response send by the server to the client
   * @throws ServletException if an error occurred
   * @throws IOException if an error occurred
   */
  public void doGet(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
    String username = request.getParameter("username"); // 采用的编码是: iso-8859-1
    String password = request.getParameter("password");
    
    // 采用iso8859-1的编码对姓名进行逆转, 转换成字节数组, 再使用utf-8编码对数据进行转换, 字符串
    username = new String(username.getBytes("iso8859-1"), "utf-8");
    password = new String(password.getBytes("iso8859-1"), "utf-8");
    
    System.out.println("姓名: " + username);
    System.out.println("密码: " + password);
    
    if("lisi".equals(username) && "123".equals(password)) {
      /*
       * getBytes 默认情况下, 使用的iso8859-1的编码, 但如果发现码表中没有当前字符, 
       * 会使用当前系统下的默认编码: GBK
       */ 
      response.getOutputStream().write("登录成功".getBytes("utf-8"));
    } else {
      response.getOutputStream().write("登录失败".getBytes("utf-8"));
    }
  }

  /**
   * The doPost method of the servlet. <br>
   *
   * This method is called when a form has its tag value method equals to post.
   * 
   * @param request the request send by the client to the server
   * @param response the response send by the server to the client
   * @throws ServletException if an error occurred
   * @throws IOException if an error occurred
   */
  public void doPost(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
    System.out.println("doPost");
    doGet(request, response);
  }

}
```

### Httpurlconnection实现提交数据到服务器  

```
public class NetUtils {

  private static final String TAG = "NetUtils";
  
  /**
   * 使用post的方式登录
   * @param userName
   * @param password
   * @return
   */
  public static String loginOfPost(String userName, String password) {
    HttpURLConnection conn = null;
    try {
      URL url = new URL("http://10.0.2.2:8080/ServerItheima28/servlet/LoginServlet");
      
      conn = (HttpURLConnection) url.openConnection();
      
      conn.setRequestMethod("POST");
      conn.setConnectTimeout(10000); // 连接的超时时间
      conn.setReadTimeout(5000); // 读数据的超时时间
      conn.setDoOutput(true); // 必须设置此方法, 允许输出
//      conn.setRequestProperty("Content-Length", 234);   // 设置请求头消息, 可以设置多个
      
      // post请求的参数
      String data = "username=" + userName + "&password=" + password;
      
      // 获得一个输出流, 用于向服务器写数据, 默认情况下, 系统不允许向服务器输出内容
      OutputStream out = conn.getOutputStream();  
      out.write(data.getBytes());
      out.flush();
      out.close();
      
      int responseCode = conn.getResponseCode();
      if(responseCode == 200) {
        InputStream is = conn.getInputStream();
        String state = getStringFromInputStream(is);
        return state;
      } else {
        Log.i(TAG, "访问失败: " + responseCode);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      if(conn != null) {
        conn.disconnect();
      }
    }
    return null;
  }

  /**
   * 使用get的方式登录
   * @param userName
   * @param password
   * @return 登录的状态
   */
  public static String loginOfGet(String userName, String password) {
    HttpURLConnection conn = null;
    try {
      String data = "username=" + URLEncoder.encode(userName) + "&password=" + URLEncoder.encode(password);
      URL url = new URL("http://10.0.2.2:8080/ServerItheima28/servlet/LoginServlet?" + data);
      conn = (HttpURLConnection) url.openConnection();
      
      conn.setRequestMethod("GET");   // get或者post必须得全大写
      conn.setConnectTimeout(10000); // 连接的超时时间
      conn.setReadTimeout(5000); // 读数据的超时时间
      
      int responseCode = conn.getResponseCode();
      if(responseCode == 200) {
        InputStream is = conn.getInputStream();
        String state = getStringFromInputStream(is);
        return state;
      } else {
        Log.i(TAG, "访问失败: " + responseCode);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      if(conn != null) {
        conn.disconnect();    // 关闭连接
      }
    }
    return null;
  }
  
  /**
   * 根据流返回一个字符串信息
   * @param is
   * @return
   * @throws IOException 
   */
  private static String getStringFromInputStream(InputStream is) throws IOException {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    byte[] buffer = new byte[1024];
    int len = -1;
    
    while((len = is.read(buffer)) != -1) {
      baos.write(buffer, 0, len);
    }
    is.close();
    
    String html = baos.toString();  // 把流中的数据转换成字符串, 采用的编码是: utf-8
    
//    String html = new String(baos.toByteArray(), "GBK");
    
    baos.close();
    return html;
  }
}
```  


### HttpClient实现提交数据到服务器  


**android6.0之后HttpClient已经废弃掉了，需要自己导包** 

[android sdk源码中怎么没有httpclient的源码了_百度知道](http://zhidao.baidu.com/link?url=WSdGo6GpOMi1hUNwBBWc2mqkLmblBLtrXFP3zciAoT7tjsdWExlYgz84K2sxA8f7OJ7dhs_k8m2mLDYyFSIEdzHkJwYs09hMXT1Lkt0GRZW)


```
public class NetUtils2 {

  private static final String TAG = "NetUtils";
  
  /**
   * 使用post的方式登录
   * @param userName
   * @param password
   * @return
   */
  public static String loginOfPost(String userName, String password) {
    HttpClient client = null;
    try {
      // 定义一个客户端
      client = new DefaultHttpClient();
      
      // 定义post方法
      HttpPost post = new HttpPost("http://10.0.2.2:8080/ServerItheima28/servlet/LoginServlet");
      
      // 定义post请求的参数
      List<NameValuePair> parameters = new ArrayList<NameValuePair>();
      parameters.add(new BasicNameValuePair("username", userName));
      parameters.add(new BasicNameValuePair("password", password));
      
      // 把post请求的参数包装了一层.
      
      // 不写编码名称服务器收数据时乱码. 需要指定字符集为utf-8
      UrlEncodedFormEntity entity = new UrlEncodedFormEntity(parameters, "utf-8");
      // 设置参数.
      post.setEntity(entity);
      
      // 设置请求头消息
//      post.addHeader("Content-Length", "20");
      
      // 使用客户端执行post方法
      HttpResponse response = client.execute(post); // 开始执行post请求, 会返回给我们一个HttpResponse对象
      
      // 使用响应对象, 获得状态码, 处理内容
      int statusCode = response.getStatusLine().getStatusCode();  // 获得状态码
      if(statusCode == 200) {
        // 使用响应对象获得实体, 获得输入流
        InputStream is = response.getEntity().getContent();
        String text = getStringFromInputStream(is);
        return text;
      } else {
        Log.i(TAG, "请求失败: " + statusCode);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      if(client != null) {
        client.getConnectionManager().shutdown(); // 关闭连接和释放资源
      }
    }
    return null;
  }

  /**
   * 使用get的方式登录
   * @param userName
   * @param password
   * @return 登录的状态
   */
  public static String loginOfGet(String userName, String password) {
    HttpClient client = null;
    try {
      // 定义一个客户端
      client = new DefaultHttpClient();
      
      // 定义一个get请求方法
      String data = "username=" + userName + "&password=" + password;
      HttpGet get = new HttpGet("http://10.0.2.2:8080/ServerItheima28/servlet/LoginServlet?" + data);
      
      // response 服务器相应对象, 其中包含了状态信息和服务器返回的数据
      HttpResponse response = client.execute(get);  // 开始执行get方法, 请求网络
      
      // 获得响应码
      int statusCode = response.getStatusLine().getStatusCode();
      
      if(statusCode == 200) {
        InputStream is = response.getEntity().getContent();
        String text = getStringFromInputStream(is);
        return text;
      } else {
        Log.i(TAG, "请求失败: " + statusCode);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      if(client != null) {
        client.getConnectionManager().shutdown(); // 关闭连接, 和释放资源
      }
    }
    return null;
  }
  
  /**
   * 根据流返回一个字符串信息
   * @param is
   * @return
   * @throws IOException 
   */
  private static String getStringFromInputStream(InputStream is) throws IOException {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    byte[] buffer = new byte[1024];
    int len = -1;
    
    while((len = is.read(buffer)) != -1) {
      baos.write(buffer, 0, len);
    }
    is.close();
    
    String html = baos.toString();  // 把流中的数据转换成字符串, 采用的编码是: utf-8
    
//    String html = new String(baos.toByteArray(), "GBK");
    
    baos.close();
    return html;
  }
}
```    


### 在onCreate方法中的调用  

```
public class MainActivity extends Activity {

  private EditText etUserName;
  private EditText etPassword;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    etUserName = (EditText) findViewById(R.id.et_username);
    etPassword = (EditText) findViewById(R.id.et_password);
  }

  public void doGet(View v) {
    final String userName = etUserName.getText().toString();
    final String password = etPassword.getText().toString();
    
    new Thread(
        new Runnable() {
          
          @Override
          public void run() {
            // 使用get方式抓去数据
            final String state = NetUtils.loginOfGet(userName, password);
            
            // 执行任务在主线程中
            runOnUiThread(new Runnable() {
              @Override
              public void run() {
                // 就是在主线程中操作
                Toast.makeText(MainActivity.this, state, 0).show();
              }
            });
          }
        }).start();
  }
  
  public void doPost(View v) {
    final String userName = etUserName.getText().toString();
    final String password = etPassword.getText().toString();
    
    new Thread(new Runnable() {
      @Override
      public void run() {
        final String state = NetUtils.loginOfPost(userName, password);
        // 执行任务在主线程中
        runOnUiThread(new Runnable() {
          @Override
          public void run() {
            // 就是在主线程中操作
            Toast.makeText(MainActivity.this, state, 0).show();
          }
        });
      }
    }).start();
  }
}
```   

### 使用runOnUiThread方法，可以在子线程中实现对主线程的操作  


### 使用android-async-http开源库方法实现提交数据到服务器 ,使用方法也是将src文件夹中的内容复制到程序中即可

```
public class MainActivity2 extends Activity {

  protected static final String TAG = "MainActivity2";
  private EditText etUserName;
  private EditText etPassword;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    etUserName = (EditText) findViewById(R.id.et_username);
    etPassword = (EditText) findViewById(R.id.et_password);
  }

  public void doGet(View v) {
    final String userName = etUserName.getText().toString();
    final String password = etPassword.getText().toString();
  
        AsyncHttpClient client = new AsyncHttpClient();

        String data = "username=" + URLEncoder.encode(userName) + "&password=" + URLEncoder.encode(password);
        
        client.get("http://10.0.2.2:8080/ServerItheima28/servlet/LoginServlet?" + data, new MyResponseHandler());
  }
  
  public void doPost(View v) {
    final String userName = etUserName.getText().toString();
    final String password = etPassword.getText().toString();
    
    AsyncHttpClient client = new AsyncHttpClient();

        RequestParams params = new RequestParams();
        params.put("username", userName);
        params.put("password", password);
        
        client.post("http://10.0.2.2:8080/ServerItheima28/servlet/LoginServlet", 
            params, 
            new MyResponseHandler());
  }
  
  class MyResponseHandler extends AsyncHttpResponseHandler {

    @Override
    public void onSuccess(int statusCode, Header[] headers,
        byte[] responseBody) {
//      Log.i(TAG, "statusCode: " + statusCode);
      
      Toast.makeText(MainActivity2.this, 
          "成功: statusCode: " + statusCode + ", body: " + new String(responseBody), 
          0).show();
    }

    @Override
    public void onFailure(int statusCode, Header[] headers,
        byte[] responseBody, Throwable error) {
      Toast.makeText(MainActivity2.this, "失败: statusCode: " + statusCode, 0).show();
    }
    }
}
```  

### 可以看到使用开源框架可以更加简单，高效，安全的实现相同的效果.

### 完成










