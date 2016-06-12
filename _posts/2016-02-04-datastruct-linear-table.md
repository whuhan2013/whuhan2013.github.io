---
layout: post
title: 数据结构之线性表
date: 2016-2-4
categories: blog
tags: [数据结构]
description: 数据结构之线性表
---


### 一、线性表的顺序存储实现，利用数组实现线性表   


```
typedef struct{
  int Data[MAXSIZE];
  int Last;
}List;

List *MakeEmpty()
{
  List *PtrL;
  PtrL=(List*)malloc(sizeof(List));
  PtrL->Last=-1;
  return PtrL;
}
int FindKth( int K, List* PtrL )  //根据位序K，返回相应元素
{
  return PtrL->Data[K-1];
}
int Find(int X, List *PtrL )//在线性表L中查找X的第一次出现位置；
{
  int i=0;
  while (i<=PtrL->Last&&PtrL->Data[i]!=X)
  {
    i++;
  }
  if (i>PtrL->Last)
  {
    return -1;
  }
  else
  {
    return i;
  }
}
void Insert(int X, int i, List *PtrL)//：在位序i前插入一个新元素X；
{
  int j;
  if (PtrL->Last==MAXSIZE-1)
  {
    printf("the list is full already");
  }
  if (i>=PtrL->Last+2)
  {
    printf("the insert position is invalid");
  }

  for (j = PtrL->Last; j >=i-1; j--)
  {
    PtrL->Data[j+1]=PtrL->Data[j];
  }
  PtrL->Data[i]=X;
  PtrL->Last++;
}
void Delete( int i, List *PtrL)//：删除指定位序i的元素；
{
  int j;
  if (i<1||i>PtrL->Last+1)
  {
    printf("the number you asked is not existed");
  }
  for (j=i;j<=PtrL->Last;j++)
  {
    PtrL->Data[j-1]=PtrL->Data[j];

  }

  PtrL->Last--;
}
int Length( List *PtrL )//：返回线性表L的长度n
{
  return PtrL->Last+1;
}
void showAll(List *PtrL)
{
  int i;
  for (i = 0; i <=PtrL->Last; i++)
  {
    printf("%d ",PtrL->Data[i]);
  }
}

void main1()
{
  int i;
  List *PtrL;
  int a,b;
  printf("1.create a empty list\n");
  printf("2.insert a number\n");
  printf("3.query a number first appear\n");
  printf("4.query a number by index\n");
  printf("5.delete a number\n");
  printf("6.show the length of the list\n");
  printf("7.showall\n");
  printf("8.exit\n");
  while (1)
  {
  printf("please enter you order:");
  scanf("%d",&i);
  switch (i)
  {
  case 1:
    PtrL=MakeEmpty();
    break;
  case 2:
    printf("please enter what and where do you want to inset :");
    scanf("%d%d",&a,&b);
    Insert(a,b,PtrL);
    break;
  case 3:
    printf("please enter what do you want to query:");
    scanf("%d",&a);
    Find(a,PtrL);
    break;
  case 4:
    printf("please enter what do you want to query:");
    scanf("%d",&a);
    FindKth(a,PtrL);
    break;
  case 5:
    printf("please enter which index do you want to delete:");
    scanf("%d",&a);
    Delete(a,PtrL);
    break;
  case 6:
    printf("the length is%d:",Length(PtrL));
    break;
  case 7:
    showAll(PtrL);
    break;
  case 8:
    exit(1);
  }
  }
}
```   

![顺序存储实现](http://img.blog.csdn.net/20160305145301924)

### 线性表的链式存储实现，利用链表实现   


```
typedef struct Node{
  int Data;
  struct Node *Next;
} List;

int Length(List *PtrL)
{
  List *p=PtrL;
  int j=0;
  while (p)
  {
    p=p->Next;
    j++;
  }

  return j;
}

List *FindKth( int K, List *PtrL)
{
  List *p=PtrL;
  int i=1;
  while (p!=NULL&&i<K)
  {
    p=p->Next;
    i++;
  }

  if (i==K)
  {
    return p;
  }
  else
  {
    return NULL;
  }

}

List *Find( int X, List *PtrL )
{
  List *p=PtrL;
  while (p&&p->Data!=X)
  {
    p=p->Next;
  }
  return p;
}

List *Insert( int X, int i, List *PtrL )
{
  List *p,*s;
  if (i==1)
  {
    s=(List*)malloc(sizeof(List));
    s->Data=X;
    s->Next=PtrL;
    return s;
  }

  p=FindKth(i-1,PtrL);
  if (p==NULL)
  {
    printf("the insert is invalid");
    return PtrL;
  }else
  {
    s=(List*)malloc(sizeof(List));
    s->Data=X;
    s->Next=p->Next;
    p->Next=s;
    return PtrL;
  }


}

List *Delete( int i, List *PtrL )
{
  List *s,*p;
  if (i==1)
  {
    s=PtrL;
    if (PtrL)
    {
      PtrL=PtrL->Next;

    }else
    {
      return NULL;
    }

    free(s);
    return PtrL;
  }

  p=FindKth(i-1,PtrL);
  if (p==NULL)
  {
    printf("the %d node is null",i-1);
  }else if (p->Next==NULL)
  {
    printf("the %d node is null",i);
  }

  s=p->Next;
  p->Next=s->Next;
  free(s);
  return PtrL;
}

void show(List *PtrL)
{
  List *P=PtrL;
  while (P)
  {
    printf("%d ",P->Data);
    P=P->Next;
  }
}

void main()
{
  List *PtrL=NULL;
  int x;
  int a,b;
  printf("1.insert a node\n");
  printf("2.delete a node\n");
  printf("3.query a node\n");
  printf("4.show all\n");
  printf("5,delete\n");
  while (1)
  {
    printf("\nplase enter your order:");
    scanf("%d",&x);
    switch (x)
    {
    case 1:
      printf("please enter what and where do you want to insert:");
      scanf("%d%d",&a,&b);
      PtrL=Insert(a,b,PtrL);
      break;
    case 2:
      printf("plEase enter which one do you want to delete:");
      scanf("%d",&a);
        PtrL=Delete(a,PtrL);
      break;
    case 3:
      printf("PLease enter which one do you want to query:");
      scanf("%d",&a);
      Find(a,PtrL);
      break;
    case 4:
      show(PtrL);
      break;
    case 5:
      exit(0);
      break;
    default:
      break;
    }
  }
}
```  


![链式存储实现](http://img.blog.csdn.net/20160305145328627)








