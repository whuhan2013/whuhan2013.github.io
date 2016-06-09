---
layout: post
title: JavaScript解析List数据
date: 2016-6-09
categories: blog
tags: [javascript]
description: JavaScript解析List数据
---   

### 主要步骤

1. 在服务器端将List转化成json返回 
2. 在客户端解析json

### 其他一些内容 

1. js获取session中的数据 
2. 通过web.xml设置index

### 返回json数据  

```
public void doGet(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

    response.setContentType("text/html");

        response.setCharacterEncoding("UTF-8");

        PrintWriter out = response.getWriter();
        PoliceDao policedao=new PoliceDao();
        
        System.out.println("requesttHhhhhhheerrr");
        List<LegalCase> legalcases = policedao.queryAll();

        StringBuffer sb = new StringBuffer();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

        sb.append('[');

        for (LegalCase legalcase: legalcases) {

                sb.append('{').append("\"id\":").append("\""+legalcase.getId()+"\"").append(",");   
                sb.append("\"updatetime\":").append("\""+sdf.format(legalcase.getUpdatetime())+"\"").append(",");
                sb.append("\"name\":").append("\""+legalcase.getName()+"\"").append(",");
                sb.append("\"telephone\":").append("\""+legalcase.getTelephone()+"\"").append(",");
                sb.append("\"address\":").append("\""+legalcase.getAddress()+"\"").append(",");
                sb.append("\"latitude\":").append("\""+legalcase.getLatitude()+"\"").append(",");
                sb.append("\"longtitude\":").append("\""+legalcase.getLongtitude()+"\"").append(",");
                sb.append("\"description\":").append("\""+legalcase.getDescription()+"\"").append(",");
                sb.append("\"dealed\":").append("\""+legalcase.getDealed()+"\"").append(",");
                sb.append("\"policeid\":").append(legalcase.getPoliceid());
                sb.append('}').append(",");
        }

        sb.deleteCharAt(sb.length() - 1);

        sb.append(']');

        out.write(new String(sb));

        out.flush();

        out.close();

  }
```


### 在客户端解析 


**方法1**

```
$.ajax({
        url : "QueryAll",
        type : "post",
        success : function(data) {
          alert(data);
          var obj = $.parseJSON(data);
          $.each(obj, function(id, val) {
            alert(id)
            alert(val.name);
          });
        },

        error : function(json) {
          alert("json=error");
          return false;
        }
      });
```

**方法二** 

```
$.ajax({
        url : "QueryAll",
        type : "post",
        success : function(data) {
          //alert(data);
          var obj = $.parseJSON(data);
          var List=obj;
    for ( var student in List) { //第二层循环取list中的对象 
            //alert(List[student].id);
            //alert(List[student].name);
            //alert("123");
            longtitude=List[student].longtitude;
            latitude=List[student].latitude;
            caseID[temp]=List[student].id;
            policeID[temp]=List[student].policeid;
            caseState[temp]=List[student].dealed;
            
            createcase();
            temp++;
          }
        },

        error : function(json) {
          alert("json=error");
          return false;
        }
      });
```

**参考链接**

[js解析json读取List中的实体对象示例代码 - JavaScript技巧 - 大学IT网](http://www.daxueit.com/article/2677.html)

### js获取session中的数据

```
var s="<%=session.getAttribute("caselist")%>"; 
```


### 设置index页面


```
<welcome-file-list>
  <welcome-file>
  GetAllServlet
  </welcome-file>
  </welcome-file-list>
```


### 完成