---
layout: post
title: 数据结构常用工具类
date: 2016-8-3
categories: blog
tags: [数据结构]
description: 工具类
---   

**C++分割字符串**          

```
//字符串分割函数
std::vector<std::string> split(std::string str,std::string pattern)
{
    std::string::size_type pos;
    std::vector<std::string> result;
    str+=pattern;//扩展字符串以方便操作
    int size=str.size();

    for(int i=0; i<size; i++)
    {
        pos=str.find(pattern,i);
        if(pos<size)
        {
            std::string s=str.substr(i,pos-i);
            result.push_back(s);
            i=pos+pattern.size()-1;
        }
    }
    return result;
}
```

**字典序全排列实现**        

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


