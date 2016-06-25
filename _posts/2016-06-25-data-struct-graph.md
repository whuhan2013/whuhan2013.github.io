---
layout: post
title: 数据结构之图的实现
date: 2016-6-25
categories: blog
tags: [数据结构]
description: 数据结构之图的实现
---


**本文主要包括以下内容** 

1. 邻接矩阵实现无向图  
2. 邻接表实现无向图  
3. 邻接矩阵实现有向图
4. 邻接表实现有向图  


**图的理论基础，请参考：**[图的理论基础 - 如果天空不死 - 博客园](http://www.cnblogs.com/skywang12345/p/3691463.html)


### 邻接矩阵实现无向图

MatrixUDG是邻接矩阵对应的结构体。               
mVexs用于保存顶点，mVexNum是顶点数，mEdgNum是边数；mMatrix则是用于保存矩阵信息的二维数组。例如，mMatrix[i][j]=1，则表示"顶点i(即mVexs[i])"和"顶点j(即mVexs[j])"是邻接点；mMatrix[i][j]=0，则表示它们不是邻接点。

```
/**
 * C++: 邻接矩阵表示的"无向图(List Undirected Graph)"
 *
 * @author skywang
 * @date 2014/04/19
 */

#include <iomanip>
#include <iostream>
#include <vector>
using namespace std;

#define MAX 100
class MatrixUDG {
    private:
        char mVexs[MAX];    // 顶点集合
        int mVexNum;             // 顶点数
        int mEdgNum;             // 边数
        int mMatrix[MAX][MAX];   // 邻接矩阵

    public:
        // 创建图(自己输入数据)
        MatrixUDG();
        // 创建图(用已提供的矩阵)
        MatrixUDG(char vexs[], int vlen, char edges[][2], int elen);
        ~MatrixUDG();

        // 打印矩阵队列图
        void print();

    private:
        // 读取一个输入字符
        char readChar();
        // 返回ch在mMatrix矩阵中的位置
        int getPosition(char ch);
};

/* 
 * 创建图(自己输入数据)
 */
MatrixUDG::MatrixUDG()
{
    char c1, c2;
    int i, p1, p2;
    
    // 输入"顶点数"和"边数"
    cout << "input vertex number: ";
    cin >> mVexNum;
    cout << "input edge number: ";
    cin >> mEdgNum;
    if ( mVexNum < 1 || mEdgNum < 1 || (mEdgNum > (mVexNum * (mVexNum-1))))
    {
        cout << "input error: invalid parameters!" << endl;
        return ;
    }
    
    // 初始化"顶点"
    for (i = 0; i < mVexNum; i++)
    {
        cout << "vertex(" << i << "): ";
        mVexs[i] = readChar();
    }

    // 初始化"边"
    for (i = 0; i < mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        cout << "edge(" << i << "): ";
        c1 = readChar();
        c2 = readChar();

        p1 = getPosition(c1);
        p2 = getPosition(c2);
        if (p1==-1 || p2==-1)
        {
            cout << "input error: invalid edge!" << endl;
            return ;
        }

        mMatrix[p1][p2] = 1;
        mMatrix[p2][p1] = 1;
    }
}

/*
 * 创建图(用已提供的矩阵)
 *
 * 参数说明：
 *     vexs  -- 顶点数组
 *     vlen  -- 顶点数组的长度
 *     edges -- 边数组
 *     elen  -- 边数组的长度
 */
MatrixUDG::MatrixUDG(char vexs[], int vlen, char edges[][2], int elen)
{
    int i, p1, p2;
    
    // 初始化"顶点数"和"边数"
    mVexNum = vlen;
    mEdgNum = elen;
    // 初始化"顶点"
    for (i = 0; i < mVexNum; i++)
        mVexs[i] = vexs[i];

    // 初始化"边"
    for (i = 0; i < mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        p1 = getPosition(edges[i][0]);
        p2 = getPosition(edges[i][1]);

        mMatrix[p1][p2] = 1;
        mMatrix[p2][p1] = 1;
    }
}

/* 
 * 析构函数
 */
MatrixUDG::~MatrixUDG() 
{
}

/*
 * 返回ch在mMatrix矩阵中的位置
 */
int MatrixUDG::getPosition(char ch)
{
    int i;
    for(i=0; i<mVexNum; i++)
        if(mVexs[i]==ch)
            return i;
    return -1;
}

/*
 * 读取一个输入字符
 */
char MatrixUDG::readChar()
{
    char ch;

    do {
        cin >> ch;
    } while(!((ch>='a'&&ch<='z') || (ch>='A'&&ch<='Z')));

    return ch;
}

/*
 * 打印矩阵队列图
 */
void MatrixUDG::print()
{
    int i,j;

    cout << "Martix Graph:" << endl;
    for (i = 0; i < mVexNum; i++)
    {
        for (j = 0; j < mVexNum; j++)
            cout << mMatrix[i][j] << " ";
        cout << endl;
    }
}

int main()
{
    char vexs[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
    char edges[][2] = {
        {'A', 'C'}, 
        {'A', 'D'}, 
        {'A', 'F'}, 
        {'B', 'C'}, 
        {'C', 'D'}, 
        {'E', 'G'}, 
        {'F', 'G'}};
    int vlen = sizeof(vexs)/sizeof(vexs[0]);
    int elen = sizeof(edges)/sizeof(edges[0]);
    MatrixUDG* pG;

    // 自定义"图"(输入矩阵队列)
    //pG = new MatrixUDG();
    // 采用已有的"图"
    pG = new MatrixUDG(vexs, vlen, edges, elen);

    pG->print();   // 打印图

    return 0;
}
```



### 邻接表实现无向图


(01) ListUDG是邻接表对应的结构体。              
mVexNum是顶点数，mEdgNum是边数；mVexs则是保存顶点信息的一维数组。
 
(02) VNode是邻接表顶点对应的结构体。               
data是顶点所包含的数据，而firstEdge是该顶点所包含链表的表头指针。
 
(03) ENode是邻接表顶点所包含的链表的节点对应的结构体。               
ivex是该节点所对应的顶点在vexs中的索引，而nextEdge是指向下一个节点的


```
/**
 * C++: 邻接表表示的"无向图(List Undirected Graph)"
 *
 * @author skywang
 * @date 2014/04/19
 */

#include <iomanip>
#include <iostream>
#include <vector>
using namespace std;

#define MAX 100
// 邻接表
class ListUDG
{
    private: // 内部类
        // 邻接表中表对应的链表的顶点
        class ENode
        {
            public:
                int ivex;           // 该边所指向的顶点的位置
                ENode *nextEdge;    // 指向下一条弧的指针
        };

        // 邻接表中表的顶点
        class VNode
        {
            public:
                char data;          // 顶点信息
                ENode *firstEdge;   // 指向第一条依附该顶点的弧
        };

    private: // 私有成员
        int mVexNum;             // 图的顶点的数目
        int mEdgNum;             // 图的边的数目
        VNode mVexs[MAX];

    public:
        // 创建邻接表对应的图(自己输入)
        ListUDG();
        // 创建邻接表对应的图(用已提供的数据)
        ListUDG(char vexs[], int vlen, char edges[][2], int elen);
        ~ListUDG();

        // 打印邻接表图
        void print();

    private:
        // 读取一个输入字符
        char readChar();
        // 返回ch的位置
        int getPosition(char ch);
        // 将node节点链接到list的最后
        void linkLast(ENode *list, ENode *node);
};

/*
 * 创建邻接表对应的图(自己输入)
 */
ListUDG::ListUDG()
{
    char c1, c2;
    int v, e;
    int i, p1, p2;
    ENode *node1, *node2;

    // 输入"顶点数"和"边数"
    cout << "input vertex number: ";
    cin >> mVexNum;
    cout << "input edge number: ";
    cin >> mEdgNum;
    if ( mVexNum < 1 || mEdgNum < 1 || (mEdgNum > (mVexNum * (mVexNum-1))))
    {
        cout << "input error: invalid parameters!" << endl;
        return ;
    }
 
    // 初始化"邻接表"的顶点
    for(i=0; i<mVexNum; i++)
    {
        cout << "vertex(" << i << "): ";
        mVexs[i].data = readChar();
        mVexs[i].firstEdge = NULL;
    }

    // 初始化"邻接表"的边
    for(i=0; i<mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        cout << "edge(" << i << "): ";
        c1 = readChar();
        c2 = readChar();

        p1 = getPosition(c1);
        p2 = getPosition(c2);
        // 初始化node1
        node1 = new ENode();
        node1->ivex = p2;
        // 将node1链接到"p1所在链表的末尾"
        if(mVexs[p1].firstEdge == NULL)
          mVexs[p1].firstEdge = node1;
        else
            linkLast(mVexs[p1].firstEdge, node1);
        // 初始化node2
        node2 = new ENode();
        node2->ivex = p1;
        // 将node2链接到"p2所在链表的末尾"
        if(mVexs[p2].firstEdge == NULL)
          mVexs[p2].firstEdge = node2;
        else
            linkLast(mVexs[p2].firstEdge, node2);
    }
}

/*
 * 创建邻接表对应的图(用已提供的数据)
 */
ListUDG::ListUDG(char vexs[], int vlen, char edges[][2], int elen)
{
    char c1, c2;
    int i, p1, p2;
    ENode *node1, *node2;

    // 初始化"顶点数"和"边数"
    mVexNum = vlen;
    mEdgNum = elen;
    // 初始化"邻接表"的顶点
    for(i=0; i<mVexNum; i++)
    {
        mVexs[i].data = vexs[i];
        mVexs[i].firstEdge = NULL;
    }

    // 初始化"邻接表"的边
    for(i=0; i<mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        c1 = edges[i][0];
        c2 = edges[i][1];

        p1 = getPosition(c1);
        p2 = getPosition(c2);
        // 初始化node1
        node1 = new ENode();
        node1->ivex = p2;
        // 将node1链接到"p1所在链表的末尾"
        if(mVexs[p1].firstEdge == NULL)
          mVexs[p1].firstEdge = node1;
        else
            linkLast(mVexs[p1].firstEdge, node1);
        // 初始化node2
        node2 = new ENode();
        node2->ivex = p1;
        // 将node2链接到"p2所在链表的末尾"
        if(mVexs[p2].firstEdge == NULL)
          mVexs[p2].firstEdge = node2;
        else
            linkLast(mVexs[p2].firstEdge, node2);
    }
}

/* 
 * 析构函数
 */
ListUDG::~ListUDG() 
{
}

/*
 * 将node节点链接到list的最后
 */
void ListUDG::linkLast(ENode *list, ENode *node)
{
    ENode *p = list;

    while(p->nextEdge)
        p = p->nextEdge;
    p->nextEdge = node;
}

/*
 * 返回ch的位置
 */
int ListUDG::getPosition(char ch)
{
    int i;
    for(i=0; i<mVexNum; i++)
        if(mVexs[i].data==ch)
            return i;
    return -1;
}

/*
 * 读取一个输入字符
 */
char ListUDG::readChar()
{
    char ch;

    do {
        cin >> ch;
    } while(!((ch>='a'&&ch<='z') || (ch>='A'&&ch<='Z')));

    return ch;
}

/*
 * 打印邻接表图
 */
void ListUDG::print()
{
    int i,j;
    ENode *node;

    cout << "List Graph:" << endl;
    for (i = 0; i < mVexNum; i++)
    {
        cout << i << "(" << mVexs[i].data << "): ";
        node = mVexs[i].firstEdge;
        while (node != NULL)
        {
            cout << node->ivex << "(" << mVexs[node->ivex].data << ") ";
            node = node->nextEdge;
        }
        cout << endl;
    }
}

int main()
{
    char vexs[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
    char edges[][2] = {
        {'A', 'C'}, 
        {'A', 'D'}, 
        {'A', 'F'}, 
        {'B', 'C'}, 
        {'C', 'D'}, 
        {'E', 'G'}, 
        {'F', 'G'}};
    int vlen = sizeof(vexs)/sizeof(vexs[0]);
    int elen = sizeof(edges)/sizeof(edges[0]);
    ListUDG* pG;

    // 自定义"图"(输入矩阵队列)
    //pG = new ListUDG();
    // 采用已有的"图"
    pG = new ListUDG(vexs, vlen, edges, elen);

    pG->print();   // 打印图

    return 0;
}
```

### 邻接矩阵实现有向图

MatrixDG是邻接矩阵有向图对应的结构体。 

mVexs用于保存顶点，mVexNum是顶点数，mEdgNum是边数；mMatrix则是用于保存矩阵信息的二维数组。例如，mMatrix[i][j]=1，则表示"顶点i(即mVexs[i])"和"顶点j(即mVexs[j])"是邻接点，且顶点i是起点，顶点j是终点。

```
/**
 * C++: 邻接矩阵图
 *
 * @author skywang
 * @date 2014/04/19
 */

#include <iomanip>
#include <iostream>
#include <vector>
using namespace std;

#define MAX 100
class MatrixDG {
    private:
        char mVexs[MAX];    // 顶点集合
        int mVexNum;             // 顶点数
        int mEdgNum;             // 边数
        int mMatrix[MAX][MAX];   // 邻接矩阵

    public:
        // 创建图(自己输入数据)
        MatrixDG();
        // 创建图(用已提供的矩阵)
        MatrixDG(char vexs[], int vlen, char edges[][2], int elen);
        ~MatrixDG();

        // 打印矩阵队列图
        void print();

    private:
        // 读取一个输入字符
        char readChar();
        // 返回ch在mMatrix矩阵中的位置
        int getPosition(char ch);
};

/* 
 * 创建图(自己输入数据)
 */
MatrixDG::MatrixDG()
{
    char c1, c2;
    int i, p1, p2;
    
    // 输入"顶点数"和"边数"
    cout << "input vertex number: ";
    cin >> mVexNum;
    cout << "input edge number: ";
    cin >> mEdgNum;
    if ( mVexNum < 1 || mEdgNum < 1 || (mEdgNum > (mVexNum * (mVexNum-1))))
    {
        cout << "input error: invalid parameters!" << endl;
        return ;
    }
    
    // 初始化"顶点"
    for (i = 0; i < mVexNum; i++)
    {
        cout << "vertex(" << i << "): ";
        mVexs[i] = readChar();
    }

    // 初始化"边"
    for (i = 0; i < mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        cout << "edge(" << i << "): ";
        c1 = readChar();
        c2 = readChar();

        p1 = getPosition(c1);
        p2 = getPosition(c2);
        if (p1==-1 || p2==-1)
        {
            cout << "input error: invalid edge!" << endl;
            return ;
        }

        mMatrix[p1][p2] = 1;
    }
}

/*
 * 创建图(用已提供的矩阵)
 *
 * 参数说明：
 *     vexs  -- 顶点数组
 *     vlen  -- 顶点数组的长度
 *     edges -- 边数组
 *     elen  -- 边数组的长度
 */
MatrixDG::MatrixDG(char vexs[], int vlen, char edges[][2], int elen)
{
    int i, p1, p2;
    
    // 初始化"顶点数"和"边数"
    mVexNum = vlen;
    mEdgNum = elen;
    // 初始化"顶点"
    for (i = 0; i < mVexNum; i++)
        mVexs[i] = vexs[i];

    // 初始化"边"
    for (i = 0; i < mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        p1 = getPosition(edges[i][0]);
        p2 = getPosition(edges[i][1]);

        mMatrix[p1][p2] = 1;
    }
}

/* 
 * 析构函数
 */
MatrixDG::~MatrixDG() 
{
}

/*
 * 返回ch在mMatrix矩阵中的位置
 */
int MatrixDG::getPosition(char ch)
{
    int i;
    for(i=0; i<mVexNum; i++)
        if(mVexs[i]==ch)
            return i;
    return -1;
}

/*
 * 读取一个输入字符
 */
char MatrixDG::readChar()
{
    char ch;

    do {
        cin >> ch;
    } while(!((ch>='a'&&ch<='z') || (ch>='A'&&ch<='Z')));

    return ch;
}

/*
 * 打印矩阵队列图
 */
void MatrixDG::print()
{
    int i,j;

    cout << "Martix Graph:" << endl;
    for (i = 0; i < mVexNum; i++)
    {
        for (j = 0; j < mVexNum; j++)
            cout << mMatrix[i][j] << " ";
        cout << endl;
    }
}

int main()
{
    char vexs[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
    char edges[][2] = {
        {'A', 'B'}, 
        {'B', 'C'}, 
        {'B', 'E'}, 
        {'B', 'F'}, 
        {'C', 'E'}, 
        {'D', 'C'}, 
        {'E', 'B'}, 
        {'E', 'D'}, 
        {'F', 'G'}}; 
    int vlen = sizeof(vexs)/sizeof(vexs[0]);
    int elen = sizeof(edges)/sizeof(edges[0]);
    MatrixDG* pG;

    // 自定义"图"(输入矩阵队列)
    //pG = new MatrixDG();
    // 采用已有的"图"
    pG = new MatrixDG(vexs, vlen, edges, elen);

    pG->print();   // 打印图

    return 0;
}
```

### 邻接表实现有向图


(01) ListDG是邻接表对应的结构体。 mVexNum是顶点数，mEdgNum是边数；mVexs则是保存顶点信息的一维数组。      
(02) VNode是邻接表顶点对应的结构体。 data是顶点所包含的数据，而firstEdge是该顶点所包含链表的表头指针。      
(03) ENode是邻接表顶点所包含的链表的节点对应的结构体。 ivex是该节点所对应的顶点在vexs中的索引，而nextEdge是指向下一个节点的。     


```
/**
 * C++: 邻接表图
 *
 * @author skywang
 * @date 2014/04/19
 */

#include <iomanip>
#include <iostream>
#include <vector>
using namespace std;

#define MAX 100
// 邻接表
class ListDG
{
    private: // 内部类
        // 邻接表中表对应的链表的顶点
        class ENode
        {
            public:
                int ivex;           // 该边所指向的顶点的位置
                ENode *nextEdge;    // 指向下一条弧的指针
        };

        // 邻接表中表的顶点
        class VNode
        {
            public:
                char data;          // 顶点信息
                ENode *firstEdge;   // 指向第一条依附该顶点的弧
        };

    private: // 私有成员
        int mVexNum;             // 图的顶点的数目
        int mEdgNum;             // 图的边的数目
        VNode mVexs[MAX];

    public:
        // 创建邻接表对应的图(自己输入)
        ListDG();
        // 创建邻接表对应的图(用已提供的数据)
        ListDG(char vexs[], int vlen, char edges[][2], int elen);
        ~ListDG();

        // 打印邻接表图
        void print();

    private:
        // 读取一个输入字符
        char readChar();
        // 返回ch的位置
        int getPosition(char ch);
        // 将node节点链接到list的最后
        void linkLast(ENode *list, ENode *node);
};

/*
 * 创建邻接表对应的图(自己输入)
 */
ListDG::ListDG()
{
    char c1, c2;
    int v, e;
    int i, p1, p2;
    ENode *node1, *node2;

    // 输入"顶点数"和"边数"
    cout << "input vertex number: ";
    cin >> mVexNum;
    cout << "input edge number: ";
    cin >> mEdgNum;
    if ( mVexNum < 1 || mEdgNum < 1 || (mEdgNum > (mVexNum * (mVexNum-1))))
    {
        cout << "input error: invalid parameters!" << endl;
        return ;
    }
 
    // 初始化"邻接表"的顶点
    for(i=0; i<mVexNum; i++)
    {
        cout << "vertex(" << i << "): ";
        mVexs[i].data = readChar();
        mVexs[i].firstEdge = NULL;
    }

    // 初始化"邻接表"的边
    for(i=0; i<mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        cout << "edge(" << i << "): ";
        c1 = readChar();
        c2 = readChar();

        p1 = getPosition(c1);
        p2 = getPosition(c2);
        // 初始化node1
        node1 = new ENode();
        node1->ivex = p2;
        // 将node1链接到"p1所在链表的末尾"
        if(mVexs[p1].firstEdge == NULL)
          mVexs[p1].firstEdge = node1;
        else
            linkLast(mVexs[p1].firstEdge, node1);
    }
}

/*
 * 创建邻接表对应的图(用已提供的数据)
 */
ListDG::ListDG(char vexs[], int vlen, char edges[][2], int elen)
{
    char c1, c2;
    int i, p1, p2;
    ENode *node1, *node2;

    // 初始化"顶点数"和"边数"
    mVexNum = vlen;
    mEdgNum = elen;
    // 初始化"邻接表"的顶点
    for(i=0; i<mVexNum; i++)
    {
        mVexs[i].data = vexs[i];
        mVexs[i].firstEdge = NULL;
    }

    // 初始化"邻接表"的边
    for(i=0; i<mEdgNum; i++)
    {
        // 读取边的起始顶点和结束顶点
        c1 = edges[i][0];
        c2 = edges[i][1];

        p1 = getPosition(c1);
        p2 = getPosition(c2);
        // 初始化node1
        node1 = new ENode();
        node1->ivex = p2;
        // 将node1链接到"p1所在链表的末尾"
        if(mVexs[p1].firstEdge == NULL)
          mVexs[p1].firstEdge = node1;
        else
            linkLast(mVexs[p1].firstEdge, node1);
    }
}

/* 
 * 析构函数
 */
ListDG::~ListDG() 
{
}

/*
 * 将node节点链接到list的最后
 */
void ListDG::linkLast(ENode *list, ENode *node)
{
    ENode *p = list;

    while(p->nextEdge)
        p = p->nextEdge;
    p->nextEdge = node;
}


/*
 * 返回ch的位置
 */
int ListDG::getPosition(char ch)
{
    int i;
    for(i=0; i<mVexNum; i++)
        if(mVexs[i].data==ch)
            return i;
    return -1;
}

/*
 * 读取一个输入字符
 */
char ListDG::readChar()
{
    char ch;

    do {
        cin >> ch;
    } while(!((ch>='a'&&ch<='z') || (ch>='A'&&ch<='Z')));

    return ch;
}

/*
 * 打印邻接表图
 */
void ListDG::print()
{
    int i,j;
    ENode *node;

    cout << "List Graph:" << endl;
    for (i = 0; i < mVexNum; i++)
    {
        cout << i << "(" << mVexs[i].data << "): ";
        node = mVexs[i].firstEdge;
        while (node != NULL)
        {
            cout << node->ivex << "(" << mVexs[node->ivex].data << ") ";
            node = node->nextEdge;
        }
        cout << endl;
    }
}

int main()
{
    char vexs[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
    char edges[][2] = {
        {'A', 'B'}, 
        {'B', 'C'}, 
        {'B', 'E'}, 
        {'B', 'F'}, 
        {'C', 'E'}, 
        {'D', 'C'}, 
        {'E', 'B'}, 
        {'E', 'D'}, 
        {'F', 'G'}}; 
    int vlen = sizeof(vexs)/sizeof(vexs[0]);
    int elen = sizeof(edges)/sizeof(edges[0]);
    ListDG* pG;

    // 自定义"图"(输入矩阵队列)
    //pG = new ListDG();
    // 采用已有的"图"
    pG = new ListDG(vexs, vlen, edges, elen);

    pG->print();   // 打印图

    return 0;
}
```