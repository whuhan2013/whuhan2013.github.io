---
layout: post
title: 数据结构之二叉树的遍历
date: 2016-2-6
categories: blog
tags: [数据结构]
description: 数据结构之二叉树的遍历
---

**二叉树的遍历分为前序遍历，中序遍历，后序遍历，层序遍历** 

在本文中，前三种由递归实现，层序遍历由队列实现。 


```
#include "stdio.h"
#include "stdlib.h"
#include "windows.h"
typedef struct Node
{
  char data;
  struct Node *Left;
  struct Node *Right;
  struct Node *Next;
}BT;

typedef struct { /* 链队列结构 */
  BT *rear; /* 指向队尾结点 */
  BT *front; /* 指向队头结点 */
} LinkQueue;
//入队
LinkQueue* AddQuee(LinkQueue *PtrL,BT* item)
{
  BT *node;
  node=PtrL->rear;
  if (PtrL->front==NULL)
  {
    BT *q=(BT*)malloc(sizeof(BT));
    q->Left=item->Left;
    q->Right=item->Right;
    q->Next=NULL;
    q->data=item->data;
    PtrL->front=q;
    PtrL->rear=q;
    return PtrL;
  }
  else
  {
    BT *q=(BT*)malloc(sizeof(BT));
    q->Next=NULL;
    q->Left=item->Left;
    q->Right=item->Right;
    q->data=item->data;
    node->Next=q;
    PtrL->rear=q;
    return PtrL;
  }
}
//出队
BT* DeleteQ ( LinkQueue *PtrQ )
{
  BT *firstNode;
  //BT* NodeItem;

  if (PtrQ->front==NULL)
  {
    printf("queue is empty");
    return NULL;
  }

  firstNode=PtrQ->front;
  if (PtrQ->front==PtrQ->rear)
  {
    PtrQ->front=PtrQ->rear=NULL;
  }else
  {
    PtrQ->front=PtrQ->front->Next;
  }

  //NodeItem->data=firstNode->data;
  //free(firstNode);
  return firstNode;
}
//判断是否为空
int isempty(LinkQueue *PtrL)
{
  if (PtrL->rear==NULL)
  {
    return 1;
  }else
  {
    return 0;
  }
}

BT *CreateBiTree()
{
  char ch;
  BT *T;
  printf("please enter tree node:");
  scanf("%c",&ch);
  if (ch=='#')
  {
    T=NULL;
  }else
  {
    T=(BT*)malloc(sizeof(BT));
    T->data=ch;
    T->Left=CreateBiTree();
    T->Right=CreateBiTree();
  }

  return T;
}

//先序遍历
void PreOrderTraversal( BT* tree)
{
  if (tree)
  {
    printf("%c ",tree->data);
    PreOrderTraversal(tree->Left);
    PreOrderTraversal(tree->Right);
  }
}

//中序遍历
void InOrderTraversal(BT* tree)
{
  if (tree)
  {

    PreOrderTraversal(tree->Left);
    printf("%c ",tree->data);
    PreOrderTraversal(tree->Right);
  }
}

//后序遍历
void PostOderTraversal(BT *tree)
{
  if (tree)
  {

    PostOderTraversal(tree->Left);

    PostOderTraversal(tree->Right);
    printf("%c ",tree->data);
  }
}
//层序遍历
void LevelOrderTraversal(BT *tree)
{
  BT *bt;
  LinkQueue *q=(LinkQueue*)malloc(sizeof(LinkQueue));
  q->front=NULL;
  q->rear=NULL;
  if (!tree)
  {
    return;
  }
  AddQuee(q,tree);

  while(isempty(q)==0)
  {
    bt=DeleteQ(q);
    printf("%c ",bt->data);
    if (bt->Left) AddQuee(q,bt->Left);
    if (bt->Right) AddQuee(q,bt->Right);

  }

}

int PostOrderGetHeight( BT* tree )
{
  int HL, HR, MaxH;
  if( tree ) {
    HL = PostOrderGetHeight(tree->Left); /*求左子树的深度*/
    HR = PostOrderGetHeight(tree->Right); /*求右子树的深度*/
    MaxH =(HL> HR)? HL : HR;/*取左右子树较大的深度*/
    return ( MaxH + 1 ); /*返回树的深度*/
  }
  else return 0; /* 空树深度为0 */
}

void main()
{
  BT *t;
  int a;
  t=CreateBiTree();
  printf("\n1.PreOrderTraversal\n");
  printf("2.MidOrderTraversal\n");
  printf("3.PostOrderTraversal\n");
  printf("4.EXIT\n");
  printf("5.LevelOrderTraversal\n");
  printf("6.show the hieght of the tree");
  while (1)
  {
    printf("please enter your order:");
    scanf("%d",&a);
    switch (a)
    {
    case 1:
      PreOrderTraversal(t);
      break;
    case 2:
      InOrderTraversal(t);
      break;
    case 3:
      PostOderTraversal(t);
      break;
    case 4:
      exit(0);
      break;
    case 5:
      LevelOrderTraversal(t);
      break;
    case 6:
      printf("%d",PostOrderGetHeight(t));
      break;
    default:
      break;
    }
  }
}
```

### 运行结果    

![结果](http://img.blog.csdn.net/20160306201810586)


**C++实现二叉树的遍历** 


```
#include "iostream"
#include "stack"
#include "queue"


using namespace std;
 
//二叉树结点
typedef struct BiTNode{
    //数据
    char data;
    //左右孩子指针
    struct BiTNode *lchild,*rchild;
}BiTNode,*BiTree;
 
//按先序序列创建二叉树
int CreateBiTree(BiTree &T){
    char data;
    //按先序次序输入二叉树中结点的值（一个字符），‘#’表示空树
    scanf("%c",&data);
    if(data == '#'){
        T = NULL;
    }
    else{
        T = (BiTree)malloc(sizeof(BiTNode));
        //生成根结点
        T->data = data;
        //构造左子树
        CreateBiTree(T->lchild);
        //构造右子树
        CreateBiTree(T->rchild);
    }
    return 0;
}
//输出
void Visit(BiTree T){
    if(T->data != '#'){
        printf("%c ",T->data);
    }
}
//先序遍历
void PreOrder(BiTree T){
    if(T != NULL){
        //访问根节点
        Visit(T);
        //访问左子结点
        PreOrder(T->lchild);
        //访问右子结点
        PreOrder(T->rchild);
    }
}
//中序遍历  
void InOrder(BiTree T){  
    if(T != NULL){  
        //访问左子结点  
        InOrder(T->lchild);  
        //访问根节点  
        Visit(T);  
        //访问右子结点  
        InOrder(T->rchild);  
    }  
}  
//后序遍历
void PostOrder(BiTree T){
    if(T != NULL){
        //访问左子结点
        PostOrder(T->lchild);
        //访问右子结点
        PostOrder(T->rchild);
        //访问根节点
        Visit(T);
    }
}
/* 先序遍历(非递归)
   思路：访问T->data后，将T入栈，遍历左子树；遍历完左子树返回时，栈顶元素应为T，出栈，再先序遍历T的右子树。
*/
void PreOrder2(BiTree T){
    stack<BiTree> stack;
    //p是遍历指针
    BiTree p = T;
    //栈不空或者p不空时循环
    while(p || !stack.empty()){
        if(p != NULL){
            //存入栈中
            stack.push(p);
            //访问根节点
            printf("%c ",p->data);
            //遍历左子树
            p = p->lchild;
        }
        else{
            //退栈
            p = stack.top();
            stack.pop();
            //访问右子树
            p = p->rchild;
        }
    }//while
}
/* 中序遍历(非递归)
   思路：T是要遍历树的根指针，中序遍历要求在遍历完左子树后，访问根，再遍历右子树。
         先将T入栈，遍历左子树；遍历完左子树返回时，栈顶元素应为T，出栈，访问T->data，再中序遍历T的右子树。
*/
void InOrder2(BiTree T){
    stack<BiTree> stack;
    //p是遍历指针
    BiTree p = T;
    //栈不空或者p不空时循环
    while(p || !stack.empty()){
        if(p != NULL){
            //存入栈中
            stack.push(p);
            //遍历左子树
            p = p->lchild;
        }
        else{
            //退栈，访问根节点
            p = stack.top();
            printf("%c ",p->data);
            stack.pop();
            //访问右子树
            p = p->rchild;
        }
    }//while
}
 
//后序遍历(非递归)
typedef struct BiTNodePost{
    BiTree biTree;
    char tag;
}BiTNodePost,*BiTreePost;
 
void PostOrder2(BiTree T){
    stack<BiTreePost> stack;
    //p是遍历指针
    BiTree p = T;
    BiTreePost BT;
    //栈不空或者p不空时循环
    while(p != NULL || !stack.empty()){
        //遍历左子树
        while(p != NULL){
            BT = (BiTreePost)malloc(sizeof(BiTNodePost));
            BT->biTree = p;
            //访问过左子树
            BT->tag = 'L';
            stack.push(BT);
            p = p->lchild;
        }
        //左右子树访问完毕访问根节点
        while(!stack.empty() && (stack.top())->tag == 'R'){
            BT = stack.top();
            //退栈
            stack.pop();
            BT->biTree;
            printf("%c ",BT->biTree->data);
        }
        //遍历右子树
        if(!stack.empty()){
            BT = stack.top();
            //访问过右子树
            BT->tag = 'R';
            p = BT->biTree;
            p = p->rchild;
        }
    }//while
}
//层次遍历
void LevelOrder(BiTree T){
    BiTree p = T;
    //队列
  
    queue<BiTree> queue;
  
    //根节点入队
    queue.push(p);
    //队列不空循环
    while(!queue.empty()){
        //对头元素出队
        p = queue.front();
        //访问p指向的结点
        printf("%c ",p->data);
        //退出队列
        queue.pop();
        //左子树不空，将左子树入队
        if(p->lchild != NULL){
            queue.push(p->lchild);
        }
        //右子树不空，将右子树入队
        if(p->rchild != NULL){
            queue.push(p->rchild);
        }
    }
}
int main()
{
    BiTree T;
    CreateBiTree(T);
    printf("先序遍历：\n");
    PreOrder(T);
    printf("\n");
    printf("先序遍历(非递归)：\n");
    PreOrder2(T);
    printf("\n");
    printf("中序遍历：\n");
    InOrder(T);
    printf("\n");
    printf("中序遍历(非递归)：\n");
    InOrder2(T);
    printf("\n");
    printf("后序遍历：\n");
    PostOrder(T);
    printf("\n");
    printf("后序遍历(非递归)：\n");
    PostOrder2(T);
    printf("\n");
    printf("层次遍历：\n");
    LevelOrder(T);
    printf("\n");
  system("pause");
    return 0;
}
```


![这里写图片描述](http://img.blog.csdn.net/20160613200939472)