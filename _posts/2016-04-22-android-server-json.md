---
layout: post
title: Android客户端与服务器之间传递json数据
date: 2016-4-22
categories: blog
tags: [网络编程]
description: Android客户端与服务器之间传递json数据
---

### 在服务器与客户端之间通信，json数据是一种常用格式，本文主要在服务器端构建数据，在客户端接收显示，并且在listview上显示出来  

### 服务器端的构建
简单的javabean与返回结果函数与插入函数略过  

```
public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
          response.setContentType("text/html");

          response.setCharacterEncoding("UTF-8");

          PrintWriter out = response.getWriter();

          List<MyShop> shops = JsonService.getListShop();

          StringBuffer sb = new StringBuffer();

          sb.append('[');

          for (MyShop shop : shops) {

                  sb.append('{').append("\"name\":").append("\""+shop.getName()+"\"").append(",");   
                  sb.append("\"detail\":").append("\""+shop.getDetail()+"\"").append(",");
                  sb.append("\"distance\":").append("\""+shop.getDistance()+"\"").append(",");
                  sb.append("\"address\":").append("\""+shop.getAddress()+"\"").append(",");
                  sb.append("\"popularity\":").append(shop.getPopularity());
                  sb.append('}').append(",");
          }

          sb.deleteCharAt(sb.length() - 1);

          sb.append(']');

          out.write(new String(sb));

          out.flush();

          out.close();

        
    }
```  

在浏览器中直接输入访问http://localhost:8080/AppServer/JsonServlet
可得
![json数据](http://img.blog.csdn.net/20160303165444160)

可以在服务器端直接查看json数据

### 客户端接收与解析json数据    


```
public class JsonParse {
    
    /**

     * 解析Json数据

     *

     * @param urlPath

     * @return mlists

     * @throws Exception

     */

    public static List<MyShop> getListShop(String urlPath) throws Exception {

            List<MyShop> mlists = new ArrayList<MyShop>();

            byte[] data = readParse(urlPath);

            JSONArray array = new JSONArray(new String(data));

            for (int i = 0; i < array.length(); i++) {

                    JSONObject item = array.getJSONObject(i);

                    String name = item.getString("name");
                    String detail = item.getString("detail");
                    String distance = item.getString("distance");

                    String popularity = item.getString("popularity");

                    String address = item.getString("address");

                    mlists.add(new MyShop(name, detail, distance,address,popularity));

            }

            return mlists;

    }

    /**

     * 从指定的url中获取字节数组

     *

     * @param urlPath

     * @return 字节数组

     * @throws Exception

     */

    public static byte[] readParse(String urlPath) throws Exception {

            ByteArrayOutputStream outStream = new ByteArrayOutputStream();

            byte[] data = new byte[1024];

            int len = 0;

            URL url = new URL(urlPath);

            HttpURLConnection conn = (HttpURLConnection) url.openConnection();

            InputStream inStream = conn.getInputStream();

            while ((len = inStream.read(data)) != -1) {

                    outStream.write(data, 0, len);

            }

            inStream.close();

            return outStream.toByteArray();

    }

}
```   

### 在activity中开启子线程来接收服务器数据  

```
new Thread(new Runnable() {
            
            private String tag;

            @Override
            public void run() {
                // TODO Auto-generated method stub
                //获得新闻集合
                
                List<MyShop> shopList = null;
                try {
                    shopList = JsonParse.getListShop("http://192.168.247.1:8080/AppServer/JsonServlet");
                } catch (Exception e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                Log.i(tag, "在RUN中LIST长度为"+shopList.size());
                Message msg=new Message();
                if(shopList!=null)
                {
                    msg.what=SUCCESS;
                    msg.obj=shopList;
                    
                }else
                {
                    msg.what=FAILED;
                }
                handler.sendMessage(msg);
                Log.i(tag, "***********T长度为"+shopList.size());
                    
                
            }
        }).start();
```  

### 消息处理器  

```
 //消息处理器
    private Handler handler=new Handler(){

        /**
         * 接收消息
         */
        @Override
        public void handleMessage(Message msg) {
            // TODO Auto-generated method stub
            
            String tag = null;
            switch (msg.what) {
            case SUCCESS:  //访问成功，有数据
                
                //绑定数据
                Log.i(tag,"%%%%%%%%%%到达了消息处理器");
                myshoplist=(List<MyShop>) msg.obj;
                Log.i(tag, "handleMessage中数据newInfoList长度为"+myshoplist.size());
                NearAdapter adapter=new NearAdapter();
                Log.i(tag, "有没有到达ADAPTER"+adapter.getCount());
                showList.setAdapter(adapter);
                
                break;
                
          case FAILED:    //访问失败
                Toast.makeText(ChooseMer.this, "当前网络崩溃了", 0).show();
                break;

            default:
                break;
            }
            
        }
        
        
    };
``` 

### 配置适配器     

```
public class NearAdapter extends BaseAdapter {

        @Override
        public int getCount() {
            // TODO Auto-generated method stub
            return  myshoplist.size();
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

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {

            if (convertView == null) {
                LayoutInflater inflater = LayoutInflater
                        .from(ChooseMer.this);
                convertView = inflater.inflate(R.layout.nearby_list_item, null);
                init(convertView,position);
            }

            return convertView;
        }

        public void init(View convertView,int position) {
            hold.name = (TextView) convertView
                    .findViewById(R.id.nearby_item_name);
            MyShop shop=myshoplist.get(position);
            hold.name.setText(shop.getName());
            hold.local = (TextView) convertView
                    .findViewById(R.id.nearby_item_local);
            hold.local.setText(shop.getDetail());
            hold.dis1 = (TextView) convertView
                    .findViewById(R.id.nearby_item_dis1);
            hold.dis1.setText(shop.getDistance());
            hold.dis2 = (TextView) convertView
                    .findViewById(R.id.nearby_item_dis2);
            hold.dis2.setText(shop.getAddress());
            hold.dis3 = (TextView) convertView
                    .findViewById(R.id.nearby_item_dis3);
            hold.dis3.setText(shop.getPopularity());
        }
    }
```

### 配置完成，效果如下    
![安卓显示](http://img.blog.csdn.net/20160303170044053)    



### 安卓客户端向服务器发送json数据实现  

### 方法一

```
public void sendJsonToServer() {
    HttpClient httpClient = new DefaultHttpClient();
    try {

      HttpPost httpPost = new HttpPost(constant.url);
      HttpParams httpParams = new BasicHttpParams();
      List<NameValuePair> nameValuePair = new ArrayList<NameValuePair>();
      Gson gson = new Gson();
      String str = gson.toJson(initData());
      nameValuePair.add(new BasicNameValuePair("jsonString", URLEncoder
          .encode(str, "utf-8")));
      httpPost.setEntity(new UrlEncodedFormEntity(nameValuePair));
      httpPost.setParams(httpParams);
      Toast.makeText(Main.this, "发送的数据：\n" + str.toString(),
          Toast.LENGTH_SHORT).show();
      httpClient.execute(httpPost);
      HttpResponse response = httpClient.execute(httpPost);
      StatusLine statusLine = response.getStatusLine();
      if (statusLine != null && statusLine.getStatusCode() == 200) {
        HttpEntity entity = response.getEntity();
        if (entity != null) {
          Toast.makeText(
              Main.this,
              "服务器处理返回结果：" + readInputStream(entity.getContent()),
              Toast.LENGTH_SHORT).show();
        } else {
          Toast.makeText(Main.this, "没有返回相关数据", Toast.LENGTH_SHORT)
              .show();
        }
      } else {
        Toast.makeText(Main.this, "发送失败，可能服务器忙，请稍后再试",
            Toast.LENGTH_SHORT).show();
      }
    } catch (Exception e) {
      throw new RuntimeException(e);
    }
  }

  private static String readInputStream(InputStream is) throws IOException {
    if (is == null)
      return null;
    ByteArrayOutputStream bout = new ByteArrayOutputStream();
    int len = 0;
    byte[] buf = new byte[1024];
    while ((len = is.read(buf)) != -1) {
      bout.write(buf, 0, len);
    }
    is.close();
    return URLDecoder.decode(new String(bout.toByteArray()), "utf-8");
  }
/*
 * 填充数据源
 */
  public List<Product> initData() {
    List<Product> persons = new ArrayList<Product>();
    for (int i = 0; i < 5; i++) {
      Product p = new Product();
      p.setLocation("所在位置");
      p.setName("名称" + i);
      persons.add(p);
    }
    return persons;
  }
```   

### 参考链接 
[Android向服务器端发送json数据 - 推酷](http://www.tuicool.com/articles/FZJR3eB)      



### 方法二 


```
public void SendData()
        {
            if(IntentState!=1)
            {
                return;
            }
            
            if(nowId>=getTop())
            {
                return;
            }
            
            CarBean myCar=queryItem(nowId);
             /*StringBuffer sb = new StringBuffer();
             sb.append('[');
             sb.append('{').append("\"rpm\":").append("\""+myCar.getRpm()+"\"").append(",");   
              sb.append("\"speed\":").append("\""+myCar.getSpeed()+"\"").append(",");
              sb.append("\"airSpeed\":").append("\""+myCar.getAirSpeed()+"\"").append(",");
              sb.append("\"throttle\":").append("\""+myCar.getThrottle()+"\"").append(",");
              sb.append("\"oilLevel\":").append("\""+myCar.getOilLevel()+"\"");
              sb.append('}');
              sb.append(']');
              Log.i("test", sb.toString());*/
             
              JSONObject jsonObject=new JSONObject();
              try {
                jsonObject.put("0C", myCar.getRpm());
                jsonObject.put("0D",myCar.getSpeed());
                jsonObject.put("10",myCar.getAirSpeed());
                jsonObject.put("11",myCar.getThrottle());
                jsonObject.put("2F",myCar.getOilLevel());
                
               strJSON = String.valueOf(jsonObject);
                System.out.println(strJSON);
                Log.i("test", "result===="+strJSON);
            } catch (JSONException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
              
              
              new Thread(new Runnable() {
                
                @Override
                public void run() {
                    // TODO Auto-generated method stub
                    System.out.println("strJSON ready to launch!" + strJSON);
                    HttpURLConnection conn = null;
                    String content = "";
                    try {
                        Log.i("test", "test1");
                        URL url = new URL(
                                "http://192.168.1.130:8080/Test/transAction");
                        conn = (HttpURLConnection) url.openConnection();
                        Log.i("test", "test2");
                        conn.setDoInput(true);
                        conn.setDoOutput(true);
                        conn.setConnectTimeout(1000 * 30);
                        conn.setReadTimeout(1000 * 30);
                        conn.setRequestMethod("GET");
                        conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
                        OutputStream output = conn.getOutputStream();
                        

                        output.write(strJSON.getBytes());

                    
                        output.flush();
                        output.close();
                        // 关闭流对象。此时，不能再向对象输出流写入任何数据，先前写入的数据存在于内存缓冲区中,
                        // 在调用下边的getInputStream()函数时才把准备好的http请求正式发送到服务器
                        // objOutputStrm.close();
                        InputStream in = conn.getInputStream();
                        Log.i("test", "test3");
                        int responseCode = conn.getResponseCode();
                        if (responseCode == HttpURLConnection.HTTP_OK) {
                            Log.i("test", "test4");
                            System.out.println("responseCode=" + HttpURLConnection.HTTP_OK);
                            // InputStreamReader reader = new
                            // InputStreamReader(in,charSet);
                            delete(nowId);
                            nowId++;
                            Message msg=new Message();
                            msg.what=3;
                            handler.sendMessage(msg);
                            
                            BufferedReader reader = new BufferedReader(new InputStreamReader(in, "UTF-8"));
                            String line = null;
                            while ((line = reader.readLine()) != null) {
                                content += line;
                            }
                            System.out.println("result:" + content);
                        }else
                        {
                            Log.i("test", "test5");
                        }
                        in.close();
                    } catch (Exception e) {
                        e.printStackTrace();
                        Log.i("test", "test6");
                    } finally {
                        conn.disconnect();
                        conn = null;
                        Log.i("test", "test7");
                    }
                    strJSON = "null";
                }
            }).start();
        }

```  


### 完成  































