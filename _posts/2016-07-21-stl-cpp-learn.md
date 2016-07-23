---
layout: post
title: STL学习
date: 2016-7-21
categories: blog
tags: [数据结构]
description: STL
---


 **STL简介**
 
 
1 概况.......................................................... 2     
 
1.1 STL是什么............................................... 2     
 
1.2 为什么我们需要学习STL................................... 2
 
1.3 初识STL................................................. 2
 
1.4 STL 的组成.............................................. 5
 
2 容器.......................................................... 6
 
2.1 基本容器——向量（vector）.............................. 6
 
2.2  双端队列（deque容器类）............................... 9
 
2.3 表（List容器类）....................................... 10
 
2.4 集和多集（set 和multiset 容器类）：.................... 12
 
2.5 映射和多重映射（map 和multimap）....................... 13
 
3 算法（algorithm）：.......................................... 15
 
3.1 翻转和复制（reverse()和copy()）........................ 15
 
3.2 单值交换（Swap()）..................................... 16
 
3.3 查找（find()）......................................... 17
 
3.4 得到数目（Count()）.................................... 18
 
3.5 排序（sort()）......................................... 19
 
4 迭代器（itertor）............................................ 21
 
5 STL的其他标准组件........................................... 22
 
5.1 函数对象（functor或者funtion objects）................ 22
 
5.2 适配器（adapter）...................................... 23
 
 

**STL 的组成**
 
STL有三大核心部分：容器（Container）、算法（Algorithms）、迭代器（Iterator），容器适配器（container adaptor），函数对象(functor)，除此之外还有STL其他标准组件。通俗的讲： 

- 容器：装东西的东西，装水的杯子，装咸水的大海，装人的教室……STL里的容器是可容纳一些数据的模板类。 

- 算法：就是往杯子里倒水，往大海里排污，从教室里撵人……STL里的算法，就是处理容器里面数据的方法、操作。 

- 迭代器：往杯子里倒水的水壶，排污的管道，撵人的那个物业管理人员……STL里的迭代器：遍历容器中数据的对象。对存储于容器中的数据进行处理时，迭代器能从一个成员移向另一个成员。他能按预先定义的顺序在某些容器中的成员间移动。对普通的一维数组、向量、双端队列和列表来说，迭代器是一种指针。
 
下面让我们来看看专家是怎么说的：
 
- 容器（container）：容器是数据在内存中组织的方法，例如，数组、堆栈、队列、链表或二叉树（不过这些都不是STL标准容器）。STL中的容器是一种存储T（Template）类型值的有限集合的数据结构,容器的内部实现一般是类。这些值可以是对象本身，如果数据类型T代表的是Class的话。 

- 算法（algorithm）：算法是应用在容器上以各种方法处理其内容的行为或功能。例如，有对容器内容排序、复制、检索和合并的算法。在STL中，算法是由模板函数表现的。这些函数不是容器类的成员函数。相反，它们是独立的函数。令人吃惊的特点之一就是其算法如此通用。不仅可以将其用于STL容器，而且可以用于普通的C＋＋数组或任何其他应用程序指定的容器。 

- 迭代器(iterator)：一旦选定一种容器类型和数据行为(算法)，那么剩下唯一要他做的就是用迭代器使其相互作用。可以把达代器看作一个指向容器中元素的普通指针。可以如递增一个指针那样递增迭代器，使其依次指向容器中每一个后继的元素。迭代器是STL的一个关键部分，因为它将算法和容器连在一起。 

下面我将依次介绍STL的这三个主要组件。


### 容器      
STL中的容器有队列容器和关联容器，容器适配器（congtainer adapters：stack,queue，priority queue），位集（bit_set），串包(string_package)等等。

在本文中，我将介绍list,vector，deque等队列容器，和set和multisets,map和multimaps等关联容器，一共7种基本容器类。

队列容器（顺序容器）：队列容器按照线性排列来存储T类型值的集合，队列的每个成员都有自己的特有的位置。顺序容器有向量类型、双端队列类型、列表类型三种。

#### 基本容器——向量（vector）
 
向量（vector容器类）：＃include <vector>，vector是一种动态数组，是基本数组的类模板。其内部定义了很多基本操作。既然这是一个类，那么它就会有自己的构造函数。vector 类中定义了4中种构造函数：

- 默认构造函数，构造一个初始长度为0的空向量，如：vector<int> v1; 
- 带有单个整形参数的构造函数，此参数描述了向量的初始大小。这个构造函数还有一个可选的参数，这是一个类型为T的实例，描述了各个向量种各成员的初始值；如：vector<int> v2(n,0); 如果预先定义了：n,他的成员值都被初始化为0； 
- 复制构造函数，构造一个新的向量，作为已存在的向量的完全复制，如：vector<int> v3(v2); 
- 带两个常量参数的构造函数，产生初始值为一个区间的向量。区间由一个半开区间[first,last) 来指定。如：vector<int> v4(first,last） 

下面一个例子用的是第四种构造方法，其它的方法读者可以自己试试。

```
//程序：初始化演示
 
#include <cstring>  

#include <vector>
 
#include <iostream>
 
using namespace std;
 
 
 
int ar[10] = {  12, 45, 234, 64, 12, 35, 63, 23, 12, 55  };
 
char* str = "Hello World";
 
 
 
int main()
 
{
 
    vector <int> vec1(ar, ar+10);   //first=ar,last=ar+10,不包括ar+10
 
    vector < char > vec2(str,str+strlen(str)); //first=str,last= str+strlen(str), 

    cout<<"vec1:"<<endl;   

    //打印vec1和vec2，const_iterator是迭代器，后面会讲到
 
    //当然，也可以用for (int i=0; i<vec1.size(); i++)cout << vec[i];输出
 
    //size()是vector的一个成员函数
 
    for(vector<int>::const_iterator p=vec1.begin();p!=vec1.end(); ++p)
 
        cout<<*p;
 
        cout<<'\n'<<"vec2:"<<endl;
 
    for(vector< char >::const_iterator p1=vec2.begin();p1!=vec2.end(); ++p1)
 
        cout<<*p1;
 
    cout<<'\n';
 
    return 0;
 
}     
```

为了帮助理解向量的概念，这里写了一个小例子，其中用到了vector的成员函数：begin()，end()，push_back()，assign()，front()，back()，erase()，empty()，at()，size()。
 
 
```
#include <iostream>
 
#include <vector>
 
using namespace std;
 
 
 
typedef vector<int> INTVECTOR;//自定义类型INTVECTOR
 
//测试vector容器的功能
 
 
 
int main()
 
{
 
    //vec1对象初始为空
 
    INTVECTOR vec1;   

    //vec2对象最初有10个值为6的元素  

    INTVECTOR vec2(10,6);  

    //vec3对象最初有3个值为6的元素，拷贝构造
 
    INTVECTOR vec3(vec2.begin(),vec2.begin()+3);  

    //声明一个名为i的双向迭代器
 
    INTVECTOR::iterator i;
 
    //从前向后显示vec1中的数据
 
    cout<<"vec1.begin()--vec1.end():"<<endl;
 
    for (i =vec1.begin(); i !=vec1.end(); ++i)
 
        cout << *i << " ";
 
    cout << endl;
 
    //从前向后显示vec2中的数据
 
    cout<<"vec2.begin()--vec2.end():"<<endl;
 
    for (i =vec2.begin(); i !=vec2.end(); ++i)
 
        cout << *i << " ";
 
    cout << endl;
 
    //从前向后显示vec3中的数据
 
    cout<<"vec3.begin()--vec3.end():"<<endl;
 
    for (i =vec3.begin(); i !=vec3.end(); ++i)
 
        cout << *i << " ";
 
    cout << endl;
 
    //测试添加和插入成员函数，vector不支持从前插入
 
    vec1.push_back(2);//从后面添加一个成员
 
    vec1.push_back(4);
 
    vec1.insert(vec1.begin()+1,5);//在vec1第一个的位置上插入成员5
 
    //从vec1第一的位置开始插入vec3的所有成员
 
    vec1.insert(vec1.begin()+1,vec3.begin(),vec3.end());
 
    cout<<"after push() and insert() now the vec1 is:" <<endl;
 
    for (i =vec1.begin(); i !=vec1.end(); ++i)
 
        cout << *i << " ";
 
    cout << endl;
 
    //测试赋值成员函数
 
    vec2.assign(8,1);   // 重新给vec2赋值，8个成员的初始值都为1
 
    cout<<"vec2.assign(8,1):" <<endl;
 
    for (i =vec2.begin(); i !=vec2.end(); ++i)
 
        cout << *i << " ";
 
    cout << endl;
 
    //测试引用类函数
 
    cout<<"vec1.front()="<<vec1.front()<<endl;//vec1第零个成员
 
    cout<<"vec1.back()="<<vec1.back()<<endl;//vec1的最后一个成员
 
    cout<<"vec1.at(4)="<<vec1.at(4)<<endl;//vec1的第五个成员
 
    cout<<"vec1[4]="<<vec1[4]<<endl;
 
    //测试移出和删除
 
    vec1.pop_back();//将最后一个成员移出vec1
 
    vec1.erase(vec1.begin()+1,vec1.end()-2);//删除成员
 
    cout<<"vec1.pop_back() and vec1.erase():" <<endl;
 
    for (i =vec1.begin(); i !=vec1.end(); ++i)
 
        cout << *i << " ";
 
    cout << endl;
 
    //显示序列的状态信息
 
    cout<<"vec1.size(): "<<vec1.size()<<endl;//打印成员个数
 
    cout<<"vec1.empty(): "<<vec1.empty()<<endl;//清空
 
}
```

 
push_back()是将数据放入vector（向量）或deque（双端队列）的标准函数。Insert()是一个与之类似的函数，然而它在所有容器中都可以使用，但是用法更加复杂。end()实际上是取末尾加一，以便让循环正确运行--它返回的指针指向最靠近数组界限的数据。
 
在Java里面也有向量的概念。Java中的向量是对象的集合。其中，各元素可以不必同类型，元素可以增加和删除，不能直接加入原始数据类型。


#### 双端队列（deque容器类）

deque（读音：deck，意即：double queue，#include<qeque>）容器类与vector类似，支持随机访问和快速插入删除，它在容器中某一位置上的操作所花费的是线性时间。与vector不同的是，deque还支持从开始端插入数据：push_front()。此外deque也不支持与vector的capacity()、reserve()类似的操作。 

```
#include <iostream>
 
#include <deque>
 
using namespace std;
 
 
 
typedef deque<int> INTDEQUE;//有些人很讨厌这种定义法，呵呵
 
 
 
//从前向后显示deque队列的全部元素
 
void put_deque(INTDEQUE deque, char *name)
 
{
 
    INTDEQUE::iterator pdeque;//仍然使用迭代器输出
 
    cout << "The contents of " << name << " : ";
 
    for(pdeque = deque.begin(); pdeque != deque.end(); pdeque++)
 
        cout << *pdeque << " ";//注意有 "*"号哦，没有"*"号的话会报错
 
    cout<<endl;
 
}
 
 
 
//测试deqtor容器的功能
 
int main()
 
{
 
    //deq1对象初始为空
 
    INTDEQUE deq1;   

    //deq2对象最初有10个值为6的元素  

    INTDEQUE deq2(10,6);  

    //声明一个名为i的双向迭代器变量
 
    INTDEQUE::iterator i;
 
    //从前向后显示deq1中的数据
 
    put_deque(deq1,"deq1");
 
    //从前向后显示deq2中的数据
 
    put_deque(deq2,"deq2");
 
    //从deq1序列后面添加两个元素
 
    deq1.push_back(2);
 
    deq1.push_back(4);
 
    cout<<"deq1.push_back(2) and deq1.push_back(4):"<<endl;
 
    put_deque(deq1,"deq1");
 
    //从deq1序列前面添加两个元素
 
    deq1.push_front(5);
 
    deq1.push_front(7);
 
    cout<<"deq1.push_front(5) and deq1.push_front(7):"<<endl;
 
    put_deque(deq1,"deq1");
 
    //在deq1序列中间插入数据
 
    deq1.insert(deq1.begin()+1,3,9);
 
    cout<<"deq1.insert(deq1.begin()+1,3,9):"<<endl;
 
    put_deque(deq1,"deq1");
 
    //测试引用类函数
 
    cout<<"deq1.at(4)="<<deq1.at(4)<<endl;
 
    cout<<"deq1[4]="<<deq1[4]<<endl;
 
    deq1.at(1)=10;
 
    deq1[2]=12;
 
    cout<<"deq1.at(1)=10 and deq1[2]=12 :"<<endl;
 
    put_deque(deq1,"deq1");
 
    //从deq1序列的前后各移去一个元素
 
    deq1.pop_front();
 
    deq1.pop_back();
 
    cout<<"deq1.pop_front() and deq1.pop_back():"<<endl;
 
    put_deque(deq1,"deq1");
 
    //清除deq1中的第2个元素
 
    deq1.erase(deq1.begin()+1);
 
    cout<<"deq1.erase(deq1.begin()+1):"<<endl;
 
    put_deque(deq1,"deq1");
 
    //对deq2赋值并显示
 
    deq2.assign(8,1);
 
    cout<<"deq2.assign(8,1):"<<endl;
 
    put_deque(deq2,"deq2");
 
}
```

 
上面我们演示了deque如何进行插入删除等操作，像erase(),assign()是大多数容器都有的操作。关于deque的其他操作请参阅其他书籍。


#### 表（List容器类）
 
List（#include<list>）又叫链表，是一种双线性列表，只能顺序访问（从前向后或者从后向前），图2是list的数据组织形式。与前面两种容器类有一个明显的区别就是：它不支持随机访问。要访问表中某个下标处的项需要从表头或表尾处（接近该下标的一端）开始循环。而且缺少下标预算符：operator[]。
 

同时，list仍然包涵了erase(),begin(),end(),insert(),push_back(),push_front()这些基本函数，下面我们来演示一下list的其他函数功能。merge()：合并两个排序列表；splice()：拼接两个列表；sort()：列表的排序。

```
#include <iostream>
 
#include <string>
 
#include <list>
 
using namespace std;
 
 
 
void PrintIt(list<int> n)
 
{
 
    for(list<int>::iterator iter=n.begin(); iter!=n.end(); ++iter)
 
      cout<<*iter<<" ";//用迭代器进行输出循环 

}
 
    

int main()
 
{
 
    list<int> listn1,listn2;    //给listn1,listn2初始化 

    listn1.push_back(123);
 
    listn1.push_back(0);
 
    listn1.push_back(34);
 
    listn1.push_back(1123);    //now listn1:123,0,34,1123 

    listn2.push_back(100);
 
    listn2.push_back(12);    //now listn2:12,100
 
    listn1.sort();
 
    listn2.sort();    //给listn1和listn2排序
 
    //now listn1:0,34,123,1123         listn2:12,100 

    PrintIt(listn1);
 
    cout<<endl;
 
    PrintIt(listn2);
 
    listn1.merge(listn2);    //合并两个排序列表后,listn1:0，12，34，100，123，1123 

    cout<<endl;
 
    PrintIt(listn1);
 
}
```

上面并没有演示splice()函数的用法，这是一个拗口的函数。用起来有点麻烦。图3所示是splice函数的功能。将一个列表插入到另一个列表当中。list容器类定义了splice()函数的3个版本： 

splice(position,list_value);
 
splice(position,list_value,ptr);
 
splice(position,list_value,first,last);
 
list_value是一个已存在的列表，它将被插入到源列表中，position是一个迭代参数，他当前指向的是要进行拼接的列表中的特定位置。

listn1:123,0,34,1123   listn2:12,100

执行listn1.splice(find(listn1.begin(),listn1.end(),0),listn2);之后，listn1将变为：123，12，100，0,34，1123。即把listn2插入到listn1的0这个元素之前。其中，find()函数找到0这个元素在listn1中的位置。值得注意的是，在执行splice之后，list_value将不复存在了。这个例子中是listn2将不再存在。

第二个版本当中的ptr是一个迭代器参数，执行的结果是把ptr所指向的值直接插入到position当前指向的位置之前.这将只向源列表中插入一个元素。

第三个版本的first和last也是迭代器参数，并不等于list_value.begin(),list_value.end()。First指的是要插入的列的第一个元素，last指的是要插入的列的最后一个元素。
 
如果listn1:123,0,34,1123 listn2:12,43，87，100 执行完以下函数之后
 
listn1.splice(find(listn1.begin(),listn1.end(),0),++listn2.begin(),--listn2.end());
 
listn1:123,43,87,0,34,1123  listn2:12,100
 
以上，我们学习了vector,deque,list三种基本顺序容器，其他的顺序容器还有：slist,bit_vector等等。



### 算法（algorithm）
 
#inlcude <algorithm> 

STL中算法的大部分都不作为某些特定容器类的成员函数，他们是泛型的，每个算法都有处理大量不同容器类中数据的使用。值得注意的是，STL中的算法大多有多种版本，用户可以依照具体的情况选择合适版本。中在STL的泛型算法中有4类基本的算法： 

- 变序型队列算法：可以改变容器内的数据； 

- 非变序型队列算法：处理容器内的数据而不改变他们； 

- 排序值算法：包涵对容器中的值进行排序和合并的算法，还有二叉搜索算法、通用数值算法。（注：STL的算法并不只是针对STL容器，对一般容器也是适用的。）
 
- 变序型队列算法：又叫可修改的序列算法。这类算法有复制（copy）算法、交换（swap）算法、替代（replace）算法、删除（clear）算法，移动（remove）算法、翻转（reverse）算法等等。这些算法可以改变容器中的数据（数据值和值在容器中的位置）。

#### 翻转和复制（reverse()和copy()）
 
下面介绍2个比较常用的算法reverse()和copy()。 

```
#include <iostream>
 
#include <algorithm>
 
#include <iterator>
 
//下面用到了输出迭代器ostream_iterator
 
using namespace std;
 
 
 
int main()
 
{
 
    int arr[6]={1,12,3,2,1215,90};
 
    int arr1[7];
 
    int arr2[6]={2,5,6,9,0,-56};
 
    copy(arr,(arr+6),arr1);//将数组aar复制到arr1
 
    cout<<"arr[6] copy to arr1[7],now arr1: "<<endl;
 
    for(int i=0;i<7;i++)
 
        cout<<" "<<arr1[i];
 
    reverse(arr,arr+6);//将排好序的arr翻转
 
    cout<<'\n'<<"arr reversed ,now arr:"<<endl;
 
    copy(arr,arr+6,ostream_iterator<int>(cout, " "));//复制到输出迭代器
 
    swap_ranges(arr,arr+6,arr2);//交换arr和arr2序列
 
    cout<<'\n'<<"arr swaped to arr2,now arr:"<<endl;
 
    copy(arr,arr+6,ostream_iterator<int>(cout, " "));
 
    cout<<'\n'<<"arr2:"<<endl;
 
    copy(arr2,arr2+6,ostream_iterator<int>(cout, " "));
 
    return 0;
 
}
```
 
revese()的功能是将一个容器内的数据顺序翻转过来，它的原型是： 

template<class Bidirectional>
 
void reverse(Bidirectional first, Bidirectional last);
 
将first和last之间的元素翻转过来，上例中你也可以只将arr中的一部分进行翻转： 

reverse(arr+3,arr+6); 这也是有效的。First和last需要指定一个操作区间。
 
Copy()是要将一个容器内的数据复制到另一个容器内，它的原型是： 

  Template<class InputIterator ，class OutputIterator>
 
  OutputIterator copy(InputIterator first, InputIterator last, OutputIterator result);
 
它把[first,last－1]内的队列成员复制到区间[result,result+(last-first)-1]中。


**泛型交换算法：单值交换（Swap())**
 
Swap()操作的是单值交换，它的原型是： 

template<class T>
 
void swap(T& a,T& b);
 
swap_ranges()操作的是两个相等大小区间中的值，它的原型是： 

template<class ForwardIterator1, class ForwardIterator2>
 
ForwardIterator2swap_ranges(ForwardIterator1 first1,ForwardIterator1 last1, ForwardIterator1 first2);
 
交换区间[first1,last1-1]和[first2, first2+(last1-first1)-1]之间的值，并假设这两个区间是不重叠的。 

非变序型队列算法，又叫不可修改的序列算法。这一类算法操作不影响其操作的容器的内容，包括搜索队列成员算法，等价性检查算法，计算队列成员个数的算法。我将用下面的例子介绍其中的find(),search(),count()： 

```
#include <iostream>
 
#include <vector>
 
#include <algorithm>
 
using namespace std;
 
 
 
int main()
 
{
 
    int a[10]={12,31,5,2,23,121,0,89,34,66};
 
    vector<int> v1(a,a+10);
 
    vector<int>::iterator result1,result2;//result1和result2是随机访问迭代器
 
    result1=find(v1.begin(),v1.end(),2);
 
    //在v1中找到2，result1指向v1中的2 

    result2=find(v1.begin(),v1.end(),8);
 
    //在v1中没有找到8，result2指向的是v1.end() 

    cout<<result1-v1.begin()<<endl; //3－0＝3或4－1＝3，屏幕结果是3
 
    cout<<result2-v1.end()<<endl;    

    int b[9]={5,2,23,54,5,5,5,2,2};
 
    vector<int> v2(a+2,a+8);
 
    vector<int> v3(b,b+4);
 
    result1=search(v1.begin(),v1.end(),v2.begin(),v2.end());
 
    cout<<*result1<<endl; 

    //在v1中找到了序列v2，result1指向v2在v1中开始的位置 

     result1=search(v1.begin(),v1.end(),v3.begin(),v3.end());
 
     cout<<*(result1-1)<<endl;
 
    //在v1中没有找到序列v3，result指向v1.end(),屏幕打印出v1的最后一个元素66    

     vector<int> v4(b,b+9); 

     int i=count(v4.begin(),v4.end(),5);
 
     int j=count(v4.begin(),v4.end(),2);
 
     cout<<"there are "<<i<<" members in v4 equel to 5"<<endl;
 
     cout<<"there are "<<j<<" members in v4 equel to 2"<<endl; 

     //计算v4中有多少个成员等于 5,2
 
     return 0;        

}
```

查找（find()）
 
**find()的原型是：**

template<class InputIterator，class EqualityComparable>
 
InputIterator find(InputIterator first, InputIterator last, const EqualityComparable& value);
 
其功能是在序列[first,last-1]中查找value值，如果找到，就返回一个指向value在序列中第一次出现的迭代，如果没有找到，就返回一个指向last的迭代（last并不属于序列）。
 
**search()的原型是：**

template <class ForwardIterator1, class ForwardIterator2>
 
ForwardIterator1 search(ForwardIterator1 first1, ForwardIterator1 last1,                        ForwardIterator2 first2, ForwardIterator2 last2);
 
其功能是在源序列[first1,last1-1]查找目标序列[first2，last2-1]如果查找成功，就返回一个指向源序列中目标序列出现的首位置的迭代。查找失败则返回一个指向last的迭代。
 
得到数目（Count()）
 
**Count()的原型是：**

template <class InputIterator, class EqualityComparable>
 
iterator_traits<InputIterator>::difference_type count(InputIterator first, 

InputIterator last, const EqualityComparable& value);
 
其功能是在序列[first,last-1]中查找出等于value的成员，返回等于value得成员的个数。



**排序算法（sort algorithm）             
这一类算法很多，功能强大同时也相对复杂一些。这些算法依赖的是关系运算。在这里我只介绍其中比较简单的几种排序算法：sort(),merge(),includes()            

```
#include <iostream>
 
#include <algorithm>
 
using namespace std;
 
 
 
int main()
 
{
 
    int a[10]={12,0,5,3,6,8,9,34,32,18};
 
    int b[5]={5,3,6,8,9};
 
    int d[15];
 
    sort(a,a+10);
 
    for(int i=0;i<10;i++)
 
      cout<<" "<<a[i];
 
    sort(b,b+5);
 
    if(includes(a,a+10,b,b+5))
 
       cout<<'\n'<<"sorted b members are included in a."<<endl;
 
    else
 
       cout<<"sorted a dosn`t contain sorted b!";
 
    merge(a,a+10,b,b+5,d);
 
    for(int j=0;j<15;j++)
 
       cout<<" "<<d[j]; 

    return 0;
 
}
```

**sort()的原型是：** 

template <class RandomAccessIterator>
 
void sort(RandomAccessIterator first, RandomAccessIterator last);
 
功能是对[first,last-1]区间内的元素进行排序操作。与之类似的操作还有：partial_sort(), stable_sort()，partial_sort_copy()等等。 

**merge()的原型是：**

template <class InputIterator1, class InputIterator2, class OutputIterator>
 
OutputIterator merge(InputIterator1 first1, InputIterator1 last1,InputIterator2  first2, InputIterator2 st2,OutputIterator result);
 
将有序区间[first1,last1-1]和[first2,last2-1]合并到[result, result + (last1 - first1) + (last2 - first2)-1]区间内。
 
**Includes()的原型是： **

template <class InputIterator1, class InputIterator2>
 
bool includes(InputIterator1 first1, InputIterator1 last1, InputIterator2 first2, InputIterator2 last2);
 
其功能是检查有序区间[first2,last2-1]内元素是否都在[first1,last1-1]区间内，返回一个bool值。
 

以上我们学习了STL容器和算法的概念，以及一些简单的STL容器和算法。在使用算法处理容器内的数据时，需要从一个数据成员移向另一个数据成员，迭代器恰好实现了这一功能。下面我们来学习STL迭代器 。


**堆的用法（heap）**                        
STL里面的堆操作一般用到的只有4个。    
他们就是 

make_heap();、pop_heap();、push_heap();、sort_heap();     
他们的头函数是algorithm 

**首先是make_heap();**                  
他的函数原型是：             
void make_heap(first_pointer,end_pointer,compare_function);           
一个参数是数组或向量的头指针，第二个向量是尾指针。第三个参数是比较函数的名字
。在缺省的时候，默认是大跟堆。（下面的参数都一样就不解释了）
作用：把这一段的数组或向量做成一个堆的结构。范围是(first,last)

**然后是pop_heap();**          
它的函数原型是：                                               
void pop_heap(first_pointer,end_pointer,compare_function);                   
作用：pop_heap()不是真的把最大（最小）的元素从堆中弹出来。而是重新排序堆。它把first和last交换，然后将[first,last-1)的数据再做成一个堆。       

**接着是push_heap()**                                         
void pushheap(first_pointer,end_pointer,compare_function);                            
作用：push_heap()假设由[first,last-1)是一个有效的堆，然后，再把堆中的新元素加     
进来，做成一个堆。

**最后是sort_heap()**                                            
void sort_heap(first_pointer,end_pointer,compare_function);                         
作用是sort_heap对[first,last)中的序列进行排序。它假设这个序列是有效堆。（当然
，经过排序之后就不是一个有效堆了）                 

下面是例程：

```
#include<algorithm>
#include<cstdio>
using namespace std;
bool cmp(int a,int b)
{
    return a>b;
}
int main()
{
    int i,number[20]={29,23,20,22,17,15,26,51,19,12,35,40};
    make_heap(&number[0],&number[12]);
    //结果是:51 35 40 23 29 20 26 22 19 12 17 15
    for(i=0;i<12;i++)
        printf("%d ",number[i]);
    printf("\n");
    make_heap(&number[0],&number[12],cmp);
    //结果：12 17 15 19 23 20 26 51 22 29 35 40
    for(i=0;i<12;i++)
        printf("%d ",number[i]);
    printf("\n");
    //加入元素8
    number[12]=8;
    //加入后调整
    push_heap(&number[0],&number[13],cmp);
    //结果：8 17 12 19 23 15 26 51 22 35 40 20
    for(i=0;i<13;i++)
        printf("%d ",number[i]);
    printf("\n");
    //弹出元素8
    pop_heap(&number[0],&number[13],cmp);
    //结果：12 17 15 19 23 20 26 51 22 29 35 40
    for(i=0;i<13;i++)
        printf("%d ",number[i]);
    printf("\n");
    sort_heap(&number[0],&number[12],cmp);
    //结果不用说都知道是有序的了！
    for(i=0;i<12;i++)
        printf("%d ",number[i]);
    return 0;
}
```


### 迭代器（itertor）
 
#include<iterator>
 
迭代器实际上是一种泛化指针，如果一个迭代器指向了容器中的某一成员，那么迭代器将可以通过自增自减来遍历容器中的所有成员。迭代器是联系容器和算法的媒介，是算法操作容器的接口。在运用算法操作容器的时候，我们常常在不知不觉中已经使用了迭代器。           
STL中定义了6种迭代器：
 
输入迭代器，在容器的连续区间内向前移动，可以读取容器内任意值； 

输出迭代器，把值写进它所指向的队列成员中； 

前向迭代器，读取队列中的值，并可以向前移动到下一位置（++p,p++）； 

双向迭代器，读取队列中的值，并可以向前向后遍历容器； 

随机访问迭代器, vector<T>::iterator，list<T>::iterator等都是这种迭代器； 

流迭代器，可以直接输出、输入流中的值； 

实际上，在前面的例子中，我们不停的在用迭代器。下面我们用几个例子来帮助理解这些迭代器的用法。
下面的例子用到了输入输出迭代器： 

```
#include <iostream>
 
#include <fstream>
 
#include <iterator>
 
#include <vector>
 
#include <string>
 
using namespace std;
 
 
 
int main()
 
{
 
    vector<string> v1;
 
    ifstream file("Text1.txt");
 
    if(file.fail())
 
    {
 
        cout<<"open file Text1.txt failed"<<endl;
 
        return 1;
 
    }    

    copy(istream_iterator<string>(file),istream_iterator<string>(),inserter(v1,v1.begin()));
 
    copy(v1.begin(),v1.end(),ostream_iterator<string>(cout," "));
 
    cout<<endl;
 
    return 0;
 
}
```

这里用到了输入迭代器istream_iterator，输出迭代器ostream_iterator。程序完成了将一个文件输出到屏幕的功能，先将文件读入，然后通过输入迭代器把文件内容复制到类型为字符串的向量容器内，最后由输出迭代器输出。Inserter是一个输入迭代器的一个函数(迭代器适配器)，它的使用方法是：
 
inserter (container ,pos);
 
container是将要用来存入数据的容器，pos是容器存入数据的开始位置。上例中，是把文件内容存入（copy()）到向量v1中。

**References** 

[STL in ACM - To be an ACMan - 博客园](http://www.cnblogs.com/ACMan/archive/2012/05/30/2526927.html)