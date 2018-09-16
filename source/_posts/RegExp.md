---
title: RegExp
date: 2018-07-17 16:47:28
tags: [js原生,RegExp]
catetories: [js原生,RegExp]
---
## RegExp对象创建
新建正则表达式有两种方法。一种是使用字面量，以斜杠表示开始和结束。
```javascript
var regx = /xyz/;
```
<!--more-->
另一种是使用RegExp构造函数；RegExp构造函数还可以接受第二个参数，表示修饰符
```javascript
  var regex = new RegExp('xyz');
```
## 实例属性
### 修饰符相关属性
返回一个布尔值，表示对应的修饰符是否设置,三个属性都是只读的。
* `RegExp.prototype.ignoreCase`：返回一个布尔值，表示是否设置了 **i** 修饰符。
* `RegExp.prototype.global`：返回一个布尔值，表示是否设置了 **g** 修饰符。
* `RegExp.prototype.multiline`：返回一个布尔值，表示是否设置了 **m** 修饰符。

### 与修饰符无关的属性
* `RegExp.prototype.lastIndex`：返回一个整数，表示下一次开始搜索的位置。该属性可读写，但是只在进行连续搜索时有意义。
* `RegExp.prototype.source`：返回正则表达式的字符串形式（不包括反斜杠），该属性只读。
```javascript
  var r = /abc/igm;

  r.lastIndex // 0
  r.source // "abc"
```
## 实例方法
### RegExp.prototype.test()
正则实例对象的`test`方法返回一个布尔值，表示当前模式是否能匹配参数字符串。
```javascript
  /cat/.test('cats and dogs') // true
```
如果正则表达式带有g修饰符，则每一次`test`方法都从上一次结束的位置开始向后匹配。
```javascript
  var r = /x/g;
  var s = '_x_x';

  r.lastIndex // 0
  r.test(s) // true

  r.lastIndex // 2
  r.test(s) // true

  r.lastIndex // 4
  // 字符串的第五个位置开始搜索，这个位置是没有字符的，所以返回false。
  r.test(s) // false
```
注意，带有g修饰符时，正则表达式内部会记住上一次的lastIndex属性，这时不应该更换所要匹配的字符串，否则会有一些难以察觉的错误。
```javascript
  var r = /bb/g;
  r.test('bb') // true
  r.test('-bb-') // false 
```
### RegExp.prototype.exec()
正则实例对象的`exec`方法，用来返回匹配结果。如果发现匹配，就返回一个数组，成员是匹配成功的子字符串，否则返回`null`。
```javascript
  var s = '_x_x';
  var r1 = /x/;
  var r2 = /y/;

  r1.exec(s) // ["x"]
  r2.exec(s) // null
```
如果正则表示式包含圆括号（即含有“组匹配”），则返回的数组会包括多个成员。第一个成员是整个匹配成功的结果，后面的成员就是圆括号对应的匹配成功的组。也就是说，第二个成员对应第一个括号，第三个成员对应第二个括号，以此类推。整个数组的length属性等于组匹配的数量再加1。
```javascript
  var s = '_x_x';
  var r = /_(x)/;

  r.exec(s) // ["_x", "x"]
```
`exec`方法的返回数组还包含以下两个属性：

* `input`：整个原字符串。
* `index`：整个模式匹配成功的开始位置（从0开始计数）。

```javascript
  var r = /a(b+)a/;
  var arr = r.exec('_abbba_aba_');
  arr // ["abbba", "bbb"]
  arr.index // 1
  arr.input // "_abbba_aba_"
```
如果正则表达式加上g修饰符，则可以使用多次exec方法，下一次搜索的位置从上一次匹配成功结束的位置开始。
```javascript
  var reg = /a/g;
  var str = 'abc_abc_abc'

  var r1 = reg.exec(str);
  r1 // ["a"]
  r1.index // 0
  reg.lastIndex // 1

  var r2 = reg.exec(str);
  r2 // ["a"]
  r2.index // 4
  reg.lastIndex // 5

  var r3 = reg.exec(str);
  r3 // ["a"]
  r3.index // 8
  reg.lastIndex // 9

  var r4 = reg.exec(str);
  r4 // null
  reg.lastIndex // 0
```
利用g修饰符允许多次匹配的特点，可以用一个循环完成全部匹配。
```javascript
  var reg = /a/g;
  var str = 'abc_abc_abc'

  while(true) {
    var match = reg.exec(str);
    if (!match) break;
    console.log('#' + match.index + ':' + match[0]);
  }
  // #0:a
  // #4:a
  // #8:a
```
## 字符串的实例方法
字符串的实例方法之中，有4种与正则表达式有关。
* `String.prototype.match()`：返回一个数组，成员是所有匹配的子字符串。
* `String.prototype.search()`：按照给定的正则表达式进行搜索，返回一个整数，表示匹配开始的位置。
* `String.prototype.replace()`：按照给定的正则表达式进行替换，返回替换后的字符串。
* `String.prototype.split()`：按照给定规则进行字符串分割，返回一个数组，包含分割后的各个成员。

### String.prototype.match() 
字符串实例对象的`match`方法对字符串进行正则匹配，返回匹配结果。
```javascript
  var s = '_x_x';
  var r1 = /x/;
  var r2 = /y/;

  s.match(r1) // ["x"]
  s.match(r2) // null
```
从上面代码可以看到，字符串的`match`方法与正则对象的`exec`方法非常类似：匹配成功返回一个数组，匹配失败返回`null`。
如果正则表达式带有 **g** 修饰符，则该方法与正则对象的 **exec** 方法行为不同，会一次性返回所有匹配成功的结果。
```javascript
  var s = 'abba';
  var r = /a/g;
  // 会一次性返回所有匹配成功的结果。
  s.match(r) // ["a", "a"]
  r.exec(s) // ["a"]
```
设置正则表达式的`lastIndex`属性，对`match`方法无效，匹配总是从字符串的第一个字符开始。
```javascript
  var r = /a|b/g;
  r.lastIndex = 7;
  'xaxb'.match(r) // ['a', 'b']
  r.lastIndex // 0
```

### String.prototype.search()
字符串对象的`search`方法，返回第一个满足条件的匹配结果在整个字符串中的位置。如果没有任何匹配，则返回`-1`。
```javascript
  '_x_x'.search(/x/)
  // 1
```

### String.prototype.replace()
字符串对象的replace方法可以替换匹配的值。它接受两个参数，第一个是正则表达式，表示搜索模式，第二个是替换的内容。
```javascript
  str.replace(search, replacement)
```
正则表达式如果不加g修饰符，就替换第一个匹配成功的值，否则替换所有匹配成功的值。
```javascript
  'aaa'.replace('a', 'b') // "baa"
  'aaa'.replace(/a/, 'b') // "baa"
  'aaa'.replace(/a/g, 'b') // "bbb"
```
replace方法的一个应用，就是消除字符串首尾两端的空格。
```javascript
  var str = '  #id div.class  ';

  str.replace(/^\s+|\s+$/g, '')
  // "#id div.class"
```
replace方法的第二个参数可以使用美元符号$，用来指代所替换的内容。
* **$&**：匹配的子字符串。
* **$`**：匹配结果前面的文本。
* **$'**：匹配结果后面的文本。
* **$n**：匹配成功的第n组内容，n是从1开始的自然数。
* **$$**: 指代美元符号`$`。
```javascript
  'hello world'.replace(/(\w+)\s(\w+)/, '$2 $1')
  // "world hello"

  'abc'.replace('b', '[$`-$&-$\']')
  // "a[a-b-c]c"

  '3 and 5'.replace(/[0-9]+/g, function (match) {
    return 2 * match;
  })
  // "6 and 10"
```
作为`replace`方法第二个参数的替换函数，可以接受多个参数。其中，第一个参数是捕捉到的内容，第二个参数是捕捉到的组匹配（有多少个组匹配，就有多少个对应的参数）。此外，最后还可以添加两个参数，倒数第二个参数是捕捉到的内容在整个字符串中的位置（比如从第五个位置开始），最后一个参数是原字符串。下面是一个网页模板替换的例子。
```javascript
  var prices = {
    'p1': '$1.99',
    'p2': '$9.99',
    'p3': '$5.00'
  };

  var template = '<span id="p1"></span>'
    + '<span id="p2"></span>'
    + '<span id="p3"></span>';

  template.replace(
    /(<span id=")(.*?)(">)(<\/span>)/g,
    function(match, $1, $2, $3, $4){
      return $1 + $2 + $3 + prices[$2] + $4;
    }
  );
  // "<span id="p1">$1.99</span><span id="p2">$9.99</span><span id="p3">$5.00</span>"
```
### String.prototype.split()
字符串对象的`split`方法按照正则规则分割字符串，返回一个由分割后的各个部分组成的数组。
```javascript
  str.split(separator, [limit]);
```
该方法接受两个参数，第一个参数可以是正则表达式，表示分隔规则，第二个参数是返回数组的最大成员数。
```javascript
  // 非正则分隔
  'a,  b,c, d'.split(',')
  // [ 'a', '  b', 'c', ' d' ]

  // 正则分隔，去除多余的空格
  'a,  b,c, d'.split(/, */)
  // [ 'a', 'b', 'c', 'd' ]

  // 指定返回数组的最大成员
  'a,  b,c, d'.split(/, */, 2)
  [ 'a', 'b' ]
```
## 正则语法
### 字面量字符和元字符
大部分字符在正则表达式中，就是字面的含义，比如/a/匹配a，/b/匹配b。如果在正则表达式之中，某个字符只表示它字面的含义（就像前面的a和b），那么它们就叫做“字面量字符”（literal characters）。
```javascript
  /dog/.test('old dog') // true
```
#### 点字符（.）
点字符（`.`）匹配除回车（`\r`）、换行(`\n`) 、行分隔符（`\u2028`）和段分隔符（`\u2029`）以外的所有字符。注意，对于码点大于`0xFFFF`字符，点字符不能正确匹配，会认为这是两个字符。
#### 位置字符
位置字符用来提示字符所处的位置，主要有两个字符。
* `^` 表示字符串的开始位置
* `$` 表示字符串的结束位置

```javascript
  / test必须出现在开始位置
  /^test/.test('test123') // true

  // test必须出现在结束位置
  /test$/.test('new test') // true

  // 从开始位置到结束位置只有test
  /^test$/.test('test') // true
  /^test$/.test('test test') // false
```

#### 选择符（`|`）
竖线符号（`|`）在正则表达式中表示“或关系”（OR），即`cat|dog`表示匹配`cat`或`dog`。

### 转义符（`\`）
正则表达式中那些有特殊含义的元字符，如果要匹配它们本身，就需要在它们前面要加上反斜杠(`\`)。比如要匹配`+`，就要写成`\+`。
正则表达式中，需要反斜杠转义的，一共有12个字符：`^`、`.`、`[`、`$`、`(`、`)`、`|`、`*`、`+`、`?`、{和\。需要特别注意的是，如果使用`RegExp`方法生成正则对象，转义需要使用**两个斜杠**，因为字符串内部会先转义一次。
```javascript
  /1+1/.test('1+1')
  // false

  /1\+1/.test('1+1')
  // true
```
```javascript
  (new RegExp('1\+1')).test('1+1')
  // false

  (new RegExp('1\\+1')).test('1+1')
  // true
```
上面代码中，`RegExp`作为构造函数，参数是一个字符串。但是，在字符串内部，反斜杠也是转义字符，所以它会先被反斜杠转义一次，然后再被正则表达式转义一次，因此需要两个反斜杠转义。

### 特殊字符
正则表达式对一些不能打印的特殊字符，提供了表达方法。
* `\cX` 表示Ctrl-[X]，其中的X是A-Z之中任一个英文字母，用来匹配控制字符。
* `[\b]` 匹配退格键(U+0008)，不要与\b混淆。
* `\n` 匹配换行键。
* `\r` 匹配回车键。
* `\t` 匹配制表符 tab（`U+0009`）。
* `\v` 匹配垂直制表符（`U+000B`）。
* `\f` 匹配换页符（`U+000C`）。
* `\0` 匹配null字符（`U+0000`）。
* `\xhh` 匹配一个以两位十六进制数（`\x00-\xFF`）表示的字符。
* `\uhhhh` 匹配一个以四位十六进制数（`\u0000-\uFFFF`）表示的 `Unicode` 字符。

### 字符类
字符类表示有一系列字符可供选择，只要匹配其中一个就可以了。所有可供选择的字符都放在方括号内，比如`[xyz]` 表示x、y、z之中任选一个匹配。
### 脱字符（`^`）
如果方括号内的第一个字符是`[^]`，则表示除了字符类之中的字符，其他字符都可以匹配。比如，`[^xyz]`表示除了x、y、z之外都可以匹配。
如果方括号内没有其他字符，即只有`[^]`，就表示匹配一切字符，其中包括换行符。相比之下，点号作为元字符（`.`）是不包括换行符的。
### 连字符（`-`）
某些情况下，对于连续序列的字符，连字符（`-`）用来提供简写形式，表示字符的连续范围。比如，`[abc]`可以写成`[a-c]`，`[0123456789]`可以写成`[0-9]`，同理`[A-Z]`表示26个大写字母。
以下都是合法的字符类简写形式。
```javascript
  [0-9.,]
  [0-9a-fA-F]
  [a-zA-Z0-9-]
  [1-31]
```

### 预定义模式
预定义模式指的是某些常见模式的简写方式。
* `\d` 匹配`0-9`之间的任一数字，相当于`[0-9]`。
* `\D` 匹配所有`0-9以外`的字符，相当于`[^0-9]`。
* `\w` 匹配任意的字母、数字和下划线，相当于`[A-Za-z0-9_]`。
* `\W` 除所有字母、数字和下划线以外的字符，相当于`[^A-Za-z0-9_]`。
* `\s` 匹配空格（包括换行符、制表符、空格符等），相等于`[ \t\r\n\v\f]`。
* `\S` 匹配非空格的字符，相当于`[^ \t\r\n\v\f]`。
* `\b` 匹配词的边界。
* `\B` 匹配非词边界，即在词的内部。

`\b` 和 `\B` 的匹配例子

```javascript
  // \s 的例子
  /\s\w*/.exec('hello world') // [" world"]

  // \b 的例子
  /\bworld/.test('hello world') // true
  /\bworld/.test('hello-world') // true
  /\bworld/.test('helloworld') // false

  // \B 的例子
  /\Bworld/.test('hello-world') // false
  /\Bworld/.test('helloworld') // true
```
通常，正则表达式遇到换行符（\n）就会停止匹配。
```javascript
  var html = "<b>Hello</b>\n<i>world!</i>";

  /.*/.exec(html)[0]
  // "<b>Hello</b>"
```
上面代码中，字符串html包含一个换行符，结果点字符（.）不匹配换行符，导致匹配结果可能不符合原意。这时使用\s字符类，就能包括换行符。
```javascript
  var html = "<b>Hello</b>\n<i>world!</i>";

  /[\S\s]*/.exec(html)[0]
  // "<b>Hello</b>\n<i>world!</i>"
```
上面代码中，`[\S\s]`指代一切字符。

### 重复类
模式的精确匹配次数，使用大括号（`{}`）表示。`{n}`表示恰好重复`n`次，`{n,}`表示至少重复`n`次，`{n,m}`表示重复不少于`n`次，不多于`m`次。
```javascript
  /lo{2}k/.test('look') // true
  /lo{2,5}k/.test('looook') // true
```
### 量词类
量词符用来设定某个模式出现的次数。
* `?` 问号表示某个模式出现0次或1次，等同于`{0, 1}`。
* `*` 星号表示某个模式出现0次或多次，等同于`{0,}`。
* `+` 加号表示某个模式出现1次或多次，等同于`{1,}`。

### 贪婪模式
上一小节的三个量词符，默认情况下都是最大可能匹配，即匹配直到下一个字符不满足匹配规则为止。这被称为贪婪模式。
```javascript
  var s = 'aaa';
  s.match(/a+/) // ["aaa"]
```
如果想将贪婪模式改为非贪婪模式，可以在量词符后面加一个问号。
```javascript
  var s = 'aaa';
  s.match(/a+?/) // ["a"]
```
上面代码中，模式结尾添加了一个问号`/a+?/`，这时就改为非贪婪模式，一旦条件满足，就不再往下匹配。
所以非贪婪模式有：
* `*?`：表示某个模式出现0次或多次，匹配时采用非贪婪模式。
* `+?`：表示某个模式出现1次或多次，匹配时采用非贪婪模式。

### 修饰符
#### g 修饰符
`g`修饰符表示全局匹配（`global`），加上它以后，正则对象将匹配全部符合条件的结果，主要用于搜索和替换。
```javascript
  // 不带 g 每次都是从字符串头部开始匹配
  var regex = /b/;
  var str = 'abba';
  regex.test(str); // true
  regex.test(str); // true
  regex.test(str); // true

  // 含有g修饰符，每次都是从上一次匹配成功的后一位，开始向后匹配
  var regex = /b/g;
  var str = 'abba';
  regex.test(str); // true
  regex.test(str); // true
  regex.test(str); // false
```
#### i 修饰符
默认情况下，正则对象区分字母的大小写，加上`i`修饰符以后表示忽略大小写（`ignorecase`）。
```javascript
  /abc/.test('ABC') // false
  /abc/i.test('ABC') // true
```
#### m 修饰符
`m`修饰符表示多行模式（`multiline`），会修改`^`和`$`的行为。默认情况下（即不加`m`修饰符时），^和$匹配字符串的开始处和结尾处，加上`m`修饰符以后，`^`和`$`还会匹配行首和行尾，即`^`和`$`会识别换行符（`\n`）。
```javascript
  /world$/.test('hello world\n') // false
  /world$/m.test('hello world\n') // true
```

### 组匹配
正则表达式的括号表示分组匹配，括号中的模式可以用来匹配分组的内容。
```javascript
  var m = 'abcabc'.match(/(.)b(.)/);
  m // ['abc', 'a', 'c']
```
注意，使用组匹配时，不宜同时使用g修饰符，否则match方法不会捕获分组的内容。
```javascript
  var m = 'abcabc'.match(/(.)b(.)/g);
  m // ['abc', 'abc']
```
上面代码使用带`g`修饰符的正则表达式，结果match方法只捕获了匹配整个表达式的部分。这时必须使用正则表达式的`exec`方法，配合循环，才能读到每一轮匹配的组捕获。
```javascript
  var str = 'abcabc';
  var reg = /(.)b(.)/g;
  while (true) {
    var result = reg.exec(str);
    if (!result) break;
    console.log(result);
  }
  // ["abc", "a", "c"]
  // ["abc", "a", "c"]
```
正则表达式内部，还可以用\n引用括号匹配的内容，`n`是从`1`开始的自然数，表示对应顺序的括号。
```javascript
  /(.)b(.)\1b\2/.test("abcabc")
  // true

  // \1指向外层括号，\2指向内层括号。
  /y((..)\2)\1/.test('yabababab') // true
```
#### 非捕获组
`?:x` 非捕获组，表示不返回该组匹配的内容，即匹配的结果中不计入这个括号。
```javascript
  var m = 'abc'.match(/(?:.)b(.)/);
  m // ["abc", "c"]

  // 正常匹配
  var url = /(http|ftp):\/\/([^/\r\n]+)(\/[^\r\n]*)?/;

  url.exec('http://google.com/');
  // ["http://google.com/", "http", "google.com", "/"]

  // 非捕获组匹配
  var url = /(?:http|ftp):\/\/([^/\r\n]+)(\/[^\r\n]*)?/;

  url.exec('http://google.com/');
  // ["http://google.com/", "google.com", "/"]
```
#### 先行断言
`x(?=y)`称为先行断言（Positive look-ahead），`x`只有在y前面才匹配，`y`不会被计入返回结果。比如，要匹配后面跟着百分号的数字，可以写成`/\d+(?=%)/`。
```javascript
  var m = 'abc'.match(/b(?=c)/);
  m // ["b"]
```
#### 先行否定断言
`x(?!y)`称为先行否定断言（Negative look-ahead），`x`只有不在`y`前面才匹配，y不会被计入返回结果。比如，要匹配后面跟的不是百分号的数字，就要写成`/\d+(?!%)/`
```javascript
  /\d+(?!\.)/.exec('3.14')
  // ["14"]
```


