---
layout: post
title: 数据结构之字典序全排列
date: 2016-7-23
categories: blog
tags: [数据结构]
description: 字典序 
---

字典序法中，对于数字1、2、3......n的排列，不同排列的先后关系是从左到右逐个比较对应的数字的先后来决定的。例如对于5个数字的排列 12354和12345，排列12345在前，排列12354在后。按照这样的规定，5个数字的所有的排列中最前面的是12345，最后面的是 54321。

**字典序算法如下：**          
设P是1～n的一个全排列:p=p1p2......pn=p1p2......pj-1pjpj+1......pk-1pkpk+1......pn            
1）从排列的右端开始，找出第一个比右边数字小的数字的序号j（j从左端开始计算），即   j=max{i|pi<pi+1}     
2）在pj的右边的数字中，找出所有比pj大的数中最小的数字pk              
3）对换pj，pk                                  
4）再将pj+1......pk-1pkpk+1pn倒转得到排列p'=p1p2.....pj-1pjpn.....pk+1pkpk-1.....pj+1，这就是排列p的下一个排列。   

例如839647521是数字1～9的一个排列。从它生成下一个排列的步骤如下：           
自右至左找出排列中第一个比右边数字小的数字4          839647521               
在该数字后的数字中找出比4大的数中最小的一个5         839647521                     
将5与4交换                                           839657421            
将7421倒转                                           839651247         
所以839647521的下一个排列是839651247。 

#### 实现 

**递归法**      
递归的话就很简单了，以{1,2,3}为例，它的排列是：      
以1开头，后面接着{2,3}的全排列，  
以2开头，后面接着{1,3}的全排列，        
以3开头，后面接着{1,2}的全排列。               
代码如下：

```
#include<iostream>
#include<algorithm>

using namespace std;

int arry[3] = { 1,2,3 };

void Recursion(int s, int t)
{
	if (s == t)
		for_each(arry, arry + 3, [](int i) {printf("%d", i); }), printf("\n");
	else
	{
		for (int i = s; i <= t; i++)
		{
			swap(arry[i], arry[s]);
			Recursion(s + 1, t);
			swap(arry[i], arry[s]);
		}
	}
}

int main()
{

	Recursion(0, 2);

	return 0;
}
```

**排序中有重复的情况** 

```
#include<iostream>
#include<algorithm>

using namespace std;

int arry[3] = { 1,2,2 };

bool IsEqual(int s, int t)
{
	for (int i = s; i < t; i++)
		if (arry[i] == arry[t])
			return true;

	return false;
}

void Recursion(int s, int t)
{
	if (s == t)
		for_each(arry, arry + 3, [](int i) {printf("%d", i); }), printf("\n");
	else
	{
		for (int i = s; i <= t; i++)
		{
			if (!IsEqual(s, i))//不相等才能交换
			{
				swap(arry[i], arry[s]);
				Recursion(s + 1, t);
				swap(arry[i], arry[s]);
			}
		}
	}
}

int main()
{

	Recursion(0, 2);

	return 0;
}
```

**利用STL中的next_permutation方法**      

```
//简短的AC代码。调用了STL的next_permutation函数
 vector<string> Permutation(string str) {
        vector<string> answer;
        if(str.empty())
            return answer;       
        sort(str.begin(),str.end());
        do{
            answer.push_back(str);
        }
        while(next_permutation(str.begin(),str.end()));
        return answer;
}
```

**字符串字典排序**

```
class Solution {
public:
    vector<string> Permutation(string str) {
        //可以用递归来做
        vector<string> array;
        if(str.size()==0)
            return array;
        Permutation(array, str, 0);
        sort(array.begin(), array.end());
        return array;
    }
     
    void Permutation(vector<string> &array, string str, int begin)//遍历第begin位的所有可能性
    {
        if(begin==str.size()-1)
            array.push_back(str);
        for(int i=begin; i<=str.size()-1;i++)
        {
            if(i!=begin && str[i]==str[begin])//有重复字符时，跳过
                continue;
            swap(str[i], str[begin]);//当i==begin时，也要遍历其后面的所有字符；
                                    //当i!=begin时，先交换，使第begin位取到不同的可能字符，再遍历后面的字符
            Permutation(array, str, begin+1);//遍历其后面的所有字符；
             
            swap(str[i], str[begin]);//为了防止重复的情况，还需要将begin处的元素重新换回来
             
            /*举例来说“abca”，为什么使用了两次swap函数
                交换时是a与b交换，遍历；
                交换时是a与c交换，遍历；（使用一次swap时，是b与c交换）
                交换时是a与a不交换；
                */
        }
    }
};
```


**参考链接：**             
[全排列的实现方法--递归&字典序 - 半壶老酒 - 博客频道 - CSDN.NET](http://blog.csdn.net/laojiu_/article/details/51115352)

[字典序全排列 - zwb8848happy的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/zwb8848happy/article/details/7318160)



