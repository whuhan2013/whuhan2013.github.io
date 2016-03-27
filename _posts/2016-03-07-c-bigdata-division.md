---
layout: post
title: C语言实现大数据除法
date: 2016-2-3
categories: blog
tags: [数据结构]
description: C语言实现大数据除法
---

本题要求计算A/B，其中A是不超过1000位的正整数，B是1位正整数。你需要输出商数Q和余数R，使得A = B * Q + R成立。     

输入格式：

输入在1行中依次给出A和B，中间以1空格分隔。

输出格式：

在1行中依次输出Q和R，中间以1空格分隔。

输入样例：       

123456789050987654321 7    

输出样例：    

17636684150141093474 3     


代码实现如下      

```
#include "stdio.h"
#include "stdlib.h"
#include "string.h"
#define MaxSize 1000
void main()
{
  char a[MaxSize];
  int b[MaxSize];
  int c[MaxSize];
  int i;
  int n;
  int x=0;
  int lena=0;
  int lenc=1;
  memset(a,0,sizeof(a));
  memset(b,0,sizeof(b));
  memset(c,0,sizeof(c));
  scanf("%s",a);
  scanf("%d",&n);
  for(i=0;i<strlen(a);i++)
  {
    b[i+1]=a[i]-'0';
  }
  lena=strlen(a);
  for(i=1;i<=lena;i++)
  {
    c[i]=(x*10+b[i])/n;
    x=(x*10+b[i])%n;
  }

  while (c[lenc]==0&&lenc<lena)
  {
    lenc++;
  }

  c[0]=lena-lenc+1;
  for(i=1;i<=c[0];i++)
  {
    c[i]=c[lenc];
    lenc++;
  }
  for(i=1;i<=c[0];i++)
  {
    printf("%d",c[i]);
  }
  printf(" %d",x);
  system("pause");

}
```     

### 运行效果
![结果](http://img.blog.csdn.net/20160307192127340)










