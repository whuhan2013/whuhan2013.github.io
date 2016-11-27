---
layout: post
title: Matlab基础语法(三)
date: 2016-11-27
categories: blog
tags: [matlab]
description: Matlab基础语法
---


**本文主要包括以下内容**    

- 数据类型 
- 文件输入输出   
    

#### 数据类型     

**7.1 Introduction to Data Types**


The Limitation of Computers
Real numbers in mathematics:
Can be infinitely large
Have infinitely fine resolution
Computers: Finite memory
Upper limit on the largest number that can be represented
Lower limit on the absolute value of any non-zero number
The set of values that can be represented by a MATLAB variable is finite.

```
Data Types
MATLAB: many different data types
A data type is defined by:
Set of values
Set of operations that can be performed on those values
MATLAB:
All elements of a given array must be of the same type
Elementary type
Type of a MATLAB array is defined by
Number of dimensions
Size in each dimension
Elementary type
to check data type: class(x)
x = 23; class(x) = double — default data type for all numerical data in matlab (incl. complex numbers)
whos: check type & size of every variables
class(double) = char — character
class(3<4) = logical
Numerical Types
double
Default type in MATLAB
Floating point representation
Example: 12.34 = 1234 * 10-2
Mantissa and exponent
64 bits (8 bytes)
single
32-bit floating point
Integer types
Signed (both negative & non-negative numbers), unsigned (only non-negative numbers)
8-, 16-, 32-, 64-bit long
Range of Values
Inf: “Infinity” — x/0
NaN: “Not-a-Number” — 0/0
Useful functions
Type check:
class
isa
>> isa(x,’double’)
Range check:
intmax, intmin
realmax, realmin
>> intmax(’uint32’)
Conversion:
Name of function = name of desired data type
int8(x), uint32(x), double(x), etc.
Operators
Arithmetic operators
Operands of the same data type: No problem
Different data types:
“mixed-mode arithmetic”
Many rules and restrictions
Relational operators
Different data types are always allowed
Result is always of logical type
Conversion In MATLAB
>> k = uint8(500)
k =
255
>> k = uint8(256)
k =
255
>> k = uint8(-1)
k =
0
若赋值超出变量范围，则会自动变为最接近变量范围的值(clipping)
Range Check in MATLAB
>> intmax('uint8')
ans =
255
>> intmin('uint8')
ans =
0
>> intmin('int32')
ans =
-2147483648
>> realmax('double')
ans =
1.7977e+308
>> realmin('single')
ans =
1.1755e-38
```