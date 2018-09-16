---
title: Promise
date: 2018-07-18 19:39:11
tags: [js原生, ES6, Promise]
categories: [js原生, 异步编程, Promise]  
---
Promise 可以让异步操作写起来，就像在写同步操作的流程，而不必一层层地嵌套回调函数。
<!--more-->
## Promise 特点
Promise对象有以下两个特点：
* Promise对象状态不受外界影响
`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。
* 一旦状态改变，就不会再变，任何时候都可以得到这个结果。
`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 `resolved`（已定型）。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（`Event`）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

后面的resolved统一只指fulfilled状态，不包含rejected状态。

## 基本用法
下面代码创造了一个`Promise`实例。
```javascript
  const promise = new Promise(function(resolve, reject) {
    // ... some code

    if (/* 异步操作成功 */){
      resolve(value);
    } else {
      reject(error);
    }
  });
```
`Promise`构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由 `JavaScript` 引擎提供，不用自己部署。

`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”（即从 `pending` 变为 `resolved`），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”（即从 `pending` 变为 `rejected`），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。
```javascript
  promise.then(function(value) {
    // success
  }, function(error) {
    // failure
  });
```
`then`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个回调函数是`Promise`对象的状态变为`rejected`时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受`Promise`对象传出的值作为参数。

下面是一个`Promise`对象的简单例子。
```javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}
timeout(100).then((value) => {
  console.log(value);
});
```
**`Promise` 新建后就会立即执行。**
```javascript
  let promise = new Promise(function(resolve, reject) {
    console.log('Promise');
    resolve();
  });

  promise.then(function() {
    console.log('resolved.');
  });

  console.log('Hi!');

  // Promise
  // Hi!
  // resolved
```
上面代码中，`Promise` 新建后立即执行，所以首先输出的是`Promise`。然后，then方法指定的回调函数，将在当前脚本所有**同步任务**执行完才会执行，所以`resolved`最后输出。

下面是异步加载图片的例子。
```javascript
  function loadImageAsync(url) {
    return new Promise(function(resolve, reject) {
      const image = new Image();

      image.onload = function() {
        resolve(image);
      };

      image.onerror = function() {
        reject(new Error('Could not load image at ' + url));
      };

      image.src = url;
    });
  }
```
上面代码中，使用`Promise`包装了一个图片加载的异步操作。如果加载成功，就调用`resolve`方法，否则就调用`reject`方法。

下面是一个用`Promise`对象实现的 Ajax 操作的例子。
```javascript
  const getJSON = function(url) {
    const promise = new Promise(function(resolve, reject){
      const handler = function() {
        if (this.readyState !== 4) {
          return;
        }
        if (this.status === 200) {
          resolve(this.response);
        } else {
          reject(new Error(this.statusText));
        }
      };
      const client = new XMLHttpRequest();
      client.open("GET", url);
      client.onreadystatechange = handler;
      client.responseType = "json";
      client.setRequestHeader("Accept", "application/json");
      client.send();

    });

    return promise;
  };

  getJSON("/posts.json").then(function(json) {
    console.log('Contents: ' + json);
  }, function(error) {
    console.error('出错了', error);
  });
```
上面代码中，`getJSON`是对 `XMLHttpRequest` 对象的封装，用于发出一个针对 `JSON` 数据的 `HTTP` 请求，并且返回一个`Promise`对象。需要注意的是，在`getJSON`内部，`resolve`函数和`reject`函数调用时，都带有参数。

如果调用`resolve`函数和`reject`函数时带有参数，那么它们的参数会被传递给回调函数。reject函数的参数通常是`Error`对象的实例，表示抛出的错误；`resolve`函数的参数除了正常的值以外，还可能是另一个 `Promise` 实例，比如像下面这样。
```javascript
  const p1 = new Promise(function (resolve, reject) {
    // ...
  });

  const p2 = new Promise(function (resolve, reject) {
    // ...
    resolve(p1);
  })
```
上面代码中，`p1`和`p2`都是 `Promise` 的实例，但是`p2`的`resolve`方法将`p1`作为参数，即一个异步操作的结果是返回另一个异步操作。

注意，这时`p1`的状态就会传递给`p2`，也就是说，`p1`的状态决定了`p2`的状态。如果`p1`的状态是`pending`，那么`p2`的回调函数就会等待`p1`的状态改变；如果`p1`的状态已经是`resolved`或者`rejected`，那么`p2`的回调函数将会立刻执行。

注意，调用`resolve`或`reject`并不会终结 `Promise` 的参数函数的执行。

```javascript
  new Promise((resolve, reject) => {
    resolve(1);
    console.log(2);
  }).then(r => {
    console.log(r);
  });
  // 2
  // 1
```
上面代码中，调用`resolve(1)`以后，后面的`console.log(2)`还是会执行，并且会首先打印出来。这是因为立即 `resolved` 的 `Promise` 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。

一般来说，调用`resolve`或`reject`以后，`Promise` 的使命就完成了，后继操作应该放到`then`方法里面，而不应该直接写在`resolve`或`reject`的后面。所以，最好在它们前面加上`return`语句，这样就不会有意外。

```javascript
  new Promise((resolve, reject) => {
    return resolve(1);
    // 后面的语句不会执行
    console.log(2);
  })
```

## Promise.prototype.then()

`Promise` 实例具有`then`方法，也就是说，`then`方法是定义在原型对象`Promise.prototype`上的。它的作用是为 `Promise` 实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是`resolved`状态的回调函数，第二个参数（可选）是`rejected`状态的回调函数。

```javascript
  getJSON("/posts.json").then(function(json) {
    return json.post;
  }).then(function(post) {
    // ...
  });
```
采用链式的`then`，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个`Promise`对象（即有异步操作），这时后一个回调函数，就会等待该`Promise`对象的状态发生变化，才会被调用。

```javascript
  getJSON("/post/1.json").then(function(post) {
    return getJSON(post.commentURL);
  }).then(function funcA(comments) {
    console.log("resolved: ", comments);
  }, function funcB(err){
    console.log("rejected: ", err);
  });
```
上面代码中，第一个`then`方法指定的回调函数，返回的是另一个`Promise`对象。这时，第二个`then`方法指定的回调函数，就会等待这个新的`Promise`对象状态发生变化。如果变为`resolved`，就调用`funcA`，如果状态变为`rejected`，就调用`funcB`。

## Promise.prototype.catch()
`Promise.prototype.catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数。

```javascript
  getJSON('/posts.json').then(function(posts) {
    // ...
  }).catch(function(error) {
    // 处理 getJSON 和 前一个回调函数运行时发生的错误
    console.log('发生错误！', error);
  });
```

上面代码中，`getJSON`方法返回一个 `Promise` 对象，如果该对象状态变为`resolved`，则会调用`then`方法指定的回调函数；如果异步操作抛出错误，状态就会变为`rejected`，就会调用`catch`方法指定的回调函数，处理这个错误。另外，`then`方法指定的回调函数，如果运行中抛出错误，也会被`catch`方法捕获。

```javascript
  p.then((val) => console.log('fulfilled:', val))
    .catch((err) => console.log('rejected', err));

  // 等同于
  p.then((val) => console.log('fulfilled:', val))
    .then(null, (err) => console.log("rejected:", err));

  const promise = new Promise(function(resolve, reject) {
    throw new Error('test');
  });
  promise.catch(function(error) {
    console.log(error);
  });
  // Error: test
```

上面代码中，`promise`抛出一个错误，就被`catch`方法指定的回调函数捕获。注意，上面的写法与下面两种写法是等价的。
```javascript
  // 写法一
  const promise = new Promise(function(resolve, reject) {
    try {
      throw new Error('test');
    } catch(e) {
      reject(e);
    }
  });
  promise.catch(function(error) {
    console.log(error);
  });

  // 写法二
  const promise = new Promise(function(resolve, reject) {
    reject(new Error('test')); // reject方法的作用，等同于抛出错误。
  });
  promise.catch(function(error) {
    console.log(error);
  });
```
**注意**：`Promise` 在`resolve`语句后面，再抛出错误，不会被捕获

`Promise` 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个`catch`语句捕获。

```javascript
  getJSON('/post/1.json').then(function(post) {
    return getJSON(post.commentURL);
  }).then(function(comments) {
    // some code
  }).catch(function(error) {
    // 处理前面三个Promise产生的错误
  });
```

一般来说，不要在then方法里面定义 `Reject` 状态的回调函数（即`then`的第二个参数），总是使用`catch`方法。

`catch`方法返回的还是一个 `Promise` 对象，因此后面还可以接着调用then方法。
```javascript
  const someAsyncThing = function() {
    return new Promise(function(resolve, reject) {
      // 下面一行会报错，因为x没有声明
      resolve(x + 2);
    });
  };

  someAsyncThing()
  .catch(function(error) {
    console.log('oh no', error);
  })
  .then(function() {
    console.log('carry on');
  });
  // oh no [ReferenceError: x is not defined]
  // carry on
```

## Promise.prototype.finally()
`finally`方法用于指定不管 `Promise` 对象最后状态如何，都会执行的操作。该方法是 `ES2018` 引入标准的。
```javascript
  promise
  .then(result => {···})
  .catch(error => {···})
  .finally(() => {···});
```

## Promise.all()
`Promise.all`方法用于将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。
```javascript
  const p = Promise.all([p1, p2, p3]);
```
只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给p的回调函数。
```javascript
  // 生成一个Promise对象的数组
  const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
    return getJSON('/post/' + id + ".json");
  });

  Promise.all(promises).then(function (posts) {
    // ...
  }).catch(function(reason){
    // ...
  });
```
上面代码中，`promises`是包含 6 个 `Promise` 实例的数组，只有这 6 个实例的状态都变成`fulfilled`，或者其中有一个变为`rejected`，才会调用`Promise.all`方法后面的回调函数。

## Promise.race()
`Promise.race`方法同样是将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。
```javascript
  const p = Promise.race([p1, p2, p3]);
```
只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 `Promise` 实例的返回值，就传递给`p`的回调函数。

## Promise.resolve()
有时需要将现有对象转为 Promise 对象，Promise.resolve方法就起到这个作用。
```javascript
  const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```
上面代码将 `jQuery` 生成的`deferred`对象，转为一个新的 `Promise` 对象。
```javascript
  Promise.resolve('foo')
  // 等价于
  new Promise(resolve => resolve('foo'))
```

**参数是一个 Promise 实例**
如果参数是 `Promise` 实例，那么`Promise.resolve`将不做任何修改、原封不动地返回这个实例。
**参数是一个thenable对象**
```javascript
  let thenable = {
    then: function(resolve, reject) {
      resolve(42);
    }
  };

  let p1 = Promise.resolve(thenable);
  p1.then(function(value) {
    console.log(value);  // 42
  });
```
上面代码中，`thenable`对象的`then`方法执行后，对象`p1`的状态就变为`resolved`，从而立即执行最后那个`then`方法指定的回调函数，输出 42。

**参数不是具有then方法的对象，或根本就不是对象**
如果参数是一个原始值，或者是一个不具有`then`方法的对象，则`Promise.resolve`方法返回一个新的 `Promise` 对象，状态为`resolved`。
```javascript
  const p = Promise.resolve('Hello');

  p.then(function (s){
    console.log(s)
  });
  // Hello
```

**不带有任何参数**
`Promise.resolve`方法允许调用时不带参数，直接返回一个`resolved`状态的 `Promise` 对象。
需要注意的是，立即`resolve`的 `Promise` 对象，是在本轮“事件循环”（`event loop`）的结束时，而不是在下一轮“事件循环”的开始时。
```javascript
  setTimeout(function () {
    console.log('three');
  }, 0);

  Promise.resolve().then(function () {
    console.log('two');
  });

  console.log('one');

  // one
  // two
  // three
```

## Promise.reject()
`Promise.reject(reason)`方法也会返回一个新的 `Promise` 实例，该实例的状态为`rejected`。











