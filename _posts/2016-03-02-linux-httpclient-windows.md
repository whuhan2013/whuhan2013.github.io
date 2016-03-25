---
layout: post
title: 利用HttpClient实现服务器与客户端通信
date: 2016-3-02
categories: blog
tags: [linux]
description: 通告一下，我已不再每天写千字文，准备采用以下的方法进行练习，由于文章篇幅较长，链接较多，建议到简书或博客进行阅读。
---

### 本篇主要讲解了利用HttpClient实现 windows主机与linux服务器的通信与传递数据,HttpClient代码，服务器端配置

### 系统和安装软件
1. ubuntu 14.04 64位系统  
2. sudo apt-get install apache2 sqlite3 libsqlite3-dev  





### 配置apache 支持cgi

- 配置目录 /etc/apache2  

- html页面目录 /var/www/html  

- cgi-bin目录  /usr/lib/cgi-bin  

- 日志文件   /var/log/apache2  


1. /etc/apache2/mods-enable里增加支持cgi的mod  
      cd /etc/apache2/mods-enabled  

      sudo ln -s ../mods-available/cgid.conf  
      
      sudo ln -s ../mods-available/cgid.load  
      
      sudo ln -s ../mods-available/cgi.load  
      


2. 编辑cgi代码:   
      /usr/lib/cgi-bin/setScore.c  
   
      sudo gcc /usr/lib/cgi-bin/setScore.c -o /usr/lib/cgi-bin/setScore.cgi
   
3. 建立数据库


   sudo sqlite3 /var/tank/tank.db  
   create table tscore (id integer primary key autoincrement, username varchar(32) unique not null, totalscore integer not null, score integer not null);
   

4. 修改数据库文件的权限  
   sudo chmod 777 /var/tank -R  
   sudo chmod www-data:www-data /var/tank -R    
### CGI代码如下，写数据库与读数据库并且向网页打印返回
```C++
#include <stdio.h>
#include <sqlite3.h>
#include <string.h>
#include <stdlib.h>
int selectCallback(void* arg,int argc,char** argv,char** argvv)
{
	//argv[0] id
	//argv[1] username
	//argv[2] totalscore
	//argv[3] score

	// username&totalscore&score&....
	printf("%s&%s&%s&", argv[1], argv[2], argv[3]);
	return 0;
}

int main()
{
	printf("Content-type:text/html\n\n");
#if 0
	printf("This is cocos cgi-test\n");
	// 打印环境变量
	extern char** environ;
	int i;
	for(i=0; ;++i)
	{
		if(environ[i])
			printf("%s\n<br>", environ[i]);
		else
			break;
	}
#endif

	// set Score to Database
	char* queryString = getenv("QUERY_STRING");
	if(queryString == NULL)
	{
		printf("Err: queryString is NULL");
		return 0;
	}

	// totalscore=%d&score=%d&user=user%d
	// 获取参数
	char* totalScore = strtok(queryString, "&");
	char* score = strtok(NULL, "&");
	char* username = strtok(NULL, "&");

	strtok(totalScore, "=");
	totalScore = strtok(NULL, "=");

	strtok(score, "=");
	score = strtok(NULL, "=");

	strtok(username, "=");
	username = strtok(NULL, "=");

	// 写数据库
	sqlite3* db;
	int ret = sqlite3_open("/home/jjx/tank.db", &db);
	if(ret != SQLITE_OK)
	{
		printf("open database error: %s", sqlite3_errstr(sqlite3_errcode(db)));
		return 0;
	}

	char sql[2048];
	sprintf(sql, "insert into tscore (username, totalscore, score) values ('%s', %s, %s)", 
		username, totalScore, score);
	ret = sqlite3_exec(db, sql, NULL, NULL, NULL);
#if 0
	printf("%s<br>", sql);
	return 0;
#endif
	if(ret != SQLITE_OK)
	{
		printf("insert data error: %s", sqlite3_errstr(sqlite3_errcode(db)));
		sqlite3_close(db);
		return 0;
	}

	sprintf(sql, "select * from tscore order by totalscore desc limit 10");
	ret = sqlite3_exec(db, sql, selectCallback, NULL, NULL);
	if(ret != SQLITE_OK)
	{
		printf("select data error: %s", sqlite3_errstr(sqlite3_errcode(db)));
		sqlite3_close(db);
		return 0;
	}

	sqlite3_close(db);

	return 0;
}
```




### 服务器端配置完成


### 客户端发送数据代码实现



```
/**
 * 
* <p>Title: 客户端发送数据</p>
* <p>Description: </p>
* <p>Company: whu</p> 
* @author 江军祥
* @date 2016-3-25上午9:09:01
 */
	//send http request ,send total score and score to server
	int totalScore=CCUserDefault::sharedUserDefault()->getIntegerForKey("TotalScore");
	int score=CCUserDefault::sharedUserDefault()->getIntegerForKey("Score");
	int userid=CCRANDOM_0_1()*100;
	char url[2048];
	sprintf(url,"http://192.168.226.129/cgi-bin/setScore.cgi?totalscore=%d&score=%d&user=user%d",totalScore,score,userid);
	CCHttpClient *client=CCHttpClient::getInstance();
	CCHttpRequest *request=new CCHttpRequest;
	request->setUrl(url);
	request->setRequestType(CCHttpRequest::kHttpGet);
	request->setResponseCallback(this,httpresponse_selector(LayerScore::HttpResponse));
	client->send(request);
	request->release();
```

                
                
### 客户端接收数据
```
//receive data from server
//json data is most common
if (!response->isSucceed())
{
	CCLOG("request error:%s",response->getErrorBuffer());
	return;
}

std::vector<char>* data=response->getResponseData();
std::vector<char>::iterator it;
std::string str;
for(it=data->begin();it!=data->end();it++)
{
	str.push_back(*it); 
}

char *p=new char[str.size()+1];
strcpy(p,str.c_str());
p[strlen(p)-1]=0;
int index=0;
char *username=strtok(p,"&"); 
char *totalScore;
char *score;
char buf[1024];
while (username)
{
	totalScore=strtok(NULL,"&");
	score=strtok(NULL,"&");
	CCLOG("********%s,%s,%s********\n",username,totalScore,score);
	//put data into labels
	CCLabelTTF *label=(CCLabelTTF*)getChildByTag(1000+index++);
	sprintf(buf,"%s:%s:%s",username,totalScore,score);
	label->setString(buf);
	username=strtok(NULL,"&");
}
delete []p;
```
### HttpClient实现windwos主机与linux服务器通信并传递信息

### 以上是以DOGet方法，将参数设置在URL中以到达传递参数的作用，下面使用DOPost方法向服务器端上传图片
### 客户端上传代码
```
/**
 * 
* <p>Title: doPost</p>
* <p>Description: </p>
* <p>Company: whu</p> 
* @author 江军祥
* @date 2016-3-25上午9:09:01
 */
      CCHttpClient* client = CCHttpClient::getInstance();

        CCHttpRequest* req = new CCHttpRequest;
        req->setUrl("http://192.168.226.129/cgi-bin/posttest.cgi");
        req->setRequestType(CCHttpRequest::kHttpPost);
        req->setResponseCallback(this, httpresponse_selector(T24HttpClient::HttpResponse));

        char buf[8192];

        FILE* f = fopen("woman.png", "rb");
        int len = fread(buf, 1, 8192, f);
        fclose(f);
        CCLog("len=%d\n", len);

        req->setRequestData((const char*)buf, len);
        //req->setRequestData(buf,sizeof(buf));
        client->send(req);
        req->release();


        return true;
        
```

### 服务器端接收代码
```
#include <stdio.h>
#include <sqlite3.h>
#include <string.h>
#include <stdlib.h>
int main()
{
        printf("Content-type:text/html\n\n");

        
        int fd=creat("/home/jjx/tank/aaa.png");
        int len=atoi(getenv("CONTENT_LENGTH"));
        char *buf=malloc(len);
        fread(buf,len,1,stdin);
        write(fd,buf,len);
        close(fd);
        free(buf);

        printf("%s\n<br>",buf);
        return 0;
}
```

### 上传结束，可以在相应文件路径下看到图片












