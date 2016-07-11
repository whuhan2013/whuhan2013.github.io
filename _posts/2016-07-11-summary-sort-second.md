---
layout: post
title: 数据结构之排序总结(二)
date: 2016-7-11
categories: blog
tags: [数据结构]
description: 数据结构之排序总结
---


### 选择排序          

选择排序(Selection sort)是一种简单直观的排序算法。
它的基本思想是：首先在未排序的数列中找到最小(or最大)元素，然后将其存放到数列的起始位置；接着，再从剩余未排序的元素中继续寻找最小(or最大)元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

**选择排序时间复杂度**           
选择排序的时间复杂度是O(N2)。                       
假设被排序的数列中有N个数。遍历一趟的时间复杂度是O(N)，需要遍历多少次呢？N-1！因此，选择排序的时间复杂度是O(N2)。
 
**选择排序稳定性**                                   
选择排序是稳定的算法，它满足稳定算法的定义。


```
/**
 * 选择排序：C++
 *
 * @author skywang
 * @date 2014/03/11
 */

#include <iostream>
using namespace std;

/*
 * 选择排序
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     n -- 数组的长度
 */
void selectSort(int* a, int n)
{
    int i;        // 有序区的末尾位置
    int j;        // 无序区的起始位置
    int min;    // 无序区中最小元素位置

    for(i=0; i<n; i++)
    {
        min=i;

        // 找出"a[i+1] ... a[n]"之间的最小元素，并赋值给min。
        for(j=i+1; j<n; j++)
        {
            if(a[j] < a[min])
                min=j;
        }

        // 若min!=i，则交换 a[i] 和 a[min]。
        // 交换之后，保证了a[0] ... a[i] 之间的元素是有序的。
        if(min != i)
        {
            int tmp = a[i];
            a[i] = a[min];
            a[min] = tmp;
        }
    }
}

int main()
{
    int i;
    int a[] = {20,40,30,10,60,50};
    int ilen = (sizeof(a)) / (sizeof(a[0]));

    cout << "before sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    selectSort(a, ilen);

    cout << "after  sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    return 0;
}
```


### 堆排序   

堆排序(Heap Sort)是指利用堆这种数据结构所设计的一种排序算法。
因此，学习堆排序之前，有必要了解堆！若读者不熟悉堆，建议先了解堆(建议可以通过二叉堆，左倾堆，斜堆，二项堆或斐波那契堆等文章进行了解)，然后再来学习本章。
 
我们知道，堆分为"最大堆"和"最小堆"。最大堆通常被用来进行"升序"排序，而最小堆通常被用来进行"降序"排序。
鉴于最大堆和最小堆是对称关系，理解其中一种即可。本文将对最大堆实现的升序排序进行详细说明。


最大堆进行升序排序的基本思想：                   
① 初始化堆：将数列a[1...n]构造成最大堆。                                   
② 交换数据：将a[1]和a[n]交换，使a[n]是a[1...n]中的最大值；然后将a[1...n-1]重新调整为最大堆。 接着，将a[1]和a[n-1]交换，使a[n-1]是a[1...n-1]中的最大值；然后将a[1...n-2]重新调整为最大值。 依次类推，直到整个数列都是有序的。
 
 
 
下面，通过图文来解析堆排序的实现过程。注意实现中用到了"数组实现的二叉堆的性质"。              
在第一个元素的索引为 0 的情形中：              
性质一：索引为i的左孩子的索引是 (2*i+1);        
性质二：索引为i的左孩子的索引是 (2*i+2);             
性质三：索引为i的父结点的索引是 floor((i-1)/2);                

![](http://images.cnitblog.com/i/497634/201403/151545571211442.jpg)


**堆排序时间复杂度**               
堆排序的时间复杂度是O(N*lgN)。            
假设被排序的数列中有N个数。遍历一趟的时间复杂度是O(N)，需要遍历多少次呢？               
堆排序是采用的二叉堆进行排序的，二叉堆就是一棵二叉树，它需要遍历的次数就是二叉树的深度，而根据完全二叉树的定义，它的深度至少是lg(N+1)。最多是多少呢？由于二叉堆是完全二叉树，因此，它的深度最多也不会超过lg(2N)     。因此，遍历一趟的时间复杂度是O(N)，而遍历次数介于lg(N+1)和lg(2N)之间；因此得出它的时间复杂度是O(N*lgN)。
 
堆排序稳定性           
堆排序是不稳定的算法，它不满足稳定算法的定义。它在交换数据的时候，是比较父结点和子节点之间的数据，所以，即便是存在两个数值相等的兄弟节点，它们的相对顺序在排序也可能发生变化。               

```
/**
 * 堆排序：C++
 *
 * @author skywang
 * @date 2014/03/11
 */

#include <iostream>
using namespace std;

/* 
 * (最大)堆的向下调整算法
 *
 * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
 *     其中，N为数组下标索引值，如数组中第1个数对应的N为0。
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     start -- 被下调节点的起始位置(一般为0，表示从第1个开始)
 *     end   -- 截至范围(一般为数组中最后一个元素的索引)
 */
void maxHeapDown(int* a, int start, int end)
{
    int c = start;            // 当前(current)节点的位置
    int l = 2*c + 1;        // 左(left)孩子的位置
    int tmp = a[c];            // 当前(current)节点的大小
    for (; l <= end; c=l,l=2*l+1)
    {
        // "l"是左孩子，"l+1"是右孩子
        if ( l < end && a[l] < a[l+1])
            l++;        // 左右两孩子中选择较大者，即m_heap[l+1]
        if (tmp >= a[l])
            break;        // 调整结束
        else            // 交换值
        {
            a[c] = a[l];
            a[l]= tmp;
        }
    }
}

/*
 * 堆排序(从小到大)
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     n -- 数组的长度
 */
void heapSortAsc(int* a, int n)
{
    int i,tmp;

    // 从(n/2-1) --> 0逐次遍历。遍历之后，得到的数组实际上是一个(最大)二叉堆。
    for (i = n / 2 - 1; i >= 0; i--)
        maxHeapDown(a, i, n-1);

    // 从最后一个元素开始对序列进行调整，不断的缩小调整的范围直到第一个元素
    for (i = n - 1; i > 0; i--)
    {
        // 交换a[0]和a[i]。交换后，a[i]是a[0...i]中最大的。
        tmp = a[0];
        a[0] = a[i];
        a[i] = tmp;
        // 调整a[0...i-1]，使得a[0...i-1]仍然是一个最大堆。
        // 即，保证a[i-1]是a[0...i-1]中的最大值。
        maxHeapDown(a, 0, i-1);
    }
}

/* 
 * (最小)堆的向下调整算法
 *
 * 注：数组实现的堆中，第N个节点的左孩子的索引值是(2N+1)，右孩子的索引是(2N+2)。
 *     其中，N为数组下标索引值，如数组中第1个数对应的N为0。
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     start -- 被下调节点的起始位置(一般为0，表示从第1个开始)
 *     end   -- 截至范围(一般为数组中最后一个元素的索引)
 */
void minHeapDown(int* a, int start, int end)
{
    int c = start;            // 当前(current)节点的位置
    int l = 2*c + 1;        // 左(left)孩子的位置
    int tmp = a[c];            // 当前(current)节点的大小
    for (; l <= end; c=l,l=2*l+1)
    {
        // "l"是左孩子，"l+1"是右孩子
        if ( l < end && a[l] > a[l+1])
            l++;        // 左右两孩子中选择较小者
        if (tmp <= a[l])
            break;        // 调整结束
        else            // 交换值
        {
            a[c] = a[l];
            a[l]= tmp;
        }
    }
}

/*
 * 堆排序(从大到小)
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     n -- 数组的长度
 */
void heapSortDesc(int* a, int n)
{
    int i,tmp;

    // 从(n/2-1) --> 0逐次遍历每。遍历之后，得到的数组实际上是一个最小堆。
    for (i = n / 2 - 1; i >= 0; i--)
        minHeapDown(a, i, n-1);

    // 从最后一个元素开始对序列进行调整，不断的缩小调整的范围直到第一个元素
    for (i = n - 1; i > 0; i--)
    {
        // 交换a[0]和a[i]。交换后，a[i]是a[0...i]中最小的。
        tmp = a[0];
        a[0] = a[i];
        a[i] = tmp;
        // 调整a[0...i-1]，使得a[0...i-1]仍然是一个最小堆。
        // 即，保证a[i-1]是a[0...i-1]中的最小值。
        minHeapDown(a, 0, i-1);
    }
}

int main()
{
    int i;
    int a[] = {20,30,90,40,70,110,60,10,100,50,80};
    int ilen = (sizeof(a)) / (sizeof(a[0]));

    cout << "before sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    heapSortAsc(a, ilen);            // 升序排列
    //heapSortDesc(a, ilen);        // 降序排列

    cout << "after  sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    return 0;
}
```


### 归并排序 

将两个的有序数列合并成一个有序数列，我们称之为"归并"。
归并排序(Merge Sort)就是利用归并思想对数列进行排序。根据具体的实现，归并排序包括"从上往下"和"从下往上"2种方式。

1. 从下往上的归并排序：将待排序的数列分成若干个长度为1的子数列，然后将这些数列两两合并；得到若干个长度为2的有序数列，再将这些数列两两合并；得到若干个长度为4的有序数列，再将它们两两合并；直接合并成一个数列为止。这样就得到了我们想要的排序结果。(参考下面的图片)               
 
2. 从上往下的归并排序：它与"从下往上"在排序上是反方向的。它基本包括3步：      
① 分解 -- 将当前区间一分为二，即求分裂点 mid = (low + high)/2;           
② 求解 -- 递归地对两个子区间a[low...mid] 和 a[mid+1...high]进行归并排序。递归的终结条件是子区间长度为1。      
③ 合并 -- 将已排序的两个子区间a[low...mid]和 a[mid+1...high]归并为一个有序的区间a[low...high]。        


![](http://images.cnitblog.com/i/497634/201403/151853346211212.jpg)

**归并排序时间复杂度**            
归并排序的时间复杂度是O(N*lgN)。           
假设被排序的数列中有N个数。遍历一趟的时间复杂度是O(N)，需要遍历多少次呢？               
归并排序的形式就是一棵二叉树，它需要遍历的次数就是二叉树的深度，而根据完全二叉树的可以得出它的时间复杂度是O(N*lgN)。
 
**归并排序稳定性**                           
归并排序是稳定的算法，它满足稳定算法的定义。


```
/**
 * 归并排序：C++
 *
 * @author skywang
 * @date 2014/03/12
 */

#include <iostream>
using namespace std;

/*
 * 将一个数组中的两个相邻有序区间合并成一个
 *
 * 参数说明：
 *     a -- 包含两个有序区间的数组
 *     start -- 第1个有序区间的起始地址。
 *     mid   -- 第1个有序区间的结束地址。也是第2个有序区间的起始地址。
 *     end   -- 第2个有序区间的结束地址。
 */
void merge(int* a, int start, int mid, int end)
{
    int *tmp = new int[end-start+1];    // tmp是汇总2个有序区的临时区域
    int i = start;            // 第1个有序区的索引
    int j = mid + 1;        // 第2个有序区的索引
    int k = 0;                // 临时区域的索引

    while(i <= mid && j <= end)
    {
        if (a[i] <= a[j])
            tmp[k++] = a[i++];
        else
            tmp[k++] = a[j++];
    }

    while(i <= mid)
        tmp[k++] = a[i++];

    while(j <= end)
        tmp[k++] = a[j++];

    // 将排序后的元素，全部都整合到数组a中。
    for (i = 0; i < k; i++)
        a[start + i] = tmp[i];

    delete[] tmp;
}

/*
 * 归并排序(从上往下)
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     start -- 数组的起始地址
 *     endi -- 数组的结束地址
 */
void mergeSortUp2Down(int* a, int start, int end)
{
    if(a==NULL || start >= end)
        return ;

    int mid = (end + start)/2;
    mergeSortUp2Down(a, start, mid); // 递归排序a[start...mid]
    mergeSortUp2Down(a, mid+1, end); // 递归排序a[mid+1...end]

    // a[start...mid] 和 a[mid...end]是两个有序空间，
    // 将它们排序成一个有序空间a[start...end]
    merge(a, start, mid, end);
}


/*
 * 对数组a做若干次合并：数组a的总长度为len，将它分为若干个长度为gap的子数组；
 *             将"每2个相邻的子数组" 进行合并排序。
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     len -- 数组的长度
 *     gap -- 子数组的长度
 */
void mergeGroups(int* a, int len, int gap)
{
    int i;
    int twolen = 2 * gap;    // 两个相邻的子数组的长度

    // 将"每2个相邻的子数组" 进行合并排序。
    for(i = 0; i+2*gap-1 < len; i+=(2*gap))
    {
        merge(a, i, i+gap-1, i+2*gap-1);
    }

    // 若 i+gap-1 < len-1，则剩余一个子数组没有配对。
    // 将该子数组合并到已排序的数组中。
    if ( i+gap-1 < len-1)
    {
        merge(a, i, i + gap - 1, len - 1);
    }
}

/*
 * 归并排序(从下往上)
 *
 * 参数说明：
 *     a -- 待排序的数组
 *     len -- 数组的长度
 */
void mergeSortDown2Up(int* a, int len)
{
    int n;

    if (a==NULL || len<=0)
        return ;

    for(n = 1; n < len; n*=2)
        mergeGroups(a, len, n);
}

int main()
{
    int i;
    int a[] = {80,30,60,40,20,10,50,70};
    int ilen = (sizeof(a)) / (sizeof(a[0]));

    cout << "before sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    mergeSortUp2Down(a, 0, ilen-1);        // 归并排序(从上往下)
    //mergeSortDown2Up(a, ilen);            // 归并排序(从下往上)

    cout << "after  sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    return 0;
}
```


### 桶排序      

桶排序(Bucket Sort)的原理很简单，它是将数组分到有限数量的桶子里。
 
假设待排序的数组a中共有N个整数，并且已知数组a中数据的范围[0, MAX)。在桶排序时，创建容量为MAX的桶数组r，并将桶数组元素都初始化为0；将容量为MAX的桶数组中的每一个单元都看作一个"桶"。
在排序时，逐个遍历数组a，将数组a的值，作为"桶数组r"的下标。当a中数据被读取时，就将桶的值加1。例如，读取到数组a[3]=5，则将r[5]的值+1。

```
/**
 * 桶排序：C++
 *
 * @author skywang
 * @date 2014/03/13
 */

#include <iostream>
#include <cstring>
using namespace std;

/*
 * 桶排序
 *
 * 参数说明：
 *     a -- 待排序数组
 *     n -- 数组a的长度
 *     max -- 数组a中最大值的范围
 */
void bucketSort(int* a, int n, int max)
{
    int i, j;
    int *buckets;

    if (a==NULL || n<1 || max<1)
        return ;

    // 创建一个容量为max的数组buckets，并且将buckets中的所有数据都初始化为0。
    if ((buckets = new int[max])==NULL)
        return ;
    memset(buckets, 0, max*sizeof(int));

    // 1. 计数
    for(i = 0; i < n; i++) 
        buckets[a[i]]++; 

    // 2. 排序
    for (i = 0, j = 0; i < max; i++) 
        while( (buckets[i]--) >0 )
            a[j++] = i;

    delete[] buckets;
}


int main()
{
    int i;
    int a[] = {8,2,3,4,3,6,6,3,9};
    int ilen = (sizeof(a)) / (sizeof(a[0]));

    cout << "before sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    bucketSort(a, ilen, 10); // 桶排序

    cout << "after  sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    return 0;
}
```


### 基数排序  


基数排序(Radix Sort)是桶排序的扩展，它的基本思想是：将整数按位数切割成不同的数字，然后按每个位数分别比较。
具体做法是：将所有待比较数值统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列。


![](http://images.cnitblog.com/i/497634/201403/161837176365265.jpg)

在上图中，首先将所有待比较树脂统一为统一位数长度，接着从最低位开始，依次进行排序。      
1. 按照个位数进行排序。        
2. 按照十位数进行排序。               
3. 按照百位数进行排序。       
排序后，数列就变成了一个有序序列。             


```
/**
 * 基数排序：C++
 *
 * @author skywang
 * @date 2014/03/15
 */

#include<iostream>
using namespace std;

/*
 * 获取数组a中最大值
 *
 * 参数说明：
 *     a -- 数组
 *     n -- 数组长度
 */
int getMax(int a[], int n)
{
    int i, max;

    max = a[0];
    for (i = 1; i < n; i++)
        if (a[i] > max)
            max = a[i];
    return max;
}

/*
 * 对数组按照"某个位数"进行排序(桶排序)
 *
 * 参数说明：
 *     a -- 数组
 *     n -- 数组长度
 *     exp -- 指数。对数组a按照该指数进行排序。
 *
 * 例如，对于数组a={50, 3, 542, 745, 2014, 154, 63, 616}；
 *    (01) 当exp=1表示按照"个位"对数组a进行排序
 *    (02) 当exp=10表示按照"十位"对数组a进行排序
 *    (03) 当exp=100表示按照"百位"对数组a进行排序
 *    ...
 */
void countSort(int a[], int n, int exp)
{
    int output[n];             // 存储"被排序数据"的临时数组
    int i, buckets[10] = {0};

    // 将数据出现的次数存储在buckets[]中
    for (i = 0; i < n; i++)
        buckets[ (a[i]/exp)%10 ]++;

    // 更改buckets[i]。目的是让更改后的buckets[i]的值，是该数据在output[]中的位置。
    for (i = 1; i < 10; i++)
        buckets[i] += buckets[i - 1];

    // 将数据存储到临时数组output[]中
    for (i = n - 1; i >= 0; i--)
    {
        output[buckets[ (a[i]/exp)%10 ] - 1] = a[i];
        buckets[ (a[i]/exp)%10 ]--;
    }

    // 将排序好的数据赋值给a[]
    for (i = 0; i < n; i++)
        a[i] = output[i];
}

/*
 * 基数排序
 *
 * 参数说明：
 *     a -- 数组
 *     n -- 数组长度
 */
void radixSort(int a[], int n)
{
    int exp;    // 指数。当对数组按各位进行排序时，exp=1；按十位进行排序时，exp=10；...
    int max = getMax(a, n);    // 数组a中的最大值

    // 从个位开始，对数组a按"指数"进行排序
    for (exp = 1; max/exp > 0; exp *= 10)
        countSort(a, n, exp);
}

int main()
{
    int i;
    int a[] = {53, 3, 542, 748, 14, 214, 154, 63, 616};
    int ilen = (sizeof(a)) / (sizeof(a[0]));

    cout << "before sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    radixSort(a, ilen);    // 基数排序

    cout << "after  sort:";
    for (i=0; i<ilen; i++)
        cout << a[i] << " ";
    cout << endl;

    return 0;
}
```



