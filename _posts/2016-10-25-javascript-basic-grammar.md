---
layout: post
title: Javascript基础语法
date: 2016-10-25
categories: blog
tags: [javascript]
description: javascript
---

**javascript数组从头部与尾部插入与删除**        
push pop shift unshift       

**javascript对象添加与删除属性**       
添加方法与修改一致，直接加上去即可            
删除用delete ourDog.bark;        

**javascript是否拥有某个属性**         
hasOwnProperty


**javascript随机数**              

```
Math.floor(Math.random() * (ourMax - ourMin + 1)) + ourMin;
```

**正则表达式**     

```
/ is the start of the regular expression.

the is the pattern we want to match.

/ is the end of the regular expression.

g means global, which causes the pattern to return all matches in the string, not just the first one.

i means that we want to ignore the case (uppercase or lowercase) when searching for the pattern.

```

注意：\d表示数字，＼D表示相反


**javascript数组变换**           
map reduce filter sort concat       


```
var oldArray = [1, 2, 3];
var timesFour = oldArray.map(function(val){
  return val * 4;
});
console.log(timesFour); // returns [4, 8, 12]
console.log(oldArray);  // returns [1, 2, 3]


var singleVal = array.reduce(function(previousVal, currentVal) {
  return previousVal - currentVal;
}, 0);


var newArray = oldArray.filter(function(val){
  if(val<6){
    return val;
  }
});

sort can be passed a compare function as a callback. The compare function should return a negative number if a should be before b, a positive number if a should be after b, or 0 if they are equal.

var array = [1, 12, 21, 2];
array.sort(function(a, b) {
  return a - b;
});


var myArray = [1, 2, 3];
myArray.reverse();
```

同时还有string的split与join


**javascript逆转字符串**       

```
function palindrome(str) {
  // Good luck!
  var re = /[^a-zA-Z0-9]/g;
  var t1=str.replace(re,"");
  var t2=t1.toLowerCase();
  var temStr=t2.split("").reverse().join("");
  if(temStr==t2){
  return true;
  }else{
  return false;
}
  
}
palindrome("eye");
```















