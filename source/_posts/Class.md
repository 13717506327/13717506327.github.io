---
title: Class
date: 2018-07-20 21:24:04
tags: [js原生, 面向对象编程, Class, ES6]
categories: [js原生, 面向对象编程, Class]
---
`ES6` 提供了更接近传统语言的写法，引入了 `Class`（类）这个概念，作为对象的模板。通过class关键字，可以定义类。

基本上，`ES6` 的`class`可以看作只是一个语法糖，它的绝大部分功能，`ES5` 都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。上面的代码用 `ES6` 的`class`改写，就是下面这样。

```javascript
  //定义类
  class Point {
    constructor(x, y) {
      this.x = x;
      this.y = y;
    }

    toString() {
      return '(' + this.x + ', ' + this.y + ')';
    }
  }
```
上面代码定义了一个“类”，可以看到里面有一个`constructor`方法，这就是构造方法，而`this`关键字则代表实例对象。也就是说，`ES5` 的构造函数Point，对应 `ES6` 的`Point`类的构造方法。

`Point`类除了构造方法，还定义了一个`toString`方法。注意，定义“类”的方法的时候，前面不需要加上`function`这个关键字，直接把函数定义放进去了就可以了。另外，方法之间**不需要逗号**分隔，加了会报错。

`ES6` 的类，完全可以看作构造函数的另一种写法。

```javascript
  class Point {
    // ...
  }

  typeof Point // "function"
  Point === Point.prototype.constructor // true
```

`ES6` 的类，完全可以看作构造函数的另一种写法。

```javascript
  class Point {
    // ...
  }

  typeof Point // "function"
  Point === Point.prototype.constructor // true
```
上面代码表明，类的数据类型就是函数，类本身就指向构造函数。

使用的时候，也是直接对类使用`new`命令，跟构造函数的用法完全一致。

```javascript
  class Bar {
    doStuff() {
      console.log('stuff');
    }
  }

  var b = new Bar();
  b.doStuff() // "stuff"
```
构造函数的`prototype`属性，在 `ES6` 的“类”上面继续存在。事实上，类的所有方法都定义在类的`prototype`属性上面。
```javascript
  class Point {
    constructor() {
      // ...
    }

    toString() {
      // ...
    }

    toValue() {
      // ...
    }
  }

  // 等同于

  Point.prototype = {
    constructor() {},
    toString() {},
    toValue() {},
  };
```
由于类的方法都定义在`prototype`对象上面，所以类的新方法可以添加在`prototype`对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。
```javascript
  class Point {
    constructor(){
      // ...
    }
  }

  Object.assign(Point.prototype, {
    toString(){},
    toValue(){}
  });
```

## constructor方法
`constructor`方法是类的默认方法，通过`new`命令生成对象实例时，自动调用该方法。一个类必须有`constructor`方法，如果没有显式定义，一个空的`constructor`方法会被默认添加。
```javascript
  class Point {
  }

  // 等同于
  class Point {
    constructor() {}
  }
```

`constructor` 方法默认返回实例对象（即 `this` ），完全可以指定返回另外一个对象。
```javascript
  class Foo {
    constructor() {
      return Object.create(null);
    }
  }

  new Foo() instanceof Foo
  // false
```
上面代码中，`constructor`函数返回一个全新的对象，结果导致实例对象不是`Foo`类的实例。


## 类的实例对象
生成类的实例对象的写法，与 `ES5` 完全一样，也是使用 `new` 命令。前面说过，如果忘记加上 `new`，像函数那样调用`Class`，将会报错。

与 `ES5` 一样，实例的属性除非显式定义在其本身（即定义在 `this` 对象上），否则都是定义在原型上（即定义在 `class` 上）。
```javascript
  //定义类
  class Point {

    constructor(x, y) {
      this.x = x;
      this.y = y;
    }

    toString() {
      return '(' + this.x + ', ' + this.y + ')';
    }

  }

  var point = new Point(2, 3);

  point.toString() // (2, 3)

  point.hasOwnProperty('x') // true
  point.hasOwnProperty('y') // true
  point.hasOwnProperty('toString') // false
  point.__proto__.hasOwnProperty('toString') // true
```
上面代码中，`x`和`y`都是实例对象`point`自身的属性（因为定义在`this`变量上），所以`hasOwnProperty`方法返回`true`，而`toString`是原型对象的属性（因为定义在`Point`类上），所以`hasOwnProperty`方法返回`false`。这些都与 `ES5` 的行为保持一致。

与 `ES5` 一样，类的所有实例共享一个原型对象。
```javascript
  var p1 = new Point(2,3);
  var p2 = new Point(3,2);

  p1.__proto__ === p2.__proto__
  //true
```
上面代码中，`p1`和`p2`都是`Point`的实例，它们的原型都是`Point.prototype`，所以`__proto__`属性是相等的。

> `__proto__` 并不是语言本身的特性，这是各大厂商具体实现时添加的私有属性，虽然目前很多现代浏览器的 JS 引擎中都提供了这个私有属性，但依旧不建议在生产中使用该属性，避免对环境产生依赖。生产环境中，我们可以使用 `Object.getPrototypeOf` 方法来获取实例对象的原型，然后再来为原型添加方法/属性。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
```
使用实例的`__proto__`属性改写原型，必须相当谨慎，不推荐使用，因为这会改变“类”的原始定义，影响到所有实例。

## Class表达式
与函数一样，类也可以使用表达式的形式定义。
```javascript
  const MyClass = class Me {
    getClassName() {
      return Me.name;
    }
  };
```
上面代码使用表达式定义了一个类。需要注意的是，这个类的名字是`MyClass`而不是`Me`，`Me`只在 `Class` 的内部代码可用，指代当前类。

如果类的内部没用到的话，可以省略`Me`，也就是可以写成下面的形式。
```javascript
  const MyClass = class { /* ... */ };
```
## 不存在变量提升
类不存在变量提升（hoist），这一点与 ES5 完全不同。

## 私有方法和私有属性

**私有方法是常见需求，但 ES6 不提供，只能通过变通方法模拟实现。**

一种是在命名上加以区别
```javascript
  class Widget {

    // 公有方法
    foo (baz) {
      this._bar(baz);
    }

    // 私有方法
    _bar(baz) {
      return this.snaf = baz;
    }

    // ...
  }
```
这种命名是不保险的，在类的外部，还是可以调用到这个方法。
```javascript
  class Widget {
    foo (baz) {
      bar.call(this, baz);
    }

    // ...
  }

  function bar(baz) {
    return this.snaf = baz;
  }
```
还有一种方法是利用`Symbol`值的唯一性，将私有方法的名字命名为一个`Symbol`值。
```javascript
  const bar = Symbol('bar');
  const snaf = Symbol('snaf');

  export default class myClass{

    // 公有方法
    foo(baz) {
      this[bar](baz);
    }

    // 私有方法
    [bar](baz) {
      return this[snaf] = baz;
    }

    // ...
  };
```
## Class 的 Generator方法
如果某个方法之前加上星号（`*`），就表示该方法是一个 `Generator` 函数。
```javascript
  class Foo {
    constructor(...args) {
      this.args = args;
    }
    * [Symbol.iterator]() {
      for (let arg of this.args) {
        yield arg;
      }
    }
  }

  for (let x of new Foo('hello', 'world')) {
    console.log(x);
  }
  // hello
  // world
```
上面代码中，`Foo`类的`Symbol.iterator`方法前有一个星号，表示该方法是一个 `Generator` 函数。`Symbol.iterator`方法返回一个Foo类的默认遍历器，`for...of`循环会自动调用这个遍历器。

## class的 静态方法
类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上`static`关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

```javascript
  class Foo {
    static classMethod() {
      return 'hello';
    }
  }

  Foo.classMethod() // 'hello'

  var foo = new Foo();
  foo.classMethod()
  // TypeError: foo.classMethod is not a function
```
注意，如果静态方法包含this关键字，这个`this`指的是类，而不是实例。

父类的静态方法，可以被子类继承。
```javascript
  class Foo {
    static classMethod() {
      return 'hello';
    }
  }

  class Bar extends Foo {
  }

  Bar.classMethod() // 'hello'
```
静态方法也是可以从`super`对象上调用的。
```javascript
  class Foo {
    static classMethod() {
      return 'hello';
    }
  }

  class Bar extends Foo {
    static classMethod() {
      return super.classMethod() + ', too';
    }
  }

  Bar.classMethod() // "hello, too"
```
目前，只有这种写法可行，因为 `ES6` 明确规定，`Class` 内部只有静态方法，没有静态属性。
```javascript
  class Foo {
  }
  Foo.prop = 1;
  Foo.prop // 1

  // 以下两种写法都无效
  class Foo {
    // 写法一
    prop: 2

    // 写法二
    static prop: 2
  }

  Foo.prop // undefined
```
## new.target 属性
`new`是从构造函数生成实例对象的命令。`ES6` 为`new`命令引入了一个`new.target`属性，该属性一般用在构造函数之中，返回`new`命令作用于的那个构造函数。如果构造函数不是通过`new`命令调用的，`new.target`会返回`undefined`，因此这个属性可以用来确定构造函数是怎么调用的。
```javascript
  function Person(name) {
    if (new.target !== undefined) {
      this.name = name;
    } else {
      throw new Error('必须使用 new 命令生成实例');
    }
  }

  // 另一种写法
  function Person(name) {
    if (new.target === Person) {
      this.name = name;
    } else {
      throw new Error('必须使用 new 命令生成实例');
    }
  }

  var person = new Person('张三'); // 正确
  var notAPerson = Person.call(person, '张三');  // 报错
```

## Class的继承
`Class` 可以通过 `extends` 关键字实现继承，这比 `ES5` 的通过修改原型链实现继承，要清晰和方便很多。
```javascript
class Point {
}

class ColorPoint extends Point {
}
```
子类必须在`constructor`方法中调用`super`方法，否则新建实例时会报错。这是因为子类自己的`this`对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用`super`方法，子类就得不到`this`对象。
```javascript
  class Point { /* ... */ }

  class ColorPoint extends Point {
    constructor() {
    }
  }

  let cp = new ColorPoint(); // ReferenceError
```
只有调用`super`之后，才可以使用`this`关键字，否则会报错。

## super 关键字
`super`这个关键字，既可以当作函数使用，也可以当作对象使用。在这两种情况下，它的用法完全不同。

第一种情况，`super`作为函数调用时，代表父类的构造函数。`ES6` 要求，子类的构造函数必须执行一次`super`函数。
第二种情况，`super`作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。
`ES6` 规定，在子类普通方法中通过`super`调用父类的方法时，方法内部的`this`指向当前的子类实例。

如果`super`作为对象，用在静态方法之中，这时`super`将指向父类，而不是父类的原型对象。
```javascript
  class Parent {
    static myMethod(msg) {
      console.log('static', msg);
    }

    myMethod(msg) {
      console.log('instance', msg);
    }
  }

  class Child extends Parent {
    static myMethod(msg) {
      super.myMethod(msg);
    }

    myMethod(msg) {
      super.myMethod(msg);
    }
  }

  Child.myMethod(1); // static 1

  var child = new Child();
  child.myMethod(2); // instance 2
```
## prototype 和  __proto__
大多数浏览器的 ES5 实现之中，每一个对象都有`__proto__`属性，指向对应的构造函数的`prototype`属性。`Class` 作为构造函数的语法糖，同时有`prototype`属性和`__proto__`属性，因此同时存在两条继承链。
1. 子类的__proto__属性，表示构造函数的继承，总是指向父类。
2. 子类prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性。
```javascript
  class A {
  }

  class B {
  }

  // B 的实例继承 A 的实例
  Object.setPrototypeOf(B.prototype, A.prototype);

  // B 继承 A 的静态属性
  Object.setPrototypeOf(B, A);

  const b = new B();

  Object.setPrototypeOf(B.prototype, A.prototype);
  // 等同于
  B.prototype.__proto__ = A.prototype;

  Object.setPrototypeOf(B, A);
  // 等同于
  B.__proto__ = A;
```


