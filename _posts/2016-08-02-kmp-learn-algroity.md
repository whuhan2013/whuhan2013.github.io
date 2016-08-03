---
layout: post
title: KMP算法学习
date: 2016-8-2
categories: blog
tags: [数据结构]
description: KMP算法
---   


kmp算法完成的任务是：给定两个字符串O和f，长度分别为n和m，判断f是否在O中出现，如果出现则返回出现的位置。常规方法是遍历a的每一个位置，然后从该位置开始和b进行匹配，但是这种方法的复杂度是O(nm)。kmp算法通过一个O(m)的预处理，使匹配的复杂度降为O(n+m)。


参考链接：[【经典算法】——KMP，深入讲解next数组的求解 - c_cloud - 博客园](http://www.cnblogs.com/c-cloud/p/3224788.html)                
讲解的非常清楚明白

主要就是next数组的求解          

KMP算法的核心所在，就是next数组的求解！不过，在这里我找到了一个全新的理解方法！如果你懂的上面我写的的，那么下面的内容你只需稍微思考一下就行了！
 
跟刚才一样，我用一句话来阐述一下next数组的求解方法，其实也就是两个字：

**继承**      

a、当前面字符的前一个字符的对称程度为0的时候，只要将当前字符与子串第一个字符进行比较。这个很好理解啊，前面都是0，说明都不对称了，如果多加了一个字符，要对称的话最多是当前的和第一个对称。比如agcta这个里面t的是0，那么后面的a的对称程度只需要看它是不是等于第一个字符a了。

b、按照这个推理，我们就可以总结一个规律，不仅前面是0呀，如果前面一个字符的next值是1，那么我们就把当前字符与子串第二个字符进行比较，因为前面的是1，说明前面的字符已经和第一个相等了，如果这个又与第二个相等了，说明对称程度就是2了。有两个字符对称了。比如上面agctag，倒数第二个a的next是1，说明它和第一个a对称了，接着我们就把最后一个g与第二个g比较，又相等，自然对称成都就累加了，就是2了。 

c、按照上面的推理，如果一直相等，就一直累加，可以一直推啊，推到这里应该一点难度都没有吧，如果你觉得有难度说明我写的太失败了。

当然不可能会那么顺利让我们一直对称下去，如果遇到下一个不相等了，那么说明不能继承前面的对称性了，这种情况只能说明没有那么多对称了，但是不能说明一点对称性都没有，所以遇到这种情况就要重新来考虑，这个也是难点所在。

![](http://img.blog.csdn.net/20140309203650890?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTU2NDQ1Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果蓝色的部分相同，则当前next数组的值为上一个next的值加一，如果不相同，就是我们下面要说的！
如果不相同，用一句话来说，就是：

从前面来找子前后缀                 

1、如果要存在对称性，那么对称程度肯定比前面这个的对称程度小，所以要找个更小的对称，这个不用解释了吧，如果大那么就继承前面的对称性了。

2、要找更小的对称，必然在对称内部还存在子对称，而且这个必须紧接着在子对称之后。
 
如果看不懂，那么看一下图吧！

![](http://img.blog.csdn.net/20140309203712906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTU2NDQ1Ng==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

说了这么多，下面是代码实现

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define N 100

void cal_next( char * str, int * next, int len )
{
    int i, j;

    next[0] = -1;
    for( i = 1; i < len; i++ )
    {
        j = next[ i - 1 ];
        while( str[ j + 1 ] != str[ i ] && ( j >= 0 ) )
        {
            j = next[ j ];
        }
        if( str[ i ] == str[ j + 1 ] )
        {
            next[ i ] = j + 1;
        }
        else
        {
            next[ i ] = -1;
        }
    }
}

int KMP( char * str, int slen, char * ptr, int plen, int * next )
{
    int s_i = 0, p_i = 0;

    while( s_i < slen && p_i < plen )
    {
        if( str[ s_i ] == ptr[ p_i ] )
        {
            s_i++;
            p_i++;
        }
        else
        {
            if( p_i == 0 )
            {
                s_i++;
            }
            else
            {
                p_i = next[ p_i - 1 ] + 1;
            }
        }
    }
    return ( p_i == plen ) ? ( s_i - plen ) : -1;
}

int main()
{
    char str[ N ] = {0};
    char ptr[ N ] = {0};
    int slen, plen;
    int next[ N ];

    while( scanf( "%s%s", str, ptr ) )
    {
        slen = strlen( str );
        plen = strlen( ptr );
        cal_next( ptr, next, plen );
        printf( "%d\n", KMP( str, slen, ptr, plen, next ) );
    }
    return 0;
}
```


**参考链接**            
[如果你看不懂KMP算法，那就看一看这篇文章( 绝对原创，绝对通俗易懂) - Stay Hungry,Stay Foolish - 博客频道 - CSDN.NET](http://blog.csdn.net/u011564456/article/details/20862555?utm_source=tuicool&utm_medium=referral)         

[字符串匹配算法总结 - WINCOL的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/WINCOL/article/details/4795369)
