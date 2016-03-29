---
layout: post
title: Android之XML序列化和解析与网易新闻实例
date: 2016-3-29
categories: blog
tags: [android]
description: Android之XML序列化和解析
---
### 本文主要包括两个知识点，XML序列化和解析与一个组合XML，网络编程等多个知识点的网易新闻实例

### XML文件是一种常用的文件格式，可以用来存储与传递数据 ，本文是XML文件序列化与解析的一个简单示例

### 写文件到本地，并用XML格式存储   

```
/**
   * 写xml文件到本地
   */
  private void writeXmlToLocal() {
    List<Person> personList = getPersonList();
    
    // 获得序列化对象
    XmlSerializer serializer = Xml.newSerializer();
    
    try {
      File path = new File(Environment.getExternalStorageDirectory(), "persons.xml");
      FileOutputStream fos = new FileOutputStream(path);
      // 指定序列化对象输出的位置和编码
      serializer.setOutput(fos, "utf-8");
      
      serializer.startDocument("utf-8", true);  // 写开始 <?xml version='1.0' encoding='utf-8' standalone='yes' ?>
      
      serializer.startTag(null, "persons");   // <persons>
      
      for (Person person : personList) {
        // 开始写人

        serializer.startTag(null, "person");  // <person>
        serializer.attribute(null, "id", String.valueOf(person.getId()));
        
        // 写名字
        serializer.startTag(null, "name");    // <name>
        serializer.text(person.getName());
        serializer.endTag(null, "name");    // </name>
        
        // 写年龄
        serializer.startTag(null, "age");   // <age>
        serializer.text(String.valueOf(person.getAge()));
        serializer.endTag(null, "age");   // </age>
        
        serializer.endTag(null, "person");  // </person>
      }
      
      serializer.endTag(null, "persons");     // </persons>
      
      serializer.endDocument();   // 结束
    } catch (Exception e) {
      e.printStackTrace();
    }
    
  }
  
  private List<Person> getPersonList() {
    List<Person> personList = new ArrayList<Person>();
    
    for (int i = 0; i < 30; i++) {
      personList.add(new Person(i, "wang" + i, 18 + i));
    }
    
    return personList;
  }
```

### XML解析实现  

```
private List<Person> parserXmlFromLocal() {
    try {
      File path = new File(Environment.getExternalStorageDirectory(), "persons.xml");
      FileInputStream fis = new FileInputStream(path);
      
      // 获得pull解析器对象
      XmlPullParser parser = Xml.newPullParser();
      // 指定解析的文件和编码格式
      parser.setInput(fis, "utf-8");
      
      int eventType = parser.getEventType();    // 获得事件类型
      
      List<Person> personList = null;
      Person person = null;
      String id;
      while(eventType != XmlPullParser.END_DOCUMENT) {
        String tagName = parser.getName();  // 获得当前节点的名称
        
        switch (eventType) {
        case XmlPullParser.START_TAG:     // 当前等于开始节点  <person>
          if("persons".equals(tagName)) { // <persons>
            personList = new ArrayList<Person>();
          } else if("person".equals(tagName)) { // <person id="1">
            person = new Person();
            id = parser.getAttributeValue(null, "id");
            person.setId(Integer.valueOf(id));
          } else if("name".equals(tagName)) { // <name>
            person.setName(parser.nextText());
          } else if("age".equals(tagName)) { // <age>
            person.setAge(Integer.parseInt(parser.nextText()));
          }
          break;
        case XmlPullParser.END_TAG:   // </persons>
          if("person".equals(tagName)) {
            // 需要把上面设置好值的person对象添加到集合中
            personList.add(person);
          }
          break;
        default:
          break;
        }
        
        eventType = parser.next();    // 获得下一个事件类型
      }
      return personList;
    } catch (Exception e) {
      e.printStackTrace();
    }
    return null;
  }
```

### 测试结果

```
public class TestCase extends AndroidTestCase {

  public void test() {
//    writeXmlToLocal();
    
    List<Person> personList = parserXmlFromLocal();
    
    for (Person person : personList) {
      Log.i("TestCase", person.toString());
    }
  }
```  

### 网易新闻实例  

```
public class MainActivity extends Activity {

    private static final String TAG = "MainActivity";
    private final int SUCCESS = 0;
    private final int FAILED = 1;
  private ListView lvNews;
  private List<NewInfo> newInfoList;
    
    private Handler handler = new Handler() {

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

  @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        init();
    }

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

  /**
   * 返回新闻信息
   */
  private List<NewInfo> getNewsFromInternet() {
    HttpClient client = null;
    try {
      // 定义一个客户端
      client = new DefaultHttpClient();
      
      // 定义get方法
      HttpGet get = new HttpGet("http://192.168.1.254:8080/NetEaseServer/new.xml");
      
      // 执行请求
      HttpResponse response = client.execute(get);
      
      int statusCode = response.getStatusLine().getStatusCode();
      
      if(statusCode == 200) {
        InputStream is = response.getEntity().getContent();
        List<NewInfo> newInfoList = getNewListFromInputStream(is);
        return newInfoList;
      } else {
        Log.i(TAG, "访问失败: " + statusCode);
      }
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      if(client != null) {
        client.getConnectionManager().shutdown();   // 关闭和释放资源
      }
    }
    return null;
  }
    
  /**
   * 从流中解析新闻集合
   * @param is
   * @return
   */
  private List<NewInfo> getNewListFromInputStream(InputStream is) throws Exception {
    XmlPullParser parser = Xml.newPullParser(); // 创建一个pull解析器
    parser.setInput(is, "utf-8"); // 指定解析流, 和编码
    
    int eventType = parser.getEventType();
    
    List<NewInfo> newInfoList = null;
    NewInfo newInfo = null;
    while(eventType != XmlPullParser.END_DOCUMENT) {  // 如果没有到结尾处, 继续循环
      
      String tagName = parser.getName();  // 节点名称
      switch (eventType) {
      case XmlPullParser.START_TAG: // <news>
        if("news".equals(tagName)) {
          newInfoList = new ArrayList<NewInfo>();
        } else if("new".equals(tagName)) {
          newInfo = new NewInfo();
        } else if("title".equals(tagName)) {
          newInfo.setTitle(parser.nextText());
        } else if("detail".equals(tagName)) {
          newInfo.setDetail(parser.nextText());
        } else if("comment".equals(tagName)) {
          newInfo.setComment(Integer.valueOf(parser.nextText()));
        } else if("image".equals(tagName)) {
          newInfo.setImageUrl(parser.nextText());
        }
        break;
      case XmlPullParser.END_TAG: // </news>
        if("new".equals(tagName)) {
          newInfoList.add(newInfo);
        }
        break;
      default:
        break;
      }
      eventType = parser.next();    // 取下一个事件类型
    }
    return newInfoList;
  }
  
  class MyAdapter extends BaseAdapter {

    /**
     * 返回列表的总长度
     */
    @Override
    public int getCount() {
      return newInfoList.size();
    }

    /**
     * 返回一个列表的子条目的布局
     */
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
      View view = null;
      
      if(convertView == null) {
        LayoutInflater inflater = getLayoutInflater();
        view = inflater.inflate(R.layout.listview_item, null);
      } else {
        view = convertView;
      }
      
      // 重新赋值, 不会产生缓存对象中原有数据保留的现象
      SmartImageView sivIcon = (SmartImageView) view.findViewById(R.id.siv_listview_item_icon);
      TextView tvTitle = (TextView) view.findViewById(R.id.tv_listview_item_title);
      TextView tvDetail = (TextView) view.findViewById(R.id.tv_listview_item_detail);
      TextView tvComment = (TextView) view.findViewById(R.id.tv_listview_item_comment);
      
      NewInfo newInfo = newInfoList.get(position);
      
      sivIcon.setImageUrl(newInfo.getImageUrl());   // 设置图片
      tvTitle.setText(newInfo.getTitle());
      tvDetail.setText(newInfo.getDetail());
      tvComment.setText(newInfo.getComment() + "跟帖");
      return view;
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
  }
}
```


### 完成










