---
layout: post
title: 反转链表实现
date: 2016-3-7
categories: blog
tags: [数据结构]
description: 反转链表实现
---

### 题目      

给定一个常数K以及一个单链表L，请编写程序将L中每K个结点反转。例如：给定L为1→2→3→4→5→6，K为3，则输出应该为3→2→1→6→5→4；如果K为4，则输出应该为4→3→2→1→5→6，即最后不到K个元素不反转。

输入格式：

每个输入包含1个测试用例。每个测试用例第1行给出第1个结点的地址、结点总个数正整数N(<= 105)、以及正整数K(<=N)，即要求反转的子链结点的个数。结点的地址是5位非负整数，NULL地址用-1表示。

接下来有N行，每行格式为：

Address Data Next

其中Address是结点地址，Data是该结点保存的整数数据，Next是下一结点的地址。

输出格式：

对每个测试用例，顺序输出反转后的链表，其上每个结点占一行，格式与输入相同。

输入样例：
00100 6 4
00000 4 99999
00100 1 12309
68237 6 -1
33218 3 00000
99999 5 68237
12309 2 33218
输出样例：
00000 4 33218
33218 3 12309
12309 2 00100
00100 1 99999
99999 5 68237
68237 6 -1

### 实现

```
#include <iostream>
using namespace std;
int main() {
  int first, k, n;
  cin >>first>>n>> k;
  int temp;
  int *data=new int[100005];
  int *next=new int[100005];
  for (int i = 0; i < n; i++) {
    cin >> temp;
    cin >> data[temp];
    cin >> next[temp];
  }

  int *list=new int[100005];
  int sum = 0;
  while (first != -1) {
    list[sum++] = first;
    first = next[first];
  }
  int *result=new int[100005];
  for (int i = 0; i < sum; i++) {
    result[i] = list[i];
  }

  for (int i = 0; i < (sum - sum % k); i++) {
    result[i] = list[i / k * k + k - 1 - i % k];
  }

  for (int i = 0; i < sum - 1; i++)
    printf("%05d %d %05d\n", result[i], data[result[i]], result[i + 1]);
  printf("%05d %d -1", result[sum - 1], data[result[sum - 1]]);
  
  system("pause");
  free(data);
  free(next);
  free(list);
  free(result);
  return 0;
}
```

### 运行结果    

![结果](http://img.blog.csdn.net/20160328085104830)










