---
layout: post
title: 数据结构之堆栈与队列
date: 2016-2-5
categories: blog
tags: [数据结构]
description: 数据结构之堆栈与队列
---

**堆栈与队列是两种重要的基础数据结构**           

一个是先入后出，一个是先入先出，有着广泛的应用，本文分别使用数组与链表实现堆栈与队列    

**顺序存储方式实现堆栈**       


```
#define MaxSize 20
#define ERROR -1
typedef struct {
  int Data[MaxSize];
  int Top;
} Stack;

Stack *CreateStack( int maxSize)//： 生成空堆栈，其最大长度为MaxSize；
{
  Stack *s=(Stack*)malloc(sizeof(Stack)*maxSize);
  s->Top=-1;
  return s;
}
int IsFull( Stack *S, int maxSize)//：判断堆栈S是否已满；
{
  if (S->Top==maxSize-1)
  {
    return 1;
  }else
  {
    return 0;
  }
}
void Push( Stack *S, int item )//：将元素item压入堆栈；
{
  if (S->Top==MaxSize-1)
  {
    printf("the stack is full");
    return;
  }
  else
  {
    S->Data[++(S->Top)]=item;
    return;
  }

}
int IsEmpty ( Stack *S )//：判断堆栈S是否为空；
{
  if (S->Top=-1)
  {
    return 1;
  }else
  {
    return 0;
  }
}
int Pop( Stack *S )//：删除并返回栈顶元素；
{
  if (S->Top==-1)
  {
    printf("the stack is empty");
    return ERROR;
  }
  else
  {
    return S->Data[(S->Top)--];
  }
}

void show(Stack *s)
{
  int i;
  for(i=0;i<=s->Top;i++)
  {
    printf("%d ",s->Data[i]);
  }
}

void main()
{
  int a,b;
  Stack *s=CreateStack(MaxSize);
  while (1)
  {
    printf("please enter your order:");
    scanf("%d",&a);
    switch (a)
    {
    case 1:
      scanf("%d",&b);
      Push(s,b);
      break;
    case 2:
    Pop(s);
    break;
    case 3:
      show(s);
      break;
    case 4:
      exit(0);
      break;
    default:
      break;
    }
  }

}
```
![堆栈](http://img.blog.csdn.net/20160305203628385)


**链式堆栈**     


```
typedef struct Node{
  int Data;
  struct Node *Next;
} LinkStack;

LinkStack *CreateStack()
{
  LinkStack *s=(LinkStack*)malloc(sizeof(LinkStack));
  s->Next=NULL;
  return s;
}

int IsEmpty(LinkStack *s)
{
  return (s->Next==NULL);
}

void Push( int item, LinkStack *S )
{
  LinkStack *TmpCell;
  TmpCell=(LinkStack*)malloc(sizeof(LinkStack));
  TmpCell->Data=item;
  TmpCell->Next=S->Next;
  S->Next=TmpCell;

}

int pop(LinkStack *s)
{
  LinkStack *firstItem;
  int topNum;
  if (IsEmpty(s))
  {
    printf("stack is empty");
    return NULL;
  }

  firstItem=s->Next;
  s->Next=firstItem->Next;
  topNum=firstItem->Data;
  free(firstItem);
  return topNum;

}
void show(LinkStack *s)
{
  s=s->Next;
  while (s)
  {
    
    printf("%d ",s->Data);
    s=s->Next;
  }
}

void main()
{
  int a,b;
  LinkStack *s=CreateStack();
  while (1)
  {
    printf("please enter your order:");
    scanf("%d",&a);
    switch (a)
    {
    case 1:
      scanf("%d",&b);
      Push(b,s);
      break;
    case 2:
      pop(s);
      break;
    case 3:
      show(s);
      break;
    case 4:
      exit(0);
      break;
    default:
      break;
    }
  }
}
```
![堆栈](http://img.blog.csdn.net/20160305203656604)

**顺序存储队列**   


```
#define MaxSize 20
#define ERROR -1
typedef struct {
  int Data[ MaxSize ];
  int rear;
  int front;
} Queue;

void AddQ( Queue *PtrQ, int item)
{
  if ((PtrQ->rear+1)%MaxSize==PtrQ->front)
  {
    printf("the queue is full");

  }

  else
  {
    PtrQ->rear=(PtrQ->rear+1)%MaxSize;
    PtrQ->Data[PtrQ->rear]=item;
  }
}

int DeleteQ ( Queue *PtrQ )
{
  if (PtrQ->front==PtrQ->rear)
  {
    printf("the queue is empty");

  }else
  {
    PtrQ->front=(PtrQ->front+1)%MaxSize;
    return PtrQ->Data[PtrQ->front-1];
  }
}
void show(Queue *PtrQ)
{
  int i;
  for(i=PtrQ->front+1;i<=PtrQ->rear;i++)
  {
    printf("%d ",PtrQ->Data[i%MaxSize]);
  }
}

void main()
{
  int a,b;
  Queue *q=(Queue*)malloc(sizeof(Queue));
  q->front=-1;
  q->rear=-1;

  while (1)
  {
    printf("please enter your order:");
    scanf("%d",&a);
    switch (a)
    {
    case 1:
      scanf("%d",&b);
      AddQ(q,b);
      break;
    case 2:
      DeleteQ(q);
      break;
    case 3:
      show(q);
      break;
    case 4:
      exit(0);
      break;
    default:
      break;
    }
  }
}
```
![队列](http://img.blog.csdn.net/20160305203750731)


**链式队列** 


```
#define ERROR -1
typedef struct Node{
  int  Data;
  struct Node *Next;
}QNode;
typedef struct { /* 链队列结构 */
  QNode *rear; /* 指向队尾结点 */
  QNode *front; /* 指向队头结点 */
} LinkQueue;

LinkQueue* AddQuee(LinkQueue *PtrL,int item)
{
  QNode *node;
  node=PtrL->rear;
  if (PtrL->front==NULL)
  {
    QNode *q=(QNode*)malloc(sizeof(QNode));
    q->Next=NULL;
    q->Data=item;
    PtrL->front=q;
    PtrL->rear=q;
    return PtrL;
  }
  else
  {
    QNode *q=(QNode*)malloc(sizeof(QNode));
    q->Next=NULL;
    q->Data=item;
      node->Next=q;
    PtrL->rear=q;
    return PtrL;
  }
}

int DeleteQ ( LinkQueue *PtrQ )
{
  QNode *firstNode;
  int NodeItem;

  if (PtrQ->front==NULL)
  {
    printf("queue is empty");
    return ERROR;
  }

  firstNode=PtrQ->front;
  if (PtrQ->front==PtrQ->rear)
  {
    PtrQ->front=PtrQ->rear=NULL;
  }else
  {
    PtrQ->front=PtrQ->front->Next;
  }

  NodeItem=firstNode->Data;
  free(firstNode);
  return NodeItem;
}

void show(LinkQueue *q)
{
  QNode *node;
  node=q->front;
  while (node)
  {
    printf("%d ",node->Data);
    node=node->Next;
  }
}

void main()
{
  int a,b;
  LinkQueue *q=(LinkQueue*)malloc(sizeof(LinkQueue));
  q->front=NULL;
  q->rear=NULL;
  

  while (1)
  {
    printf("please enter your order:");
    scanf("%d",&a);
    switch (a)
    {
    case 1:
      scanf("%d",&b);
      q=AddQuee(q,b);
      break;
    case 2:
      DeleteQ(q);
      break;
    case 3:
      show(q);
      break;
    case 4:
      exit(0);
      break;
    default:
      break;
    }
  }
}
```         

![队列](http://img.blog.csdn.net/20160305203843592)