---
title: Symbol
date: 2018-07-22 22:20:57
tags: [js原生,ES6,Symbol]
categories: [js原生,ES6,Symbol]
---
`ES5` 的对象属性名都是字符串，这容易造成属性名的冲突。比如，你使用了一个他人提供的对象，但又想为这个对象添加新的方法（`mixin` 模式），新方法的名字就有可能与现有方法产生冲突。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是 `ES6` 引入`Symbol`的原因。

`ES6` 引入了一种新的原始数据类型`Symbol`，表示独一无二的值。它是 `JavaScript` 语言的第七种数据类型，前六种是：`undefined`、`null`、布尔值（`Boolean`）、字符串（`String`）、数值（`Number`）、对象（`Object`）;凡是属性名属于 `Symbol` 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。

用法：
```JavaScript
  let s = Symbol();
  typeof s // "symbol"

```

**注意**，`Symbol` 函数前不能使用 `new` 命令，否则会报错。这是因为生成的 `Symbol` 是一个**原始类型**的值，不是对象。

`Symbol` 函数可以接受一个字符串作为参数，表示对 `Symbol` 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。
```JavaScript
  let s1 = Symbol('foo');
  let s2 = Symbol('bar');
  s1 // Symbol(foo)
  s2 // Symbol(bar)

  s1.toString() // "Symbol(foo)"
  s2.toString() // "Symbol(bar)"
```
如果 `Symbol` 的参数是一个对象，就会调用该对象的 `toString` 方法，将其转为字符串，然后才生成一个 `Symbol` 值。
```JavaScript
  Symbol({}) // Symbol([object Object])

  const obj = {
    toString() {
      return 'abc';
    }
  };
  const sym = Symbol(obj);
  sym // Symbol(abc)
```

**注意**，`Symbol` 函数的参数只是表示对当前 `Symbol` 值的描述，因此相同参数的`Symbol`函数的返回值是不相等的。

`Symbol` 值不能与其他类型的值进行运算，会报错。
```JavaScript
  let sym = Symbol('My symbol');

  "your symbol is " + sym
  // TypeError: can't convert symbol to string
  `your symbol is ${sym}`
  // TypeError: can't convert symbol to string
```

但是，`Symbol` 值可以显式转为字符串。
```JavaScript
  let sym = Symbol('My symbol');

  String(sym) // 'Symbol(My symbol)'
  sym.toString() // 'Symbol(My symbol)'
```

## 作为属性名的 Symbol
```JavaScript
  let mySymbol = Symbol();

  // 第一种写法
  let a = {};
  a[mySymbol] = 'Hello!';

  // 第二种写法
  let a = {
    [mySymbol]: 'Hello!'
  };

  // 第三种写法
  let a = {};
  Object.defineProperty(a, mySymbol, { value: 'Hello!' });

  // 以上写法都得到同样结果
  a[mySymbol] // "Hello!"
```
**注意**，`Symbol` 值作为对象属性名时，不能用点运算符。

`Symbol` 类型还可以用于定义一组常量，保证这组常量的值都是不相等的。
```JavaScript
  const log = {};

  log.levels = {
    DEBUG: Symbol('debug'),
    INFO: Symbol('info'),
    WARN: Symbol('warn')
  };
  console.log(log.levels.DEBUG, 'debug message');
  console.log(log.levels.INFO, 'info message');
```
## 属性名的遍历
`Symbol` 作为属性名，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、JSON.stringify()返回。但是，它也不是私有属性，有一个`Object.getOwnPropertySymbols`方法，可以获取指定对象的所有 S`ymbol` 属性名。

另一个新的 `API`，`Reflect.ownKeys`方法可以返回所有类型的键名，包括常规键名和 `Symbol` 键名。

由于以 `Symbol` 值作为名称的属性，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。
```JavaScript
  let size = Symbol('size');

  class Collection {
    constructor() {
      this[size] = 0;
    }

    add(item) {
      this[this[size]] = item;
      this[size]++;
    }

    static sizeOf(instance) {
      return instance[size];
    }
  }

  let x = new Collection();
  Collection.sizeOf(x) // 0

  x.add('foo');
  Collection.sizeOf(x) // 1

  Object.keys(x) // ['0']
  Object.getOwnPropertyNames(x) // ['0']
  Object.getOwnPropertySymbols(x) // [Symbol(size)]
```













