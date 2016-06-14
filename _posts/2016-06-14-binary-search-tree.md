---
layout: post
title: 数据结构之二叉搜索树
date: 2016-6-14
categories: blog
tags: [数据结构]
description: 数据结构之二叉搜索树
---

**二叉搜索树**

一棵二叉树，可以为空；如果不为空，满足以下性质：              
1. 非空左子树的所有键值小于其根结点的键值。        
2. 非空右子树的所有键值大于其根结点的键值。           
3. 左、右子树都是二叉搜索树。      


**二叉搜索树的插入，删除，查找** 


**头文件**  


```
#include "iostream"
#include "string.h"
#include "sstream"


using namespace std;


//二叉查找树节点信息
template<class T>
class TreeNode
{
public:
  TreeNode():lson(NULL),rson(NULL),freq(1){}//初始化
  T data;//值
  unsigned int freq;//频率
  TreeNode* lson;//指向左儿子的坐标
  TreeNode* rson;//指向右儿子的坐标
};


//二叉查找树类的属性和方法声明
template<class T>
class BST
{
private:
  
  TreeNode<T>* root;//根节点
  void insertpri(TreeNode<T>* &node,T x);//插入
  TreeNode<T>* findpri(TreeNode<T>* node,T x);//

  void insubtree(TreeNode<T>* node);//中序遍历
  void Deletepri(TreeNode<T>* &node,T x);//删除
  TreeNode<T>* findMinpri(TreeNode<T>* node);
  TreeNode<T>* findMaxpri(TreeNode<T>* node);
public:
  BST():root(NULL){}
  void insert(T x);//插入接口
  TreeNode<T>* find(T x);//查找接口
  void Delete(T x);//删除接口
  void traversal();//遍历接口
  TreeNode<T>* findMin();
  TreeNode<T>* findMax();

};
```


**实现**  

```
#include "head.h"

template<class T>
void BST<T>::insertpri(TreeNode<T>* &node,T x)
{
  if(node==NULL)//如果节点为空,就在此节点处加入x信息
  {
    node=new TreeNode<T>();
    node->data=x;
    
    //cout<<x;
    return;
  }
  if (node->data>x)
  {
    insertpri(node->lson,x);
  }else if (node->data<x)
  {
    insertpri(node->rson,x);
  }else
  {
    node->freq++;
  }

}

//插入接口
template<class T>
void BST<T>::insert(T x)
{
  insertpri(root,x);
}


template<class T>
TreeNode<T>* BST<T>::findpri(TreeNode<T>* node,T x)
{

  if (node==NULL)
  {
    return NULL;
  }

  if (node->data>x)
  {
    return findpri(node->lson);
  }else if (node->data<x)
  {
    return findpri(node->rson);
  }else
  {
    return node;
  }
}

template<class T>
TreeNode<T>* BST<T>::find(T x)
{
  return findpri(root,x);
}

template<class T>
TreeNode<T>* BST<T>::findMinpri(TreeNode<T>* node)
{
  if (node==NULL)
  {
    return NULL;
  }
  
  if (node->lson!=NULL)
  {
    return findMinpri(node->lson);
  }else
  {
    return node;
  }
}

template<class T>
TreeNode<T>* BST<T>::findMin()
{

  return findMinpri(root);
}

template<class T>
void BST<T>::Deletepri(TreeNode<T>* &node,T x)
{
  if(node==NULL) return ;//没有找到值是x的节点
  if(x < node->data)
    Deletepri(node->lson,x);//如果x小于节点的值,就继续在节点的左子树中删除x
  else if(x > node->data)
    Deletepri(node->rson,x);//如果x大于节点的值,就继续在节点的右子树中删除x
  else//如果相等,此节点就是要删除的节点
  {
    if(node->lson&&node->rson)//此节点有两个儿子
    {
      TreeNode<T>* temp=node->rson;//temp指向节点的右儿子
      while(temp->lson!=NULL) temp=temp->lson;//找到右子树中值最小的节点
      //把右子树中最小节点的值赋值给本节点
      node->data=temp->data;
      node->freq=temp->freq;
      Deletepri(node->rson,temp->data);//删除右子树中最小值的节点
    }
    else//此节点有1个或0个儿子
    {
      TreeNode<T>* temp=node;
      if(node->lson==NULL)//有右儿子或者没有儿子
        node=node->rson;
      else if(node->rson==NULL)//有左儿子
        node=node->lson;
      delete(temp);

      
    }
  }
  return;

}

template<class T>
void BST<T>::Delete(T x)
{
  Deletepri(root,x);
}


//中序遍历函数
template<class T>
void BST<T>::insubtree(TreeNode<T>* node)
{
  if(node==NULL) return;
  insubtree(node->lson);//先遍历左子树
  cout<<node->data<<" ";//输出根节点
  insubtree(node->rson);//再遍历右子树
}
//中序遍历接口
template<class T>
void BST<T>::traversal()
{
  insubtree(root);
}


int main()
{
  BST<int>* bst=new BST<int>();
  int temp;
  int inputNumber;
  
  
  cout<<"1.插入数据\n2.查找数据\n3.查找最小\n4.删除指定数据\n5.中序遍历\n";
  bst->insert(66);
  bst->insert(55);
  bst->insert(77);
  bst->insert(44);
  while (true)
  {
    cout<<"请输入指令\n";
    cin>>temp;
    switch (temp)
    {
    case 1:
      cin>>inputNumber;
      bst->insert(inputNumber);
      break;
    case 2:
      break;
    case 3:
    
      cout<<"结果为:";
      cout<<bst->findMin()->data;
      break;
    case 4:
      cin>>inputNumber;
      bst->Delete(inputNumber);
      break;
    case 5:
      bst->traversal();
      break;
    default:
      break;
    }
  }
  system("pause");
  return 0;

}

```   

**参考链接**

[一步一步写二叉查找树 - C小加 - C++博客](http://www.cppblog.com/cxiaojia/archive/2012/08/09/186752.html)


**效果如下** 

![这里写图片描述](http://img.blog.csdn.net/20160614203342314)

