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


**7.2 Strings**


In MATLAB: char(x)输出ASCII里第x位的符号（e.g. function char_codes）
Strings和数集一样可以用length

```
>> MOOC_title = 'MATLAB for Smarties’;
>> length(MOOC_title)
ans =
19
>> MOOC_title(1)
ans =
M

String functions

findstr:
>> MOOC_title = 'MATLAB for Smarties';
>> strfind(MOOC_title,'LAB')
ans =
4
输出的是第一个字母的位置
case sensitive
>> strfind(MOOC_title,'r')
ans =
10 15
多次出现的话会输出多个值
strcmp:
>> MOOC_title = 'MATLAB for Smarties';
>> lang = ‘MATLAB';
>> strcmp(MOOC_title, lang)
ans =
0
Meaning two strings are not the same.
case sensitive
>> strcmp(MOOC_title(1:6), lang)
ans =
1
strcmpi: to compare while ignore case
>> strcmpi(MOOC_title(1:6), 'Matlab')
ans =
1
str2num (num2str):
>> pi_number = 3.1416
pi_number =
3.1416
>> pi_digits = '3.1416'
pi_digits =
3.1416
>> str2num(pi_digits)
ans =
3.1416
sprintf:
与fprintf一样，除了有一个输出值可以赋予变量（变量类型为string/char)
```


**7.3 Structs**

Structs      
An array must be homogeneous:
It cannot contain elements of multiple types.
A struct can be heterogeneous:
It can contain multiple types.

A struct is different from an array:
fields, not elements
field names, not indices
Fields in the same struct can have different types.
Versatility inside:
A field of a struct can contain another struct.
Structs can hold arrays, and arrays can hold structs.
Struct functions

```
Create a struct
>> account.number = 1234567
account =
number: 1234567
>> account.balance = 5000
account =
number: 1234567
balance: 5000
>> account.owner.name = 'Joe Smith'
account =
number: 1234567
balance: 5000
owner: [1x1 struct]
>> account.owner.email = 'joe@matlabcooc.com'
account =
number: 1234567
balance: 5000
owner: [1x1 struct]
Create another struct with the same name — 和原先的有一样的field，原先的自动变成（1）；只限于一级子类别，对account.owner.后面的类别没影响
>> account(2).number = 7654321
account =
1x2 struct array with fields:
number
balance
owner
isfield: 输出真或假（有或无）
>> account(1:2).owner
ans =
name: 'Joe Smith'
email: 'joe@matlabcooc.com'
ans =
name: 'Jane Farmer'
age: 23
>> isfield(account(2).owner,'age')
ans =
1
>> isfield(account(1).owner,'age')
ans =
0
rmfield: remove field
>> account(1).owner = rmfield(account(2).owner,'age’);
>> account(1).owner
ans =
name: 'Jane Farmer'
struct:
>> course = struct('Area', 'CS', 'number', 103, 'title', 'Introductory Programming for Engineers and Scientists')
course =
Area: 'CS'
number: 103
title: 'Introductory Programming for Engineers and Scientists'
```

