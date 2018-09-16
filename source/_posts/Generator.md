---
title: Generator
date: 2018-07-19 19:00:42
tags: [js原生, 异步编程, Generator]
categories: [js原生, 异步编程, Generator]
---
`Generator` 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。`Generator` 函数有多种理解角度。语法上，首先可以把它理解成，`Generator` 函数是一个状态机，封装了多个内部状态。
<!--more-->
执行 `Generator` 函数会返回一个遍历器对象，也就是说，`Generator` 函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历 `Generator` 函数内部的每一个状态。

`Generator`函数有两个特征
* `function`关键字与函数名之间有一个`*`.
* 函数体内部使用`yield`表达式，定义不同的内部状态（`yield`在英语里的意思就是“产出”）。
```javascript
  function* helloWorldGenerator() {
    yield 'hello';
    yield 'world';
    return 'ending';
  }
  var hw = helloWorldGenerator();

  hw.next()
  // { value: 'hello', done: false }

  hw.next()
  // { value: 'world', done: false }

  hw.next()
  // { value: 'ending', done: true }

  hw.next()
  // { value: undefined, done: true }
```
`Generator` 函数是分段执行的，`yield`表达式是暂停执行的标记，而`next`方法可以恢复执行。

## yield 表达式
由于 `Generator` 函数返回的遍历器对象，只有调用`next`方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。`yield`表达式就是暂停标志。

遍历器对象的`next`方法的运行逻辑如下。
* 遇到`yield`表达式，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值。
* 下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式。
* 如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，并将`return`语句后面的表达式的值，作为返回的对象的`value`属性值。
* 如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`。
```javascript
  function* gen() {
    yield  123 + 456;
  }
```
上面代码中，`yield`后面的表达式`123 + 456`，不会立即求值，只会在`next`方法将指针移到这一句时，才会求值。

`yield`表达式如果用在另一个表达式之中，必须放在圆括号里面。

## next 方法的参数
`yield`表达式本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数就会被当作上一个`yield`表达式的返回值。
```javascript
  function* f() {
    for(var i = 0; true; i++) {
      var reset = yield i;
      if(reset) { i = -1; }
    }
  }

  var g = f();

  g.next() // { value: 0, done: false }
  g.next() // { value: 1, done: false }
  g.next(true) // { value: 0, done: false }
```

`Generator` 函数从暂停状态到恢复运行，它的上下文状态（`context`）是不变的。通过next方法的参数，就有办法在 `Generator` 函数开始运行之后，继续向函数体内部注入值。也就是说，可以在 `Generator` 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。
由于next方法的参数表示上一个yield表达式的返回值，所以在第一次使用next方法时，传递参数是无效的。V8 引擎直接忽略第一次使用next方法时的参数，只有从第二次使用next方法开始，参数才是有效的。从语义上讲，第一个next方法用来启动遍历器对象，所以不用带有参数。

再看一个通过`next`方法的参数，向 `Generator` 函数内部输入值的例子。
```javascript
  function* dataConsumer() {
    console.log('Started');
    console.log(`1. ${yield}`);
    console.log(`2. ${yield}`);
    return 'result';
  }

  let genObj = dataConsumer();
  genObj.next();
  // Started
  genObj.next('a')
  // 1. a
  genObj.next('b')
  // 2. b
```
### for...of 循环
`for...of`循环可以自动遍历 `Generator` 函数时生成的`Iterator`对象，且此时不再需要调用`next`方法。
```javascript
  function* foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    yield 5;
    return 6;
  }

  for (let v of foo()) {
    console.log(v);
  }
```
上面代码使用`for...of`循环，依次显示 5 个`yield`表达式的值。这里需要注意，一旦`next`方法的返回对象的`done`属性为`true`，`for...of`循环就会中止，且不包含该返回对象，所以上面代码的`return`语句返回的6，不包括在`for...of`循环之中。

下面是一个利用 `Generator` 函数和`for...of`循环，实现斐波那契数列的例子。
```javascript
  function* fibonacci() {
    let [prev, curr] = [0, 1];
    for (;;) {
      yield curr;
      [prev, curr] = [curr, prev + curr];
    }
  }

  for (let n of fibonacci()) {
    if (n > 1000) break;
    console.log(n);
  }
```
从上面代码可见，使用`for...of`语句时不需要使用`next`方法。
利用`for...of`循环，可以写出遍历任意对象（`object`）的方法。原生的 `JavaScript` 对象没有遍历接口，无法使用`for...of`循环，通过 `Generator` 函数为它加上这个接口，就可以用了。
```javascript
  function* objectEntries(obj) {
    let propKeys = Reflect.ownKeys(obj);

    for (let propKey of propKeys) {
      yield [propKey, obj[propKey]];
    }
  }

  let jane = { first: 'Jane', last: 'Doe' };

  for (let [key, value] of objectEntries(jane)) {
    console.log(`${key}: ${value}`);
  }
  // first: Jane
  // last: Doe  
```
上面代码中，对象`jane`原生不具备 `Iterator` 接口，无法用`for...of`遍历。这时，我们通过 `Generator` 函数`objectEntries`为它加上遍历器接口，就可以用`for...of`遍历了。加上遍历器接口的另一种写法是，将 `Generator` 函数加到对象的`Symbol.iterator`属性上面。
```javascript
  function* objectEntries() {
    let propKeys = Object.keys(this);

    for (let propKey of propKeys) {
      yield [propKey, this[propKey]];
    }
  }

  let jane = { first: 'Jane', last: 'Doe' };

  jane[Symbol.iterator] = objectEntries;

  for (let [key, value] of jane) {
    console.log(`${key}: ${value}`);
  }
  // first: Jane
  // last: Doe
```
除了`for...of`循环以外，扩展运算符（`...`）、解构赋值和`Array.from`方法内部调用的，都是遍历器接口。这意味着，它们都可以将 `Generator` 函数返回的 `Iterator` 对象，作为参数。
```javascript
  function* numbers () {
    yield 1
    yield 2
    return 3
    yield 4
  }

  // 扩展运算符
  [...numbers()] // [1, 2]

  // Array.from 方法
  Array.from(numbers()) // [1, 2]

  // 解构赋值
  let [x, y] = numbers();
  x // 1
  y // 2

  // for...of 循环
  for (let n of numbers()) {
    console.log(n)
  }
  // 1
  // 2
```
## Generator.prototype.throw()
`Generator` 函数返回的遍历器对象，都有一个`throw`方法，可以在函数体外抛出错误，然后在 `Generator` 函数体内捕获。
```javascript
  var g = function* () {
    try {
      yield;
    } catch (e) {
      console.log('内部捕获', e);
    }
  };

  var i = g();
  i.next();

  try {
    i.throw('a');
    i.throw('b'); // 不会被捕获
  } catch (e) {
    console.log('外部捕获', e);
  }
  // 内部捕获 a
  // 外部捕获 b
```
`throw`方法抛出的错误要被内部捕获，前提是必须至少执行过一次`next`方法。
如果 `Generator` 函数内部没有部署`try...catch`代码块，那么`throw`方法抛出的错误，将被外部`try...catch`代码块捕获。

如果 `Generator` 函数内部和外部，都没有部署`try...catch`代码块，那么程序将报错，直接中断执行。

`throw`方法被捕获以后，会附带执行下一条`yield`表达式。也就是说，会附带执行一次`next`方法。
```javascript
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}

var g = gen();
g.next() // a
g.throw() // b
g.next() // c
```
上面代码中，`g.throw`方法被捕获以后，自动执行了一次`next`方法，所以会打印`b`。另外，也可以看到，只要 `Generator` 函数内部部署了`try...catch`代码块，那么遍历器的`throw`方法抛出的错误，不影响下一次遍历。

`Generator` 函数体外抛出的错误，可以在函数体内捕获；反过来，`Generator` 函数体内抛出的错误，也可以被函数体外的`catch`捕获。

## Generator.prototype.return()
`Generator` 函数返回的遍历器对象，还有一个`return`方法，可以返回给定的值，并且终结遍历 `Generator` 函数。
```javascript
  function* gen() {
    yield 1;
    yield 2;
    yield 3;
  }

  var g = gen();

  g.next()        // { value: 1, done: false }
  g.return('foo') // { value: "foo", done: true }
  g.next()        // { value: undefined, done: true }
```

## yield* 表达式
如果在 `Generator` 函数内部，调用另一个 `Generator` 函数，默认情况下是没有效果的。

```javascript
  function* foo() {
    yield 'a';
    yield 'b';
  }

  function* bar() {
    yield 'x';
    foo();
    yield 'y';
  }

  for (let v of bar()){
    console.log(v);
  }
  // "x"
  // "y"
```

这个就需要用到`yield*`表达式，用来在一个 `Generator` 函数里面执行另一个 `Generator` 函数。
```javascript
  function* bar() {
    yield 'x';
    yield* foo();
    yield 'y';
  }

  // 等同于
  function* bar() {
    yield 'x';
    yield 'a';
    yield 'b';
    yield 'y';
  }

  // 等同于
  function* bar() {
    yield 'x';
    for (let v of foo()) {
      yield v;
    }
    yield 'y';
  }

  for (let v of bar()){
    console.log(v);
  }
  // "x"
  // "a"
  // "b"
  // "y"
```
有`return`语句时，则需要用`var value = yield* iterator`的形式获取`return`语句的值。
```javascript
  function* foo() {
    yield 2;
    yield 3;
    return "foo";
  }

  function* bar() {
    yield 1;
    var v = yield* foo();
    console.log("v: " + v);
    yield 4;
  }

  var it = bar();

  it.next()
  // {value: 1, done: false}
  it.next()
  // {value: 2, done: false}
  it.next()
  // {value: 3, done: false}
  it.next();
  // "v: foo"
  // {value: 4, done: false}
  it.next()
  // {value: undefined, done: true}
```

**yield*命令可以很方便地取出嵌套数组的所有成员。**
```javascript
  function* iterTree(tree) {
    if (Array.isArray(tree)) {
      for(let i=0; i < tree.length; i++) {
        yield* iterTree(tree[i]);
      }
    } else {
      yield tree;
    }
  }

  const tree = [ 'a', ['b', 'c'], ['d', 'e'] ];

  for(let x of iterTree(tree)) {
    console.log(x);
  }
  // a
  // b
  // c
  // d
  // e
```

## 应用
`Generator` 可以暂停函数执行，返回任意表达式的值。这种特点使得 `Generator` 有多种应用场景。

### 异步操作的同步化表达
```javascript
  function* loadUI() {
    showLoadingScreen();
    yield loadUIDataAsynchronously();
    hideLoadingScreen();
  }
  var loader = loadUI();
  // 加载UI
  loader.next()

  // 卸载UI
  loader.next()
```

上面代码中，第一次调用`loadUI`函数时，该函数不会执行，仅返回一个遍历器。下一次对该遍历器调用`next`方法，则会显示`Loading`界面（`showLoadingScreen`），并且异步加载数据（`loadUIDataAsynchronously`）。等到数据加载完成，再一次使用`next`方法，则会隐藏`Loading`界面。可以看到，这种写法的好处是所有`Loading`界面的逻辑，都被封装在一个函数，按部就班非常清晰。

`Ajax` 是典型的异步操作，通过 `Generator` 函数部署 `Ajax` 操作，可以用同步的方式表达。
```javascript
  function* main() {
    var result = yield request("http://some.url");
    var resp = JSON.parse(result);
      console.log(resp.value);
  }

  function request(url) {
    makeAjaxCall(url, function(response){
      it.next(response);
    });
  }

  var it = main();
  it.next();
```
### 控制流管理
```javascript
  function* longRunningTask(value1) {
    try {
      var value2 = yield step1(value1);
      var value3 = yield step2(value2);
      var value4 = yield step3(value3);
      var value5 = yield step4(value4);
      // Do something with value4
    } catch (e) {
      // Handle any error from step1 through step4
    }
  }

```
然后，使用一个函数，按次序自动执行所有步骤。
```javascript
  scheduler(longRunningTask(initialValue));

  function scheduler(task) {
    var taskObj = task.next(task.value);
    // 如果Generator函数未结束，就继续调用
    if (!taskObj.done) {
      task.value = taskObj.value
      scheduler(task);
    }
  }
```

