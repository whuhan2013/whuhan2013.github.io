---
layout: post
title: C++实现大数据乘法
date: 2016-2-3
categories: blog
tags: [数据结构]
description: C++实现大数据乘法
---

 1. 结构体定义与封装        


```
struct bigdatacom
{
private :
  char dataa[100];
  char datab[100];
public :
  void init(const char *str1,const char *str2)
  {
    std::cout<<typeid(*this).name()<<std::endl;
    strcpy(this->dataa,str1);
    strcpy(this->datab,str2);
  }
  char * getbigdata()
   {
     
  int lengtha = strlen(dataa);
  int lengthb = strlen(datab);
  int *pres = (int *)malloc(sizeof(int)*(lengtha + lengthb));
  memset(pres, 0, sizeof(int)*(lengtha + lengthb));//初始化
  //累乘
  for (int i = 0; i < lengtha;i++)
  {
    for (int j = 0; j < lengthb;j++)
    {
      pres[i+j+1]+=(dataa[i] - '0')*(datab[j] - '0');
    }
  }
  //进位
  for (int i = lengtha + lengthb-1;i>=0;i--)
  {
    if (pres[i]>=10)//进位
    {
      pres[i - 1] += pres[i] / 10;//进位
      pres[i] %= 10;//取出个位数
    }

  }
  int i = 0;
  while (pres[i]==0)
  {
    i++;//恰好不为0的位置
  }
  char *lastres = (char*)malloc(sizeof(char)*(lengtha + lengthb));
  int j;
  for (j = 0; j < lengtha + lengthb; j++, i++)
  {
    lastres[j] = pres[i] + '0';
  }
  lastres[j] = '\0';
  //printf("last结果=%s",lastres);

  return lastres;


}

};
```       

2. main函数    

 

```
void main()
{
  bigdatacom big1;      //C语言中结构体定义必须带struct,C++中不必
  big1.init("234546869966543","45645663453223323423423");
  int n=strlen(big1.getbigdata());
  string s=big1.getbigdata();
  s=s.substr(0,n-1);
  std::cout<<s<<std::endl;
  //big1.getbigdata
  system("pause");
}
```

 3. 运行结果
   结果后面会多出一个-号，可能是因为栈溢出或者某个地方初始化错误，利用了C++的substr方法处理了，在其中遇到一个问题，就是已经引入了
   #include <string>       
String仍旧显示未定义的标识符，原因是没有写using namespace std;











