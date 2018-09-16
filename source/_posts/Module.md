---
title: Module
date: 2018-07-20 14:18:43
tags: [js原生,]
---

`ES6` 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。`CommonJS` 和 `AMD` 模块，都只能在运行时确定这些东西。比如，`CommonJS` 模块就是对象，输入时必须查找对象属性。

```javascript
  // CommonJS模块
  let { stat, exists, readFile } = require('fs');

  // 等同于
  let _fs = require('fs');
  let stat = _fs.stat;
  let exists = _fs.exists;
  let readfile = _fs.readfile;
```
上面代码的实质是整体加载`fs`模块（即加载fs的所有方法），生成一个对象（`_fs`），然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

`ES6` 模块不是对象，而是通过`export`命令显式指定输出的代码，再通过`import`命令输入。
```javascript
  // ES6模块
  import { stat, exists, readFile } from 'fs';
```
上面代码的实质是从fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 `ES6` 可以在编译时就完成模块加载，效率要比 `CommonJS` 模块的加载方式高。当然，这也导致了没法引用 `ES6` 模块本身，因为它不是对象。

## 严格模式
**ES6** 的模块自动采用严格模式，不管你有没有在模块头部加上`"use strict"`;。
## export
> 模块功能主要由两个命令构成：`export`和`import`。`export`命令用于规定模块的对外接口，`import`命令用于输入其他模块提供的功能。

`export`的写法:
* 写法一
```javascript
  // profile.js
  export var firstName = 'Michael';
  export var lastName = 'Jackson';
  export var year = 1958;
```

* 写法二
```javascript
  // profile.js
  var firstName = 'Michael';
  var lastName = 'Jackson';
  var year = 1958;

  export {firstName, lastName, year};
```
通常情况下，`export`输出的变量就是本来的名字，但是可以使用`as`关键字重命名。
```javascript
  function v1() { ... }
  function v2() { ... }

  export {
    v1 as streamV1,
    v2 as streamV2,
    v2 as streamLatestVersion
  };
```
`export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。
```javascript
  // 写法一
  export var m = 1;

  // 写法二
  var m = 1;
  export {m};
  // 导出 function
  export function f() {};
  // 或者
  function f() {}
  export {f};

  // 写法三
  var n = 1;
  export {n as m};
```

## import 命令
```javascript
// main.js
  import {firstName, lastName, year} from './profile.js';

  function setName(element) {
    element.textContent = firstName + ' ' + lastName;
  }
```
如果想为输入的变量重新取一个名字，`import`命令要使用`as`关键字，将输入的变量重命名。
```javascript
  import { lastName as surname } from './profile.js';
```

## 模块的整体加载
除了指定加载某个输出值，还可以使用整体加载，即用星号（`*`）指定一个对象，所有输出值都加载在这个对象上面。

下面是一个circle.js文件，它输出两个方法area和circumference。
```javascript
  // circle.js

  export function area(radius) {
    return Math.PI * radius * radius;
  }

  export function circumference(radius) {
    return 2 * Math.PI * radius;
  }

  import * as circle from './circle';

  console.log('圆面积：' + circle.area(4));
  console.log('圆周长：' + circle.circumference(14));
```

## export default 命令
如果想在一条`import`语句中，同时输入默认方法和其他接口，可以写成下面这样。
```javascript
import _, { each, each as forEach } from 'lodash';
```