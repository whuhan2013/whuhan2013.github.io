---
layout: post
title: Java常用工具
date: 2016-6-20
categories: blog
tags: [java]
description: Java常用工具
---

**本文包括以下内容**  

1. Commons Collections之Bag的使用 


### Bag的使用 

Bag继承自Collection接口，定义了一个集合，该集合会记录对象在集合中出现的次数

**定义** 

```
private Bag productList = new HashBag();
```

**使用**  

```
 final int pnum = productList.getCount(food);
            if(pnum == 0){
                isShow(holder, View.GONE);//在没有选中的商品的时候，隐藏减少按钮
            }else if (pnum>0) {
                isShow(holder, View.VISIBLE);//显示减少按钮
            }
            holder.tvOrderNum.setText(pnum+"");
            //减少按钮
            holder.imgOrderSub.setOnClickListener(new View.OnClickListener() {

                @Override
                public void onClick(View v) {
                    productList.remove(food, 1);
                    foodList.updateBottomStatus(getTotalPrice(), getTotalNumber());

                    MyAdapter.this.notifyDataSetChanged();
                }
            });
            //增加
            holder.imgOrderAdd.setOnClickListener(new View.OnClickListener() {

                @Override
                public void onClick(View v) {
                    productList.add(food);
                    foodList.updateBottomStatus(getTotalPrice(), getTotalNumber());
                    MyAdapter.this.notifyDataSetChanged();
                }
            });
```


**遍历** 

```
 /**
         * 获取总金额
         * @return
         */
        public double getTotalPrice(){
            double totalProce = 0;
            if(productList.size() == 0){
                return totalProce;
            }
            for(Iterator<?> itr = productList.uniqueSet().iterator();itr.hasNext();){
                Food product = (Food)itr.next();
                totalProce += product.getPrice()*productList.getCount(product);
            }
            return totalProce;
        }

        /**
         * 获取总数量
         * @return
         */
        public int getTotalNumber(){
            int totalNumber = 0;
            if(productList.size() == 0){
                return totalNumber;
            }
            for(Iterator<?> itr = productList.uniqueSet().iterator();itr.hasNext();){
                Food product = (Food)itr.next();
                totalNumber +=productList.getCount(product);
            }
            return totalNumber;
        }
```


**参考链接** 

[Commons Collections - - ITeye技术网站](http://thaim.iteye.com/blog/1320826)


