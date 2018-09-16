---
title: this 关键字
date: 2018-07-18 12:40:30
tags: [js原生,Object,this]
categories: [js原生,Object,this]
---
`this`就是属性或方法“当前”所在的对象。
<!--more-->
```JavaScript
  var person = {
    name: '张三',
    describe: function () {
      return '姓名：'+ this.name;
    }
  };
  person.describe()
  // "姓名：张三"

  function f() {
    return '姓名：'+ this.name;
  }
  var A = {
    name: '张三',
    describe: f
  };
  var B = {
    name: '李四',
    describe: f
  };
  A.describe() // "姓名：张三"
  B.describe() // "姓名：李四"
```
再看一个网页编程的例子。
```html
  <input type="text" name="age" size=3 onChange="validate(this, 18, 99);">

  <script>
    function validate(obj, lowval, hival){
      if ((obj.value < lowval) || (obj.value > hival))
        console.log('Invalid Value!');
    }
  </script>
```
`this`的设计目的就是在函数体内部，指代函数当前的运行环境。

## 使用场景
### 全局环境
全局环境使用`this`，它指的就是顶层对象`window`。
### 构造函数
构造函数中的`this`，指的是实例对象。
### 对象的方法
如果对象的方法里面包含`this`，`this`的指向就是方法运行时所在的对象。该方法赋值给另一个对象，就会改变`this`的指向。
### 独立函数或回调函数 
独立函数（未绑定到其他对象上）的`this`指向`window`
回调函数中的 `this` 指向 不确定; 谁调用指向谁。
如：数组的`forEach`方法的回调函数中的`this`，其实是指向`window`对象，可以将`this`当作`forEach`方法的第二个参数，固定它的运行环境


但是，下面这几种用法，都会改变`this`的指向。
```JavaScript
  // 情况一
  (obj.foo = obj.foo)() // window
  // 情况二
  (false || obj.foo)() // window
  // 情况三
  (1, obj.foo)() // window
```
可以这样理解，`JavaScript` 引擎内部，`obj`和`obj.foo`储存在两个内存地址，称为地址一和地址二。`obj.foo()`这样调用时，是从地址一调用地址二，因此地址二的运行环境是地址一，`this`指向`obj`。但是，上面三种情况，都是直接取出地址二进行调用，这样的话，运行环境就是全局环境，因此this指向全局环境。上面三种情况等同于下面的代码。
```JavaScript
  var a = {
    b: {
      m: function() {
        console.log(this.p);
      },
      p: 'Hello'
    }
  };

  var hello = a.b.m; // 将最内层函数赋值给全局变量后 this 指向 全局
  hello() // undefined
```
严格模式下，如果函数内部的this指向顶层对象，就会报错。
```JavaScript
  var counter = {
    count: 0
  };
  counter.inc = function () {
    'use strict';
    this.count++
  };
  var f = counter.inc;
  f()
  // TypeError: Cannot read property 'count' of undefined
```
## 绑定this 的方法
`this`的动态切换，固然为 `JavaScript` 创造了巨大的灵活性，但也使得编程变得困难和模糊。有时，需要把`this`固定下来，避免出现意想不到的情况。`JavaScript` 提供了`call`、`apply`、`bind`这三个方法，来切换/固定`this`的指向。
### Function.prototype.call()
函数实例的`call`方法，可以指定函数内部`this`的指向（即函数执行时所在的作用域），然后在所指定的作用域中，调用该函数。

```JavaScript
  func.call(thisValue, arg1, arg2, ...)
```
`call`的第一个参数就是`this`所要指向的那个对象，后面的参数则是函数调用时所需的参数。

`call`方法的参数，应该是一个对象。如果参数为`空`、`null`和`undefined`，则默认传入全局对象。

如果`call`方法的参数是一个原始值，那么这个原始值会自动转成对应的包装对象，然后传入`call`方法。

### Function.prototype.apply()
`apply`方法的作用与`call`方法类似，也是改变`this`指向，然后再调用该函数。唯一的区别就是，它接收一个数组作为函数执行时的参数，使用格式如下。
```JavaScript
  func.apply(thisValue, [arg1, arg2, ...])
```
原函数的参数，在`call`方法中必须一个个添加，但是在`apply`方法中，必须以**数组**形式添加。

### Function.prototype.bind() 
`bind`方法用于将函数体内的`this`绑定到某个对象，然后返回一个新函数。
```JavaScript
  var counter = {
    count: 0,
    inc: function () {
      this.count++;
    }
  };

  var func = counter.inc.bind(counter);
  func();
  counter.count // 1
```
`bind`还可以接受更多的参数，将这些参数绑定原函数的参数。
```JavaScript
  var add = function (x, y) {
    return x * this.m + y * this.n;
  }
  var obj = {
    m: 2,
    n: 2
  };
  var newAdd = add.bind(obj, 5);
  newAdd(5) // 20
```
#### bind方法有一些使用注意点。
**每一次返回一个新函数**
bind方法每运行一次，就返回一个新函数，这会产生一些问题。比如，监听事件的时候，不能写成下面这样。
```JavaScript
element.addEventListener('click', o.m.bind(o));

// click事件绑定bind方法生成的一个匿名函数。这样会导致无法取消绑定
element.removeEventListener('click', o.m.bind(o));
```
正确的方法是写成下面这样：
```JavaScript
  var listener = o.m.bind(o);
  element.addEventListener('click', listener);
  //  ...
  element.removeEventListener('click', listener);
```
**结合回掉函数使用**
回调函数是 `JavaScript` 最常用的模式之一，但是一个常见的错误是，将包含`this`的方法直接当作回调函数。解决方法就是使用`bind`方法，将`counter.inc`绑定`counter`。
```JavaScript
  var counter = {
    count: 0,
    inc: function () {
      'use strict';
      this.count++;
    }
  };
  function callIt(callback) {
    callback();
  }
  callIt(counter.inc.bind(counter));
  counter.count // 1
```
上面代码中，`callIt`方法会调用回调函数。这时如果直接把`counter.inc`传入，调用时`counter.inc`内部的`this`就会指向全局对象。使用`bind`方法将`counter.inc`绑定`counter`以后，就不会有这个问题，`this`总是指向`counter`。
还有一种情况比较隐蔽，就是某些数组方法可以接受一个函数当作参数。这些函数内部的`this`指向，很可能也会出错。
```JavaScript
  var obj = {
    name: '张三',
    times: [1, 2, 3],
    print: function () {
      this.times.forEach(function (n) {
        console.log(this.name);
      });
    }
  };
  obj.print = function () {
    this.times.forEach(function (n) {
      console.log(this === window);
    });
  };

  obj.print()
  // true
  // true
  // true
```
解决这个问题，也是通过bind方法绑定this。
```JavaScript
obj.print = function () {
  this.times.forEach(function (n) {
    console.log(this.name);
  }.bind(this));
};

obj.print()
// 张三
// 张三
// 张三
```

