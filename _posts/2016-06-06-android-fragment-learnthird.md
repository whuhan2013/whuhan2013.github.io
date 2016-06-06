---
layout: post
title: Android之Fragment（三）
date: 2016-6-6
categories: blog
tags: [android]
description: Android之Fragment
---


### 本文主要包括以下内容  

1. fragment中调用findViewById 

### fragment中调用findViewById       


```
public class CompanyListFragment  extends Fragment {
 
　　private Activity activity;
 　   private ListView companyListView;
 
　　@Override
 　　public View onCreateView(LayoutInflater infalter, ViewGroup container, Bundle savedInstanceState) {
　　　　View view = infalter.inflate(R.layout.companylist_for_sport, container, false);
　　　　companyListView = (ListView)view.findViewById(R.id.companyListView);
　　　　return view;
 　　}
 
}
```

 
**注意1:**  是在 onCreateView 中
 
**注意2:**  是 view.findViewById 而不是 activity.findViewById          


**也可以在BaseFragment中使用如下方法** 

```
 protected <T extends View> T $(int id) {
        return (T) mContentView.findViewById(id);
    }
```



