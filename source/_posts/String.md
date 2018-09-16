---
title: String
date: 2018-07-17 15:50:45
tags: [js原生, String]
---
## 概述
`String`对象是 `JavaScript` 原生提供的三个包装对象之一，用来生成字符串对象。
字符串对象是一个类似数组的对象（很像数组，但不是数组）。
```JavaScript
  var s1 = 'abc';
  var s2 = new String('abc');

  typeof s1 // "string"
  typeof s2 // "object"

  s2.valueOf() // "abc"
```
<!--more-->
## 静态方法
### String.fromCharCode()
String对象提供的静态方法（即定义在对象本身，而不是定义在对象实例的方法），主要是String.fromCharCode()。该方法的参数是一个或多个数值，代表 Unicode 码点，返回值是这些码点组成的字符串。
```JavaScript
  String.fromCharCode() // ""
  String.fromCharCode(97) // "a"
  String.fromCharCode(104, 101, 108, 108, 111)
  // "hello"
```

## 实例属性
### String.prototype.charAt()
`charAt`方法返回指定位置的字符，参数是从`0`开始编号的位置。
如果参数为负数，或大于等于字符串的长度，返回空字符串。
```JavaScript
  var s = new String('abc');

  s.charAt(1) // "b"
  s.charAt(s.length - 1) // "c"

  'abc'.charAt(1) // "b"
  'abc'[1] // "b"
```
### String.prototype.charCodeAt()
`charCodeAt`方法返回字符串指定位置的 `Unicode 码点`（十进制表示），相当于`String.fromCharCode()`的逆操作。
```JavaScript
  'abc'.charCodeAt(1) // 98
```

### String.prototype.concat()
`concat`方法用于连接两个或多个字符串，返回一个新字符串，不改变原字符串。
```JavaScript
  var s1 = 'abc';
  var s2 = 'def';

  s1.concat(s2) // "abcdef"
  s1 // "abc"

  'a'.concat('b', 'c') // "abc"
```
### String.prototype.slice()
方法同数组的方法用法一致
### String.prototype.substring()
`substring`方法用于从原字符串取出子字符串并返回，不改变原字符串，跟`slice`方法很相像。它的第一个参数表示子字符串的开始位置，第二个位置表示结束位置（返回结果不含该位置）。
```JavaScript
  'JavaScript'.substring(0, 4) // "Java"
```
如果省略第二个参数，则表示子字符串一直到原字符串的结束。
```JavaScript
  'JavaScript'.substring(4) // "Script"
```
如果第一个参数大于第二个参数，substring方法会自动更换两个参数的位置。
```JavaScript
  JavaScript'.substring(10, 4) // "Script"
  // 等同于
  'JavaScript'.substring(4, 10) // "Script"
```
如果参数是负数，substring方法会自动将负数转为0。
```JavaScript
  'Javascript'.substring(-3) // "JavaScript"
  'JavaScript'.substring(4, -3) // "Java"
```

### String.prototype.substr()
`substr`方法的第一个参数是子字符串的开始位置（从0开始计算），第二个参数是子字符串的长度。
如果省略第二个参数，则表示子字符串一直到原字符串的结束。
如果第一个参数是负数，表示倒数计算的字符位置。如果第二个参数是负数，将被自动转为0，因此会返回空字符串。
```JavaScript
  'JavaScript'.substr(4, 6) // "Script"
```

### String.prototype.indexOf()，String.prototype.lastIndexOf() 
`indexOf`方法用于确定一个字符串在另一个字符串中第一次出现的位置，返回结果是匹配开始的位置。如果返回`-1`，就表示不匹配。
`indexOf`方法还可以接受第二个参数，表示从该位置开始向后匹配。
`lastIndexOf`方法的用法跟`indexOf`方法一致，主要的区别是`lastIndexOf`从尾部开始匹配，`indexOf`则是从头部开始匹配。

### String.prototype.trim()
`trim`方法用于去除字符串两端的空格，返回一个新字符串，不改变原字符串。
该方法去除的不仅是空格，还包括 **制表符**（\t、\v）、**换行符**（\n）和 **回车符**（\r）。
```JavaScript
  '  hello world  '.trim()
  // "hello world"
```
### String.prototype.toLowerCase()，String.prototype.toUpperCase()
`toLowerCase`方法用于将一个字符串全部转为小写，`toUpperCase`则是全部转为大写。它们都返回一个新字符串，不改变原字符串。
```Javascript
  'Hello World'.toLowerCase()
  // "hello world"

  'Hello World'.toUpperCase()
  // "HELLO WORLD"
```
### String.prototype.match()
`match`方法用于确定原字符串是否匹配某个子字符串，返回一个数组，成员为匹配的第一个字符串。如果没有找到匹配，则返回`null`。
```Javascript
  'cat, bat, sat, fat'.match('at') // ["at"]
  'cat, bat, sat, fat'.match('xt') // null
```
返回的数组还有index属性和input属性，分别表示匹配字符串开始的位置和原始字符串。
```Javascript
  var matches = 'cat, bat, sat, fat'.match('at');
  matches.index // 1
  matches.input // "cat, bat, sat, fat"
```
`match`方法还可以使用正则表达式作为参数
### String.prototype.search()，String.prototype.replace()
`search`方法的用法基本等同于`match`，但是返回值为匹配的第一个位置。如果没有找到匹配，则返回`-1`
`replace`方法用于替换匹配的子字符串，一般情况下只替换第一个匹配（除非使用带有`g`修饰符的正则表达式）
```JavaScript
  'cat, bat, sat, fat'.search('at') // 1
  'aaa'.replace('a', 'b') // "baa"
```
### String.prototype.split()
`split`方法按照给定规则分割字符串，返回一个由分割出来的子字符串组成的数组。
```JavaScript
  'a|b|c'.split('|') // ["a", "b", "c"]

  'a|b|c'.split('') // ["a", "|", "b", "|", "c"]
```
split方法还可以接受第二个参数，限定返回数组的最大成员数。