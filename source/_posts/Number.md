---
title: Number
date: 2018-07-17 15:30:11
tags: [js原生,Number]
categories: [js原生,Number]
---

## `Number`
对象是数值对应的包装对象，可以作为构造函数使用，也可以作为工具函数使用。
作为构造函数时，它用于生成值为数值的对象。
```javascrpt
  var n = new Number(1);
  typeof n // "object"
```
<!--more-->
作为工具函数时，它可以将任何类型的值转为数值。
```javascrpt
  Number(true) // 1
```
## 实例方法
### Number.prototype.toString()
Number对象部署了自己的toString方法，用来将一个数值转为字符串形式。
```javascrpt
  (10).toString() // "10"
```
`toString`方法可以接受一个参数，表示输出的进制。如果省略这个参数，默认将数值先转为十进制，再输出字符串；否则，就根据参数指定的进制，将一个数字转化成某个进制的字符串。
```javascrpt
  (10).toString(2) // "1010"
  (10).toString(8) // "12"
  (10).toString(16) // "a"

  // 防止小数点识别错误 
  10['toString'](2) // "1010"
```
### Number.prototype.toFixed()
`toFixed`方法先将一个数转为指定位数的小数，然后返回这个小数对应的字符串。
```javascrpt
  (10).toFixed(2) // "10.00"
  10.005.toFixed(2) // "10.01"
```
`toFixed`方法的参数为小数位数，有效范围为0到20，超出这个范围将抛出 `RangeError` 错误。
### Number.prototype.toPrecision()
`toPrecision`方法用于将一个数转为指定位数的有效数字。
```javascrpt
  (12.34).toPrecision(1) // "1e+1"
  (12.34).toPrecision(2) // "12"
  (12.34).toPrecision(3) // "12.3"
  (12.34).toPrecision(4) // "12.34"
  (12.34).toPrecision(5) // "12.340"
```

### 自定义方法
```javascrpt
  Number.prototype.add = function (x) {
    return this + x;
  };
  8['add'](2) // 10
```
