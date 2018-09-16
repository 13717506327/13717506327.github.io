---
title: function
date: 2018-07-15 06:39:58
tags: [js原生,function]
categories: js原生
---

## 函数的声明
函数 有三种申明方式：
- function 命令
- 函数表达式
- Function 构造函数

<!--more-->
### function 命令
```JavaScript
  function print(s) {
    console.log(s);
  }
```
### 函数表达式
采用函数表达式声明函数时，function命令后面不带有函数名。如果加上`函数名`，该函数名只在函数体`内部有效`，在函数体 `外部无效`。
```JavaScript
  var print = function(s) {
    console.log(s);
  };

  var print = function x(){
    // x 只能在此函数内部使用
    console.log(typeof x);
  };
```
### Function
最后一个参数是add函数的“函数体”，其他参数都是add函数的参数。
```JavaScript
  var add = new Function(
    'x',
    'y',
    'return x + y'
  );

  // 等同于
  function add(x, y) {
    return x + y;
  }
```
## 函数的属性和方法
### name属性
返回函数的名字
```JavaScript
  function f1() {}
  f1.name // "f1"
  
  var f2 = function () {};
  f2.name // "f2"

  
  var f3 = function myName() {};
  // 返回 function命令后 函数名
  f3.name // 'myName'
```
### length属性
返回函数预期传入的参数个数，即函数定义之中的参数个数。
```JavaScript
  // 不管调用时输入了多少个参数，length属性始终等于2。
  function f(a, b) {}
  f.length // 2
```
------
## 函数的参数
### 参数的传递方式
> 函数参数如果是`原始类型`的值（数值、字符串、布尔值），传递方式是`传值传递`（passes by value）。这意味着，在函数体内修改参数值，不会影响到函数外部；如果函数参数是`复合类型`的值（数组、对象、其他函数），传递方式是`传址传递`（pass by reference）
### arguments 对象
arguments读取函数调用时传入函数内部所有的参数，arguments为只读对象(伪数组)
```JavaScript
var args = Array.prototype.slice.call(arguments);

// ES6
var argus = Array.from(arguments)
```
### callee
arguments对象带有一个callee属性，返回它所对应的原函数;可以通过arguments.callee，达到调用函数自身的目的。这个属性在严格模式里面是禁用的，因此不建议使用。

## 函数的其他特性
### 闭包
闭包的最大用处有两个，一个是可以读取函数内部的变量，另一个就是让这些变量始终保持在内存中，即闭包可以使得它的诞生环境一直存在。

注意，外层函数每次运行，都会生成一个新的闭包，而这个闭包又会保留外层函数的内部变量，所以内存消耗很大。因此不能滥用闭包，否则会造成网页的性能问题。
```JavaScript
  function createIncrementor(start) {
    // 为了在函数外部读取 start的值 将 start的 值返回
    return function () {
      return start++;
    };
  }

  var inc = createIncrementor(5);
  // inc使得函数createIncrementor的内部环境，一直存在于内存中。所以，闭包可以看作是函数内部作用域的一个接口。
  inc() // 5
  inc() // 6
  inc() // 7
```