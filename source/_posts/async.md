---
title: async
date: 2018-07-19 22:37:49
tags: [js原生, ES2017, async]
categories: [js原生, 异步编程, async]
---
## 含义
`async` 函数是什么？一句话，它就是 `Generator` 函数的语法糖。
<!--more-->
前文有一个 `Generator` 函数，依次读取两个文件。
```javascript
  const fs = require('fs');

  const readFile = function (fileName) {
    return new Promise(function (resolve, reject) {
      fs.readFile(fileName, function(error, data) {
        if (error) return reject(error);
        resolve(data);
      });
    });
  };

  const gen = function* () {
    const f1 = yield readFile('/etc/fstab');
    const f2 = yield readFile('/etc/shells');
    console.log(f1.toString());
    console.log(f2.toString());
  };
```

写成 `async` 函数，就是下面这样。
```javascript
  const asyncReadFile = async function () {
    const f1 = await readFile('/etc/fstab');
    const f2 = await readFile('/etc/shells');
    console.log(f1.toString());
    console.log(f2.toString());
  };
```

`async` 函数对 `Generator` 函数的改进，体现在以下四点。
* 内置执行器
`Generator` 函数的执行必须靠执行器，所以才有了`co`模块，而`async`函数自带执行器。也就是说，`async`函数的执行，与普通函数一模一样，只要一行。
```javascript
  asyncReadFile();
```
* 更好的语义。
`async` 和 `await`，比起星号和 `yield`，语义更清楚了。`async` 表示函数里有异步操作，`await` 表示紧跟在后面的表达式需要等待结果。
* 更广的适用性
`co` 模块约定，`yield` 命令后面只能是 `Thunk` 函数或 `Promise` 对象，而 `async` 函数的 `await` 命令后面，可以是 `Promise` 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。
* 返回值是 `Promise`
`async`函数的返回值是 `Promise` 对象，这比 `Generator` 函数的返回值是 `Iterator` 对象方便多了。你可以用`then`方法指定下一步的操作。
进一步说，`async`函数完全可以看作多个异步操作，包装成的一个 `Promise` 对象，而`await`命令就是内部`then`命令的语法糖。

## 基本用法
`async`函数返回一个 `Promise` 对象，可以使用`then`方法添加回调函数。当函数执行的时候，一旦遇到`await`就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。
```javascript
  async function getStockPriceByName(name) {
    const symbol = await getStockSymbol(name);
    const stockPrice = await getStockPrice(symbol);
    return stockPrice;
  }

  getStockPriceByName('goog').then(function (result) {
    console.log(result);
  });
```
下面是另一个例子，指定多少毫秒后输出一个值。
```javascript
  function timeout(ms) {
    return new Promise((resolve) => {
      setTimeout(resolve, ms);
    });
  }

  async function asyncPrint(value, ms) {
    await timeout(ms);
    console.log(value);
  }

  asyncPrint('hello world', 50);
```
由于`async`函数返回的是 `Promise` 对象，可以作为`await`命令的参数。所以，上面的例子也可以写成下面的形式。
```javascript
  async function timeout(ms) {
    await new Promise((resolve) => {
      setTimeout(resolve, ms);
    });
  }

  async function asyncPrint(value, ms) {
    await timeout(ms);
    console.log(value);
  }

  asyncPrint('hello world', 50);
```
`async` 函数有多种使用形式。
```javascript
  // 函数声明
  async function foo() {}

  // 函数表达式
  const foo = async function () {};

  // 对象的方法
  let obj = { async foo() {} };
  obj.foo().then(...)

  // Class 的方法
  class Storage {
    constructor() {
      this.cachePromise = caches.open('avatars');
    }

    async getAvatar(name) {
      const cache = await this.cachePromise;
      return cache.match(`/avatars/${name}.jpg`);
    }
  }

  const storage = new Storage();
  storage.getAvatar('jake').then(…);

  // 箭头函数
  const foo = async () => {};
```

## async 语法
** 返回 Promise 对象
`async`函数返回一个 `Promise` 对象。
`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数。
```javascript
  async function f() {
    return 'hello world';
  }

  f().then(v => console.log(v))
  // "hello world"
```

`async`函数内部抛出错误，会导致返回的 `Promise` 对象变为`reject`状态。抛出的错误对象会被`catch`方法回调函数接收到。
```javascript
  async function f() {
    throw new Error('出错了');
  }

  f().then(
    v => console.log(v),
    e => console.log(e)
  )
  // Error: 出错了
```

** Promise 对象的状态改变
```javascript
  async function getTitle(url) {
    let response = await fetch(url);
    let html = await response.text();
    return html.match(/<title>([\s\S]+)<\/title>/i)[1];
  }
  getTitle('https://tc39.github.io/ecma262/').then(console.log)
  // "ECMAScript 2017 Language Specification"

```
下面是一个例子。
```javascript
  async function getTitle(url) {
    let response = await fetch(url);
    let html = await response.text();
    return html.match(/<title>([\s\S]+)<\/title>/i)[1];
  }
  getTitle('https://tc39.github.io/ecma262/').then(console.log)
  // "ECMAScript 2017 Language Specification"
```
上面代码中，函数`getTitle`内部有三个操作：抓取网页、取出文本、匹配页面标题。只有这三个操作全部完成，才会执行`then`方法里面的`console.log`。

** await 命令
正常情况下，`await`命令后面是一个 `Promise` 对象。如果不是，会被转成一个立即`resolve`的 `Promise` 对象。
```javascript
  async function f() {
    return await 123;
  }

  f().then(v => console.log(v))
  // 123
```
上面代码中，`await`命令的参数是数值123，它被转成 `Promise` 对象，并立即`resolve`。

`await`命令后面的 `Promise` 对象如果变为`reject`状态，则`reject`的参数会被`catch`方法的回调函数接收到。
```javascript
  async function f() {
    await Promise.reject('出错了');
  }

  f()
  .then(v => console.log(v))
  .catch(e => console.log(e))
  // 出错了
```

只要一个`await`语句后面的 `Promise` 变为`reject`，那么整个`async`函数都会中断执行。
```javascript
  async function f() {
    await Promise.reject('出错了');
    await Promise.resolve('hello world'); // 不会执行
  }
```

有时，我们希望即使前一个异步操作失败，也不要中断后面的异步操作。这时可以将第一个`await`放在`try...catch`结构里面，这样不管这个异步操作是否成功，第二个`await`都会执行。
```javascript
  async function f() {
    try {
      await Promise.reject('出错了');
    } catch(e) {
    }
    return await Promise.resolve('hello world');
  }

  f()
  .then(v => console.log(v))
  // hello world
```
另一种方法是`await`后面的 `Promise` 对象再跟一个`catch`方法，处理前面可能出现的错误。
```javascript
  async function f() {
    await Promise.reject('出错了')
      .catch(e => console.log(e));
    return await Promise.resolve('hello world');
  }

  f()
  .then(v => console.log(v))
  // 出错了
  // hello world
```

### 错误处理
如果`await`后面的异步操作出错，那么等同于`async`函数返回的 `Promise` 对象被`reject`。
```javascript
  async function f() {
    await new Promise(function (resolve, reject) {
      throw new Error('出错了');
    });
  }

  f()
  .then(v => console.log(v))
  .catch(e => console.log(e))
  // Error：出错了
```
如果有多个`await`命令，可以统一放在`try...catch`结构中。
```javascript
  async function main() {
    try {
      const val1 = await firstStep();
      const val2 = await secondStep(val1);
      const val3 = await thirdStep(val1, val2);

      console.log('Final: ', val3);
    }
    catch (err) {
      console.error(err);
    }
  }
```
## 使用注意点
第一点，前面已经说过，`await`命令后面的`Promise`对象，运行结果可能是`rejected`，所以最好把`await`命令放在`try...catch`代码块中。
```javascript
  async function myFunction() {
    try {
      await somethingThatReturnsAPromise();
    } catch (err) {
      console.log(err);
    }
  }

  // 另一种写法

  async function myFunction() {
    await somethingThatReturnsAPromise()
    .catch(function (err) {
      console.log(err);
    });
  }
```

第二点，多个`await`命令后面的异步操作，如果不存在继发关系，最好让它们同时触发。
```javascript
  // 写法一
  let [foo, bar] = await Promise.all([getFoo(), getBar()]);

  // 写法二
  let fooPromise = getFoo();
  let barPromise = getBar();
  let foo = await fooPromise;
  let bar = await barPromise;
```
第三点，`await`命令只能用在`async`函数之中，如果用在普通函数，就会报错。




