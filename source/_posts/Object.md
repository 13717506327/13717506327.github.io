---
title: Object
date: 2018-07-16 14:11:47
tags: [js原生, object]
categories: [js原生, Object]
---

## 概述
JavaScript 的所有其他对象都继承自Object对象，即那些对象都是Object的实例。
Object对象的原生方法分成两类：Object本身的方法与Object的实例方法。
### Object 对象本身的方法
所谓”本身的方法“就是直接定义在Object对象的方法。
```JavaScript
  Object.print = function (o) { console.log(o) };
```
<!--more-->
### Object的实例方法
所谓实例方法就是定义在Object原型对象Object.prototype上的方法。它可以被Object实例直接使用。
```JavaScript
  Object.prototype.print = function () {
    console.log(this);
  };

  var obj = new Object();
  obj.print() // Object
```
## Object 构造函数
`Object`不仅可以当作工具函数使用，还可以当作构造函数使用，即前面可以使用`new`命令。
### Object 用作工具函数
```JavaScript
  var obj = Object();
  // 等同于
  var obj = Object(undefined);
  var obj = Object(null);

  obj instanceof Object // true

  // 参数为原始诗句类型
  var obj = Object(1);
  obj instanceof Object // true
  obj instanceof Number // true

  var obj = Object('foo');
  obj instanceof Object // true
  obj instanceof String // true

  var obj = Object(true);
  obj instanceof Object // true
  obj instanceof Boolean // true

  // 参数为复杂数据类型
  var arr = [];
  var obj = Object(arr); // 返回原数组
  obj === arr // true

  var value = {};
  var obj = Object(value) // 返回原对象
  obj === value // true

  var fn = function () {};
  var obj = Object(fn); // 返回原函数
  obj === fn // true
```
利用工具函数 判断变量是否为对象的函数。
```JavaScript
  function isObject(value) {
    return value === Object(value);
  }

  isObject([]) // true
  isObject(true) // false
```
### Object构造函数的用法
Object构造函数的用法与工具方法很相似，几乎一模一样。使用时，可以接受一个参数，如果该参数是一个对象，则直接返回这个对象.
```JavaScript
  var o1 = {a: 1};
  var o2 = new Object(o1);
  o1 === o2 // true

  var obj = new Object(123);
  obj instanceof Number // true
```

## Object 的静态方法
所谓“静态方法”，是指部署在Object对象自身的方法。
### Object.keys（）
Object.keys方法的参数是一个对象，返回一个数组。其元素来自于从给定的object上面`可直接枚举`的属性。这些属性的顺序与手动遍历该对象属性时的一致。
```JavaScript
  var obj = {
    p1: 123,
    p2: 456
  };

  Object.keys(obj) // ["p1", "p2"]
```
### Object.getOwnPropertyNames
与Object.keys类似，也是接受一个对象作为参数，返回一个数组，包含了该对象自身的所有属性名。
### 其他方法
#### 对象属性模型的相关方法
* Object.getOwnPropertyDescriptor()：获取某个属性的描述对象。
* Object.defineProperty()：通过描述对象，定义某个属性。
* Object.defineProperties()：通过描述对象，定义多个属性。
#### 控制对象状态的方法
* Object.preventExtensions()：防止对象扩展。
* Object.isExtensible()：判断对象是否可扩展。
* Object.seal()：禁止对象配置。
* Object.isSealed()：判断一个对象是否可配置。
* Object.freeze()：冻结一个对象。
* Object.isFrozen()：判断一个对象是否被冻结。
#### 原型链相关方法
* Object.create()：该方法可以指定原型对象和属性，返回一个新的对象。
* Object.getPrototypeOf()：获取对象的Prototype对象。


## Object 的实例方法
除了静态方法，还有不少方法定义在Object.prototype对象。它们称为实例方法，所有Object的实例对象都继承了这些方法。
* Object.prototype.valueOf()：返回当前对象对应的值。
* Object.prototype.toString()：返回当前对象对应的字符串形式。
* Object.prototype.toLocaleString()：返回当前对象对应的本地字符串形式。
* Object.prototype.hasOwnProperty()：判断某个属性是否为当前对象自身的属性
* Object.prototype.isPrototypeOf()：判断当前对象是否为另一个对象的原型。
* Object.prototype.propertyIsEnumerable()：判断某个属性是否可枚举。

### Object.prototype.valueOf()
valueOf方法的作用是返回一个对象的“值”，默认情况下返回对象本身。
valueOf方法的主要用途是，JavaScript 自动类型转换时会默认调用这个方法。
```JavaScript
  var obj = new Object();
  obj.valueOf() === obj // true
  
  var obj = new Object();
  1 + obj // "1[object Object]"

  // 如果自定义valueOf方法，就可以得到想要的结果。
  var obj = new Object();
  obj.valueOf = function () {
    return 2;
  };
  1 + obj // 3
```

### Object.prototype.toString()
toString方法的作用是返回一个对象的字符串形式，默认情况下返回类型字符串。对于一个对象调用toString方法，会返回字符串`[object Object]`;字符串[object Object]本身没有太大的用处，但是通过自定义toString方法，可以让对象在自动类型转换时，得到想要的字符串形式。数组、字符串、函数、Date 对象都分别部署了自定义的toString方法，覆盖了Object.prototype.toString方法。
```JavaScript
  var o1 = new Object();
  o1.toString() // "[object Object]"

  var o2 = {a:1};
  o2.toString() // "[object Object]"

  var obj = new Object();
  obj.toString = function () {
    return 'hello';
  };
  obj + ' ' + 'world' // "hello world"
  
  // 其他数据类型的 toString方法。
  [1, 2, 3].toString() // "1,2,3"

  '123'.toString() // "123"

  (function () {
    return 123;
  }).toString()
  // "function () {
  //   return 123;
  // }"

  (new Date()).toString()
  // "Tue May 10 2016 09:11:31 GMT+0800 (CST)"
```
#### toString() 的应用：判断数据类型
Object.prototype.toString方法返回对象的类型字符串，因此可以用来判断一个值的类型。由于实例对象可能会自定义toString方法，覆盖掉Object.prototype.toString方法，所以为了得到类型字符串，最好直接使用Object.prototype.toString方法。
```JavaScript
  var type = function (o){
    var s = Object.prototype.toString.call(o);
    return s.match(/\[object (.*?)\]/)[1].toLowerCase();
  };

  ['Null',
  'Undefined',
  'Object',
  'Array',
  'String',
  'Number',
  'Boolean',
  'Function',
  'RegExp'
  ].forEach(function (t) {
    type['is' + t] = function (o) {
      return type(o) === t.toLowerCase();
    };
  });

  type.isObject({}) // true
  type.isNumber(NaN) // true
  type.isRegExp(/abc/) // true

```

### Object.prototype.toLocaleString()
这个方法的主要作用是留出一个接口，让各种不同的对象实现自己版本的toLocaleString，用来返回针对某些地域的特定的值。
```JavaScript
  var obj = {};
  obj.toString(obj) // "[object Object]"
  obj.toLocaleString(obj) // "[object Object]"
  
  var date = new Date();
  date.toString() // "Tue Jan 01 2018 12:01:33 GMT+0800 (CST)"
  date.toLocaleString() // "1/01/2018, 12:01:33 PM"
```

### Object.prototype.hasOwnProperty()
Object.prototype.hasOwnProperty方法接受一个字符串作为参数，返回一个布尔值，表示该实例对象自身是否具有该属性。
```JavaScript
  var obj = {
    p: 123
  };
  obj.hasOwnProperty('p') // true
  obj.hasOwnProperty('toString') // false
```