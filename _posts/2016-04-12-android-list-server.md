---
layout: post
title: 安卓客户端向服务器发送List数据
date: 2016-4-12
categories: blog
tags: [网络编程,java]
description: 安卓客户端向服务器发送List数据
---

### 第一步： 

 首先写一个自定义的JavaBean，以UserInfo.java为例，需要实现对象序列化的接口，因为之后输出流对象需要实现输出可序列化的对象。不这样的话，后续时发送时会报异常

```
package xl.java.bean;  
  
import java.io.Serializable;  
  
/** 
 * 用户信息 
 * @author xl 2012-9-20 
 */  
public class UserInfo implements Serializable  
{  
  
    private static final long serialVersionUID = 1L;  
  
    /** 
     * 用户名 
     */  
    private String UserName;  
  
    /** 
     * 密码 
     */  
    private String Password;  
  
    /** 
     * 昵称 
     */  
    private String NickName;  
  
    /** 
     * QQ号 
     */  
    private int QQNumber;  
  
    /** 
     * 电话号 
     */  
    private String TelNumber;  
  
    /** 
     * 年龄 
     */  
    private int Age;  
  
    public String getUserName()  
    {  
        return UserName;  
    }  
  
    public void setUserName(String userName)  
    {  
        UserName = userName;  
    }  
  
    public String getPassword()  
    {  
        return Password;  
    }  
  
    public void setPassword(String password)  
    {  
        Password = password;  
    }  
  
    public String getNickName()  
    {  
        return NickName;  
    }  
  
    public void setNickName(String nickName)  
    {  
        NickName = nickName;  
    }  
  
    public int getQQNumber()  
    {  
        return QQNumber;  
    }  
  
    public void setQQNumber(int qQNumber)  
    {  
        QQNumber = qQNumber;  
    }  
  
    public String getTelNumber()  
    {  
        return TelNumber;  
    }  
  
    public void setTelNumber(String telNumber)  
    {  
        TelNumber = telNumber;  
    }  
  
    public int getAge()  
    {  
        return Age;  
    }  
  
    public void setAge(int age)  
    {  
        Age = age;  
    }  
}  
```    

### 注意，服务器与客户端的javabean包名必须一致，不然的话会报ClassNotFoundException异常。  


### 第二步：
编写客户端模拟发送数据的类SendData.java。中间一大段的连接处理，具体解释可参考：
[JDK中的URLConnection参数详解 - wlzf6296149的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/wlzf6296149/article/details/7998641)

```
package xl.java.send;  
  
import java.io.IOException;  
import java.io.InputStream;  
import java.io.ObjectOutputStream;  
import java.io.OutputStream;  
import java.net.HttpURLConnection;  
import java.net.URL;  
import java.net.URLConnection;  
import java.util.ArrayList;  
import java.util.List;  
  
import xl.java.bean.UserInfo;  
  
/** 
 * 模拟发送数据 
 * @author xl 2012-9-20 
 *  
 */  
public class SendData  
{  
  
    private static final String BASIC_URL_QUEST =  
            "http://192.168.1.1:8080/test/TestServlet";  
  
    public static void main(String[] args)  
    {  
        SendData senddata=new SendData();  
        try  
        {  
            senddata.sendDataToServer();  
        }  
        catch (IOException e)  
        {  
            e.printStackTrace();  
        }  
    }  
  
    /** 
     * 上传处理结果 
     *  
     * @throws IOException 
     *  
     */  
    private void sendDataToServer() throws IOException  
    {  
        //用于servlet判别请求，执行相应方法  
        String QuestId = "SubmitUserInfoList";  
        //模拟发送自定义类型的List数据  
        List<UserInfo> listdata = new ArrayList<UserInfo>();  
        for (int i = 0; i < 10; i++)  
        {  
            UserInfo li = new UserInfo();  
            li.setUserName("XL" + i);  
            li.setPassword("00000" + i);  
            li.setQQNumber(1234567 + i);  
            li.setTelNumber("15012344321" + i);  
            li.setNickName("xiaolang" + i);  
            li.setAge(18 + i);  
            listdata.add(li);  
        }  
  
        URL url = new URL(BASIC_URL_QUEST);  
        try  
        {  
            URLConnection con = url.openConnection();  
            HttpURLConnection httpUrlConnection = (HttpURLConnection) con;  
            httpUrlConnection.setUseCaches(false);  
            httpUrlConnection.setDoOutput(true);  
            httpUrlConnection.setDoInput(true);  
            httpUrlConnection.setRequestProperty("Content-type",  
                    "application/x-java-serialized-object");  
            //不设置这个默认为Get,服务器会没反应，不知道什么情况，  
            //纠结了很久，改成Post的话,servlet里的  
            //doPost方法就有反应了  
            httpUrlConnection.setRequestMethod("POST");  
            httpUrlConnection.connect();  
            OutputStream outStrm = httpUrlConnection.getOutputStream();  
            ObjectOutputStream oos = new ObjectOutputStream(outStrm);  
            //输出流第一段数据是QuestId的值  
            oos.writeObject(QuestId);  
            //第二段数据是List数据  
            oos.writeObject(listdata);      oos.flush();  
            oos.close();  
            InputStream inStrm = httpUrlConnection.getInputStream();  
            System.out.println("数据发送成功!");  
        }  
  
        catch (Exception e)  
        {  
            e.printStackTrace();  
        }  
    }  
}  
```    


### 编写servlet，接收数据  

```
import java.io.IOException;  
import java.io.InputStream;  
import java.io.ObjectInputStream;  
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.ResultSet;  
import java.sql.SQLException;  
import java.sql.Statement;  
import java.util.List;  
  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
  
import xl.java.bean.UserInfo;  
  
/** 
 * @author xl 2012-9-20 
 *  
 */  
public class TestServlet extends HttpServlet  
{  
  
    private static final long serialVersionUID = 1L;  
  
    private Connection mConnection = null;  
  
    private Statement mStatement = null;  
  
    private String QuestId = "";  
  
    private static final String SUBMIT_USERINFO_LIST = "SubmitUserInfoList";// 客户端提交到用户信息  
  
    protected void doPost(HttpServletRequest request,  
            HttpServletResponse response)  
    {  
        System.out.println("________---------doPost--------_____________");  
  
        try  
        {  
            // 链接数据库  
            Class.forName("org.gjt.mm.<a href="http://lib.csdn.net/base/14" class="replace_word" title="undefined" target="_blank" style="color: rgb(223, 52, 52); font-weight: bold;">mysql</a>.Driver").newInstance();  
            mConnection =  
                    DriverManager  
                            .getConnection("jdbc:mysql://localhost/test?user=root&password=123&useUnicode=true&characterEncoding=UTF-8");  
            mStatement =  
                    mConnection.createStatement(  
                            ResultSet.TYPE_SCROLL_INSENSITIVE,  
                            ResultSet.CONCUR_READ_ONLY);  
            // 如果不是通过URL的Get形式上传数据时，调用此方法，获取上传的list数据  
            getListDataByObjectInputStream(request, response);  
        }  
        catch (SQLException e)  
        {  
            e.printStackTrace();  
        }  
        catch (InstantiationException e)  
        {  
            e.printStackTrace();  
        }  
        catch (IllegalAccessException e)  
        {  
            e.printStackTrace();  
        }  
        catch (ClassNotFoundException e)  
        {  
            e.printStackTrace();  
        }  
        catch (IOException e)  
        {  
            e.printStackTrace();  
        }  
  
    }  
  
    /** 
     * 获取输入流中的数据 
     *  
     * @param request 
     * @param response 
     * @throws IOException 
     * @throws ClassNotFoundException 
     */  
    private void getListDataByObjectInputStream(HttpServletRequest request,  
            HttpServletResponse response) throws IOException,  
            ClassNotFoundException  
    {  
        System.out.println("---------getListDataByObjectInputStream--------");  
        response.setContentType("text/html");  
        InputStream inStream = request.getInputStream();  
        ObjectInputStream objInStream = new ObjectInputStream(inStream);  
        QuestId = (String) objInStream.readObject();  
        @SuppressWarnings("unchecked")  
        List<UserInfo> inList = (List<UserInfo>) objInStream.readObject();  
        if (QuestId.equals(SUBMIT_USERINFO_LIST))  
        {  
            System.out.println("QuestId.equals(SUBMIT_ORDER_LIST)");  
            submitOrderList(request, response, inList);  
        }  
        objInStream.close();  
        System.out.println("objInStream.close()");  
    }  
  
    /** 
     * @param request 
     * @param response 
     * @param inList 
     */  
    private void submitOrderList(HttpServletRequest request,  
            HttpServletResponse response, List<UserInfo> inList)  
    {  
        // 获取数据，插入数据库  
        for (UserInfo item : inList)  
        {  
            System.out.println("UserName=" + item.getUserName());  
            System.out.println("Password=" + item.getPassword());  
            System.out.println("NickName=" + item.getNickName());  
            System.out.println("QQNumber=" + item.getQQNumber());  
            System.out.println("TelNumber=" + item.getTelNumber());  
            System.out.println("Age=" + item.getAge() + "\n");  
        }  
        /** 
         * 插入数据库代码可以写在这.. 
         */  
    }  
}  
```   


### 最后：
 运行SendData.java文件,可看到控制台输出,根据前面的设定我们知道，客户端和服务端已经建立连接，并且数据成功发送给了服务端

### 参考链接：
[关于servlet服务端接收客户端发送的List<?>数据的问题 - wlzf6296149的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/wlzf6296149/article/details/8001433)














