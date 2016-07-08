---
layout: post
title: 数据结构之最小生成树
date: 2016-7-8
categories: blog
tags: [数据结构]
description: 数据结构之最小生成树
---


### prime算法    

普里姆(Prim)算法，是用来求加权连通图的最小生成树的算法。 

**基本思想**                
对于图G而言，V是所有顶点的集合；现在，设置两个新的集合U和T，其中U用于存放G的最小生成树中的顶点，T存放G的最小生成树中的边。 从所有uЄU，vЄ(V-U) (V-U表示出去U的所有顶点)的边中选取权值最小的边(u, v)，将顶点v加入集合U中，将边(u, v)加入集合T中，如此不断重复，直到U=V为止，最小生成树构造完毕，这时集合T中包含了最小生成树中的所有边。

**邻接矩阵实现** 

基本定义 

```
class MatrixUDG {
    #define MAX    100
    #define INF    (~(0x1<<31))        // 无穷大(即0X7FFFFFFF)
    private:
        char mVexs[MAX];    // 顶点集合
        int mVexNum;             // 顶点数
        int mEdgNum;             // 边数
        int mMatrix[MAX][MAX];   // 邻接矩阵

    public:
        // 创建图(自己输入数据)
        MatrixUDG();
        // 创建图(用已提供的矩阵)
        //MatrixUDG(char vexs[], int vlen, char edges[][2], int elen);
        MatrixUDG(char vexs[], int vlen, int matrix[][9]);
        ~MatrixUDG();

        // 深度优先搜索遍历图
        void DFS();
        // 广度优先搜索（类似于树的层次遍历）
        void BFS();
        // prim最小生成树(从start开始生成最小生成树)
        void prim(int start);
        // 打印矩阵队列图
        void print();

    private:
        // 读取一个输入字符
        char readChar();
        // 返回ch在mMatrix矩阵中的位置
        int getPosition(char ch);
        // 返回顶点v的第一个邻接顶点的索引，失败则返回-1
        int firstVertex(int v);
        // 返回顶点v相对于w的下一个邻接顶点的索引，失败则返回-1
        int nextVertex(int v, int w);
        // 深度优先搜索遍历图的递归实现
        void DFS(int i, int *visited);

};
```

MatrixUDG是邻接矩阵对应的结构体。 
mVexs用于保存顶点，mVexNum是顶点数，mEdgNum是边数；mMatrix则是用于保存矩阵信息的二维数组。例如，mMatrix[i][j]=1，则表示"顶点i(即mVexs[i])"和"顶点j(即mVexs[j])"是邻接点；mMatrix[i][j]=0，则表示它们不是邻接点。


**prime算法** 

```
/*
 * prim最小生成树
 *
 * 参数说明：
 *   start -- 从图中的第start个元素开始，生成最小树
 */
void MatrixUDG::prim(int start)
{
    int min,i,j,k,m,n,sum;
    int index=0;         // prim最小树的索引，即prims数组的索引
    char prims[MAX];     // prim最小树的结果数组
    int weights[MAX];    // 顶点间边的权值

    // prim最小生成树中第一个数是"图中第start个顶点"，因为是从start开始的。
    prims[index++] = mVexs[start];

    // 初始化"顶点的权值数组"，
    // 将每个顶点的权值初始化为"第start个顶点"到"该顶点"的权值。
    for (i = 0; i < mVexNum; i++ )
        weights[i] = mMatrix[start][i];
    // 将第start个顶点的权值初始化为0。
    // 可以理解为"第start个顶点到它自身的距离为0"。
    weights[start] = 0;

    for (i = 0; i < mVexNum; i++)
    {
        // 由于从start开始的，因此不需要再对第start个顶点进行处理。
        if(start == i)
            continue;

        j = 0;
        k = 0;
        min = INF;
        // 在未被加入到最小生成树的顶点中，找出权值最小的顶点。
        while (j < mVexNum)
        {
            // 若weights[j]=0，意味着"第j个节点已经被排序过"(或者说已经加入了最小生成树中)。
            if (weights[j] != 0 && weights[j] < min)
            {
                min = weights[j];
                k = j;
            }
            j++;
        }

        // 经过上面的处理后，在未被加入到最小生成树的顶点中，权值最小的顶点是第k个顶点。
        // 将第k个顶点加入到最小生成树的结果数组中
        prims[index++] = mVexs[k];
        // 将"第k个顶点的权值"标记为0，意味着第k个顶点已经排序过了(或者说已经加入了最小树结果中)。
        weights[k] = 0;
        // 当第k个顶点被加入到最小生成树的结果数组中之后，更新其它顶点的权值。
        for (j = 0 ; j < mVexNum; j++)
        {
            // 当第j个节点没有被处理，并且需要更新时才被更新。
            if (weights[j] != 0 && mMatrix[k][j] < weights[j])
                weights[j] = mMatrix[k][j];
        }
    }

    // 计算最小生成树的权值
    sum = 0;
    for (i = 1; i < index; i++)
    {
        min = INF;
        // 获取prims[i]在mMatrix中的位置
        n = getPosition(prims[i]);
        // 在vexs[0...i]中，找出到j的权值最小的顶点。
        for (j = 0; j < i; j++)
        {
            m = getPosition(prims[j]);
            if (mMatrix[m][n]<min)
                min = mMatrix[m][n];
        }
        sum += min;
    }
    // 打印最小生成树
    cout << "PRIM(" << mVexs[start] << ")=" << sum << ": ";
    for (i = 0; i < index; i++)
        cout << prims[i] << " ";
    cout << endl;
}
```


### kruskal算法          

克鲁斯卡尔(Kruskal)算法，是用来求加权连通图的最小生成树的算法。 

基本思想：按照权值从小到大的顺序选择n-1条边，并保证这n-1条边不构成回路。 
具体做法：首先构造一个只含n个顶点的森林，然后依权值从小到大从连通网中选择边加入到森林中，并使森林中不产生回路，直至森林变成一棵树为止。


**克鲁斯卡尔算法分析**
 
根据前面介绍的克鲁斯卡尔算法的基本思想和做法，我们能够了解到，克鲁斯卡尔算法重点需要解决的以下两个问题：   
问题一 对图的所有边按照权值大小进行排序。                 
问题二 将边添加到最小生成树中时，怎么样判断是否形成了回路。                     

问题一很好解决，采用排序算法进行排序即可。
 
问题二，处理方式是：记录顶点在"最小生成树"中的终点，顶点的终点是"在最小生成树中与它连通的最大顶点"(关于这一点，后面会通过图片给出说明)                                                      。然后每次需要将一条边添加到最小生存树时，判断该边的两个顶点的终点是否重合，重合的话则会构成回路。 以下图来进行说明：            
![](https://github.com/wangkuiwu/datastructs_and_algorithm/blob/master/pictures/graph/kruskal/04.jpg?raw=true)

在将<E,F> <C,D> <D,E>加入到最小生成树R中之后，这几条边的顶点就都有了终点： 

(01) C的终点是F。        
(02) D的终点是F。           
(03) E的终点是F。           
(04) F的终点是F。              

关于终点，就是将所有顶点按照从小到大的顺序排列好之后；某个顶点的终点就是"与它连通的最大顶点"。 因此，接下来，虽然<C,E>是权值最小的边。但是C和E的重点都是F，即它们的终点相同，因此，将<C,E>加入最小生成树的话，会形成回路。这就是判断回路的方式。          


**邻接矩阵实现**  

基本定义

```
// 边的结构体
class EData
{
    public:
        char start; // 边的起点
        char end;   // 边的终点
        int weight; // 边的权重

    public:
        EData(){}
        EData(char s, char e, int w):start(s),end(e),weight(w){}
};
```

EData是邻接矩阵边对应的结构体。

```
class MatrixUDG {
    #define MAX    100
    #define INF    (~(0x1<<31))        // 无穷大(即0X7FFFFFFF)
    private:
        char mVexs[MAX];    // 顶点集合
        int mVexNum;             // 顶点数
        int mEdgNum;             // 边数
        int mMatrix[MAX][MAX];   // 邻接矩阵

    public:
        // 创建图(自己输入数据)
        MatrixUDG();
        // 创建图(用已提供的矩阵)
        //MatrixUDG(char vexs[], int vlen, char edges[][2], int elen);
        MatrixUDG(char vexs[], int vlen, int matrix[][9]);
        ~MatrixUDG();

        // 深度优先搜索遍历图
        void DFS();
        // 广度优先搜索（类似于树的层次遍历）
        void BFS();
        // prim最小生成树(从start开始生成最小生成树)
        void prim(int start);
        // 克鲁斯卡尔（Kruskal)最小生成树
        void kruskal();
        // 打印矩阵队列图
        void print();

    private:
        // 读取一个输入字符
        char readChar();
        // 返回ch在mMatrix矩阵中的位置
        int getPosition(char ch);
        // 返回顶点v的第一个邻接顶点的索引，失败则返回-1
        int firstVertex(int v);
        // 返回顶点v相对于w的下一个邻接顶点的索引，失败则返回-1
        int nextVertex(int v, int w);
        // 深度优先搜索遍历图的递归实现
        void DFS(int i, int *visited);
        // 获取图中的边
        EData* getEdges();
        // 对边按照权值大小进行排序(由小到大)
        void sortEdges(EData* edges, int elen);
        // 获取i的终点
        int getEnd(int vends[], int i);
};
```

MatrixUDG是邻接矩阵对应的结构体。 
mVexs用于保存顶点，mVexNum是顶点数，mEdgNum是边数；mMatrix则是用于保存矩阵信息的二维数组。例如，mMatrix[i][j]=1，则表示"顶点i(即mVexs[i])"和"顶点j(即mVexs[j])"是邻接点；mMatrix[i][j]=0，则表示它们不是邻接点。

```
/*
 * 克鲁斯卡尔（Kruskal)最小生成树
 */
void MatrixUDG::kruskal()
{
    int i,m,n,p1,p2;
    int length;
    int index = 0;          // rets数组的索引
    int vends[MAX]={0};     // 用于保存"已有最小生成树"中每个顶点在该最小树中的终点。
    EData rets[MAX];        // 结果数组，保存kruskal最小生成树的边
    EData *edges;           // 图对应的所有边

    // 获取"图中所有的边"
    edges = getEdges();
    // 将边按照"权"的大小进行排序(从小到大)
    sortEdges(edges, mEdgNum);

    for (i=0; i<mEdgNum; i++)
    {
        p1 = getPosition(edges[i].start);      // 获取第i条边的"起点"的序号
        p2 = getPosition(edges[i].end);        // 获取第i条边的"终点"的序号

        m = getEnd(vends, p1);                 // 获取p1在"已有的最小生成树"中的终点
        n = getEnd(vends, p2);                 // 获取p2在"已有的最小生成树"中的终点
        // 如果m!=n，意味着"边i"与"已经添加到最小生成树中的顶点"没有形成环路
        if (m != n)
        {
            vends[m] = n;                       // 设置m在"已有的最小生成树"中的终点为n
            rets[index++] = edges[i];           // 保存结果
        }
    }
    delete[] edges;

    // 统计并打印"kruskal最小生成树"的信息
    length = 0;
    for (i = 0; i < index; i++)
        length += rets[i].weight;
    cout << "Kruskal=" << length << ": ";
    for (i = 0; i < index; i++)
        cout << "(" << rets[i].start << "," << rets[i].end << ") ";
    cout << endl;
}
```

**References**             
[Prim算法(二)之 C++详解 - 如果天空不死 - 博客园](http://www.cnblogs.com/skywang12345/p/3711507.html)

[Kruskal算法(二)之 C++详解 - 如果天空不死 - 博客园](http://www.cnblogs.com/skywang12345/p/3711500.html)









