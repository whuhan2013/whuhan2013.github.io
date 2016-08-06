---
layout: post
title: python基础语法
date: 2016-8-6
categories: blog
tags: [python]
description: python基础语法.
---


python基础语法，主要基于2.7，对于3.0的改动有部分指出    


```
# 单行的注释是通过#号开始。

""" 多行的注释
     可以使用"""号包括
"""


####################################################
## 1. 基础的数据类型和操作符
####################################################

# 数字类型
3  # => 3

# 数学表达式
1 + 1  # => 2
8 - 1  # => 7
10 * 2  # => 20
35 / 5  # => 7

# 除法有一点棘手。它自动的是整除。
5 / 2  # => 2

# 要解决这个问题我们需要学习浮点型
2.0     # 这是一个浮点型
11.0 / 4.0  # => 2.75 这个结果好了很多

# 整数除法得到的结果有好的方面也有不好的方面。
5 // 3     # => 1
5.0 // 3.0 # => 1.0 
-5 // 3  # => -2
-5.0 // 3.0 # => -2.0

# 注意我们可以使用导入除法模块（在第六章模块中讲解）
# 用来计算普通的除法仅仅使用一个'/'。
from __future__ import division
11/4    # => 2.75  
11//4   # => 2 

# 取余操作符
7 % 3 # => 1

# 求幂操作
2**4 # => 16

# 括号优先
(1 + 3) * 2  # => 8

# 布尔操作符
# 注意 「and」和「or」操作符是区分大小写的
True and False #=> False
False or True #=> True

# 注意下通过布尔操作符比较整数
0 and 2 #=> 0
-5 or 0 #=> -5
0 == False #=> True
2 == True #=> False
1 == True #=> True

# 通过「not」取反
not True  # => False
not False  # => True

# 「==」表示相等
1 == 1  # => True
2 == 1  # => False

#「!=」 表示不等
1 != 1  # => False
2 != 1  # => True

# 更多的比较操作符
1 < 10  # => True
1 > 10  # => False
2 <= 2  # => True
2 >= 2  # => True

# 比较可以是连续的！
1 < 2 < 3  # => True
2 < 3 < 2  # => False

# 字符串可以通过「"」或者「'」创建
"This is a string."
'This is also a string.'

# 字符串也可以相加！
"Hello " + "world!"  # => "Hello world!"
# 字符串可以不通过「+」来拼接
"Hello " "world!"  # => "Hello world!"

# 还可以重复多次
"Hello" * 3  # => "HelloHelloHello"

# 一串字符串可以被当做一个字符的列表
"This is a string"[0]  # => 'T'

# 字符串通过「%」来格式化
# 即使「%」操作符在Python 3.1下是不建议使用的并且在以后的某个时候会
# 被移除掉，了解它怎么使用的还是没有坏处的。
x = 'apple'
y = 'lemon'
z = "The items in the basket are %s and %s" % (x,y)

# 一个新的方法来格式化字符串
# 这个方法是首选的方法
"{} is a {}".format("This", "placeholder")
"{0} can be {1}".format("strings", "formatted")
# 若你不想去计数，你可以使用关键词
"{name} wants to eat {food}".format(name="Bob", food="lasagna")

# None是一个对象
None  # => None

# 不要使用「==」来比较None和其他对象
# 使用「is」代替
"etc" is None  # => False
None is None  # => True

# 「is」操作符用来判断对象的合法性。
# 他对于处理私有的值不是很有用，
# 但是处理对象则不然。

# 任何一个对象都可以使用一个布尔型的内容。
# 下面的这些值是认为假：
#    - None
#    - 所有数值型类型的空 (e.g., 0, 0L, 0.0, 0j)
#    - 空的序列 (e.g., '', (), [])
#    - 空的容器 (e.g., {}, set())
#    - 符合某些条件的用户自定义的类的实例
# 详情见：https://docs.python.org/2/reference/datamodel.html#object.__nonzero__
# 所有其他的值都为真（使用bool()方法返回为True）
bool(0)  # => False
bool("")  # => False


####################################################
## 2. 变量和集合
####################################################

# Python 有一个print方法
print "I'm Python. Nice to meet you!" # => I'm Python. Nice to meet you!

# 从命令行简单的获取输入数据
input_string_var = raw_input("Enter some data: ") # 返回一个String型的值
input_var = input("Enter some data: ") # 返回一个Int型的值
# 警告：使用input()方法必须要谨慎
# 注意：在Python 3，input()不建议使用，并且raw_input()改名为input()

# 在赋值变量之前不用分配它们
some_var = 5    # 约定使用带下划线的小写
some_var  # => 5

# 访问之前没有申明过的变量会发生异常。
# 前往「流程控制」学习如何处理异常。
some_other_var  # 提出一个变量名错误

# 「if」可以被用作类似C的三元操作符「?:」
"yahoo!" if 3 > 2 else 2  # => "yahoo!"

# Lists存储
li = []
# 你可以使用预先有数据的list
other_li = [4, 5, 6]

# 通过append方法往list后面添加数据
li.append(1)    # li is now [1]
li.append(2)    # li is now [1, 2]
li.append(4)    # li is now [1, 2, 4]
li.append(3)    # li is now [1, 2, 4, 3]
# 通过pop从list最后面移除数据
li.pop()        # => 3 and li is now [1, 2, 4]
# 我们再将其加入进来
li.append(3)    # li is now [1, 2, 4, 3] again.

# 访问list就像使用数组一样
li[0]  # => 1
# 通过「=」给已经存在的索引添加一个新的值
li[0] = 42
li[0]  # => 42
li[0] = 1  # Note: setting it back to the original value
# Look at the last element
# 查看最后一个元素
li[-1]  # => 3

# 当查询越界的话会报IndexError
li[4]  # Raises an IndexError

# 你可以通过排列拆分获取list中的内容
li[1:3]  # => [2, 4]
# 省略结尾
li[2:]  # => [4, 3]
# 省略开头
li[:3]  # => [1, 2, 4]
# 隔两个步长访问
li[::2]   # =>[1, 4]
# 反转list
li[::-1]   # => [3, 4, 2, 1]
# 使用li[开始：结束：步长]来实现不同的切分

# 通过「del」删除指定位置的元素
del li[2]   # li is now [1, 2, 3]

# 你可以添加其他list
li + other_li   # => [1, 2, 3, 4, 5, 6]
# 注意：li和other_list中的数据是没有被修改的

# 通过「extend()」来连接list
li.extend(other_li)   # Now li is [1, 2, 3, 4, 5, 6]

# 删除第一个找到对应值的元素
li.remove(2)  # li is now [1, 3, 4, 5, 6]
li.remove(2)  # 得到一个ValueError，因为2已经不在li中了

# 在特定位置插入一个元素
li.insert(1, 2)  # li is now [1, 2, 3, 4, 5, 6] again

# 找到对应元素的位置
li.index(2)  # => 1
li.index(7)  # 得到一个ValueError，因为7不在li中

# 通过「in」查看list中是否存在改元素
1 in li   # => True

# 通过「len()」获取list长度
len(li)   # => 6


# 元祖「Tuples」比较像list，但是它是不可变的
tup = (1, 2, 3)
tup[0]   # => 1
tup[0] = 3  # Raises a TypeError

# 你可以在Tuples中做下面所有list的操作
len(tup)   # => 3
tup + (4, 5, 6)   # => (1, 2, 3, 4, 5, 6)
tup[:2]   # => (1, 2)
2 in tup   # => True

# 你可以将tuples（或者lists）中的数据取到变量中
a, b, c = (1, 2, 3)     # a is now 1, b is now 2 and c is now 3
d, e, f = 4, 5, 6       # you can leave out the parentheses
# Tuples可以默认创建
g = 4, 5, 6             # => (4, 5, 6)
# 简单的交换两个变量的值
e, d = d, e     # d is now 5 and e is now 4


# 字典存储map
empty_dict = {}
# 这是一个预先定义的字典
filled_dict = {"one": 1, "two": 2, "three": 3}

# 通过「[]」查看字典中的值
filled_dict["one"]   # => 1

# 通过「keys()」获得一个所有键的list
filled_dict.keys()   # => ["three", "two", "one"]
# 注意：字典中键的排序是不规律的
# 你的结果可能和这个不完全匹配

# Get all values as a list with "values()"
# 通过「values()」获得一个所有值的list
filled_dict.values()   # => [3, 2, 1]
# 注意：值的排序同上面键的排序

# 通过「in」查看键是在字典中存在
"one" in filled_dict   # => True
1 in filled_dict   # => False

# 试图访问一个不存在的键时会有KeyError
filled_dict["four"]   # KeyError

# 通过「get()」方法来避免产生KeyError
filled_dict.get("one")   # => 1
filled_dict.get("four")   # => None
# 这个get方法在查找的值不存在时，可以有一个默认值作为参数
filled_dict.get("one", 4)   # => 1
filled_dict.get("four", 4)   # => 4
# 记住filled_dict.get("four") 仍然会返回None
# （get方法不会在字典中设置值）

# 可以通过和list一样的方法设置键值
filled_dict["four"] = 4  # now, filled_dict["four"] => 4

# 「setdefault()」方法只有在字典中不存在的时候才插入
filled_dict.setdefault("five", 5)  # filled_dict["five"] is set to 5
filled_dict.setdefault("five", 6)  # filled_dict["five"] is still 5


empty_set = set()
# 初始化set
some_set = set([1, 2, 2, 3, 4])   # some_set is now set([1, 2, 3, 4])

# 排序也是没有规律的，及时看起来似乎像排好序的
another_set = set([4, 3, 2, 2, 1])  # another_set is now set([1, 2, 3, 4])

# 从Python 2.7开始，{}可以用来声明一个set
filled_set = {1, 2, 2, 3, 4}   # => {1, 2, 3, 4}

# 往一个set中加入更多的元素
filled_set.add(5)   # filled_set is now {1, 2, 3, 4, 5}

# set集合通过「&」做交集操作
other_set = {3, 4, 5, 6}
filled_set & other_set   # => {3, 4, 5}

# set集合通过「|」做并集操作
filled_set | other_set   # => {1, 2, 3, 4, 5, 6}

# set集合通过「-」做不包含操作
{1, 2, 3, 4} - {2, 3, 5}   # => {1, 4}

# set集合通过「^」做去重操作
{1, 2, 3, 4} ^ {2, 3, 5}  # => {1, 4, 5}

# 检查右边的集合是否是左边的子集
{1, 2} >= {1, 2, 3} # => False

# 检查左边的集合是否是右边的子集
{1, 2} <= {1, 2, 3} # => True

# 通过「in」检查特定元素是否存在在集合中
2 in filled_set   # => True
10 in filled_set   # => False


####################################################
## 3. 流程控制
####################################################

# 让我们来新建一个变量
some_var = 5

# 下面是一些if语句。缩减在Python中是必须的！
if some_var > 10:
    print "some_var is totally bigger than 10."
elif some_var < 10:    # 这个elif代码块是可选的
    print "some_var is smaller than 10."
else:           # 这个同样是可选的
    print "some_var is indeed 10."


"""
通过循环迭代打印列表
prints:
    dog is a mammal
    cat is a mammal
    mouse is a mammal
"""
for animal in ["dog", "cat", "mouse"]:
    # 你可以通过{0}占位符来格式化字符串（详情见上）
    print "{0} is a mammal".format(animal)

"""
「range(number)」返回一个从零到传参数字的列表
prints:
    0
    1
    2
    3
"""
for i in range(4):
    print i

"""
「range(number)」返回一个从第一个参数到第二个参数的列表
prints:
    4
    5
    6
    7
"""
for i in range(4, 8):
    print i

"""
While可以一直循环到条件不成立
prints:
    0
    1
    2
    3
"""
x = 0
while x < 4:
    print x
    x += 1  # Shorthand for x = x + 1

# 通过try/except代码段来处理异常

# 在Python2.6以上可以使用：
try:
    # 通过「raise」抛出一个异常
    raise IndexError("This is an index error")
except IndexError as e:
    pass    # Pass就是一个空语句。通常你需要在这里恢复
except (TypeError, NameError):
    pass    # 如果有必要，多个异常都可以在这里被处理
else:   # 对于try/except代码段可选的条件。必须要跟在所有的except语句之后
    print "All good!"   # 仅仅在代码中try没有异常的时候执行
finally: #  在所有语句执行完毕之后执行
    print "We can clean up resources here"

# 你可以使用一个with语句，代替try/finally语句去清空资源
with open("myfile.txt") as f:
    for line in f:
        print line


####################################################
## 4. 函数
####################################################

# 使用「def」来创造一个新的函数
def add(x, y):
    print "x is {0} and y is {1}".format(x, y)
    return x + y    # 通过return语句返回结果

# 调用代参的函数
add(5, 6)   # => prints out "x is 5 and y is 6" and returns 11

# 通过关键字参数，是另一种调用的方式
add(y=6, x=5)   # Keyword arguments can arrive in any order.

# 你可以定义一个接收一个变量位置的参数，通过使用「*」来插入元祖的数据
def varargs(*args):
    return args

varargs(1, 2, 3)   # => (1, 2, 3)

# 你可以定义一个接收一个变量位置的参数，通过使用「**」来插入字典的数据
def keyword_args(**kwargs):
    return kwargs

# 让我们看看调用它会发生什么
keyword_args(big="foot", loch="ness")   # => {"big": "foot", "loch": "ness"}

# 当然若你喜欢，你可以同时使用它俩
def all_the_args(*args, **kwargs):
    print args
    print kwargs
"""
all_the_args(1, 2, a=3, b=4) prints:
    (1, 2)
    {"a": 3, "b": 4}
"""

# 当你调用函数的时候，你可以选择参数，通过「*」和「**」传递不同的参数类型
args = (1, 2, 3, 4)
kwargs = {"a": 3, "b": 4}
all_the_args(*args)   # equivalent to foo(1, 2, 3, 4)
all_the_args(**kwargs)   # equivalent to foo(a=3, b=4)
all_the_args(*args, **kwargs)   # equivalent to foo(1, 2, 3, 4, a=3, b=4)

def pass_all_the_args(*args, **kwargs):
    all_the_args(*args, **kwargs)
    print varargs(*args)
    print keyword_args(**kwargs)

# 函数的作用范围
x = 5

def set_x(num):
    # 函数当前的变量x和全局的变量x是不同的
    x = num # => 43
    print x # => 43

def set_global_x(num):
    global x
    print x # => 5
    x = num # 全局的变量x现在变成了6
    print x # => 6

set_x(43)
set_global_x(6)

# Python有第一类函数
def create_adder(x):
    def adder(y):
        return x + y
    return adder

add_10 = create_adder(10)
add_10(3)   # => 13

# 当然也有匿名函数
(lambda x: x > 2)(3)   # => True
(lambda x, y: x ** 2 + y ** 2)(2, 1) # => 5

# 还有很多内建的高阶函数
map(add_10, [1, 2, 3])   # => [11, 12, 13]
map(max, [1, 2, 3], [4, 2, 1])   # => [4, 2, 3]

filter(lambda x: x > 5, [3, 4, 5, 6, 7])   # => [6, 7]

# 我们可以使用list解析出漂亮的map和过滤器
[add_10(i) for i in [1, 2, 3]]  # => [11, 12, 13]
[x for x in [3, 4, 5, 6, 7] if x > 5]   # => [6, 7]


####################################################
## 5. 类
####################################################

# 我们从object中获得一个子类
class Human(object):

    # 一个类的属性。它被所有的这个类的实例共享
    species = "H. sapiens"

    # 基础的初始化，当类被实例化的时候调用。
    # 注意前后的双下划线表示对象或者属性由Python使用，但是它处在用户控制
    #的命名空间当中。你不应该这样创建自己的名字。
    def __init__(self, name):
        # 申明变量是一个实例的名字属性
        self.name = name

        # 初始化属性
        self.age = 0

    # 一个实例的方法。所有的方法都将「self」作为第一个参数
    def say(self, msg):
        return "{0}: {1}".format(self.name, msg)

    # 一个类中的方法是对所有的实例对象所共享的
    # 它们通过第一个参数所属的类被其类所调用
    @classmethod
    def get_species(cls):
        return cls.species

    # 一个静态的方法不通过一个类或者一个实例所调用
    @staticmethod
    def grunt():
        return "*grunt*"

    # 「property」就像是getter方法
    # 它将返回方法age()的只读的属性
    @property
    def age(self):
        return self._age

    # 下面的方法允许属性被set
    @age.setter
    def age(self, age):
        self._age = age

    # 下面的方法允许属性被删除
    @age.deleter
    def age(self):
        del self._age

# 实例化一个类
i = Human(name="Ian")
print i.say("hi")     # prints out "Ian: hi"

j = Human("Joel")
print j.say("hello")  # prints out "Joel: hello"

# 调用类中的方法
i.get_species()   # => "H. sapiens"

# 修改共享的属性
Human.species = "H. neanderthalensis"
i.get_species()   # => "H. neanderthalensis"
j.get_species()   # => "H. neanderthalensis"

# 调用静态方法
Human.grunt()   # => "*grunt*"

# 更新属性
i.age = 42

# 得到属性
i.age # => 42

# 删除属性
del i.age
i.age  # => raises an AttributeError


####################################################
## 6. 模块
####################################################

# 你可以导入模块
import math
print math.sqrt(16)  # => 4

# 你可以从一个模块中得到其特殊的方法
from math import ceil, floor
print ceil(3.7)  # => 4.0
print floor(3.7)   # => 3.0

# 你可以导入从一个模块中导入所有的函数
# 警告：这种方法是不建议的
from math import *

# 你可以将模块的名字缩写
import math as m
math.sqrt(16) == m.sqrt(16)   # => True
# you can also test that the functions are equivalent
from math import sqrt
math.sqrt == m.sqrt == sqrt  # => True

# Python的模块就是普通的python文件。你可以自己编写你自己的然后导入它们。
# 这些模块的名称和你的文件名称相同。

# 你可以通过「dir」找到这个模块中的方法和属性
import math
dir(math)

# 若你有一个和Python同名的叫「math.py」的脚本在当前的文件夹下，你所编写
# 的文件将会在载入时替换掉Python内建的模块。这种情况是因为你本地文件的权
# 限高于Python内建的库。


####################################################
## 7. 高级
####################################################

# Generators help you make lazy code
def double_numbers(iterable):
    for i in iterable:
        yield i + i

# A generator creates values on the fly.
# Instead of generating and returning all values at once it creates one in each
# iteration.  This means values bigger than 15 wont be processed in
# double_numbers.
# Note xrange is a generator that does the same thing range does.
# Creating a list 1-900000000 would take lot of time and space to be made.
# xrange creates an xrange generator object instead of creating the entire list
# like range does.
# We use a trailing underscore in variable names when we want to use a name that
# would normally collide with a python keyword
xrange_ = xrange(1, 900000000)

# will double all numbers until a result >=30 found
for i in double_numbers(xrange_):
    print i
    if i >= 30:
        break


# Decorators
# in this example beg wraps say
# Beg will call say. If say_please is True then it will change the returned
# message
from functools import wraps

def beg(target_function):
    @wraps(target_function)
    def wrapper(*args, **kwargs):
        msg, say_please = target_function(*args, **kwargs)
        if say_please:
            return "{} {}".format(msg, "Please! I am poor :(")
        return msg

    return wrapper

@beg
def say(say_please=False):
    msg = "Can you buy me a beer?"
    return msg, say_please

print say()  # Can you buy me a beer?
print say(say_please=True)  # Can you buy me a beer? Please! I am poor :(
```

