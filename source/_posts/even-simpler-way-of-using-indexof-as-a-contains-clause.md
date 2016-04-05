title: (#15) 利用IndexOf判断字符串是否包含时，更简单的判断使用方式
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-04-05 12:28:12
---

JavaScript中默认是没有判断字符串是否被包含的`contains`方法。当我们需要检测一个子字符串是否被另外一个字符串包含时，通常使用下面的方法:

```
var someText = 'javascript rules';
if (someText.indexOf('javascript') !== -1) {
  // 包含
}

// 或者
if (someText.indexOf('javascript') >= 0) {
  // 包含
}
```

但是我们看一下[Express](https://github.com/strongloop/express)中的一些代码片段:

<!-- more -->

[examples/mvc/lib/boot.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/examples/mvc/lib/boot.js#L26)

```
for (var key in obj) {
  // "reserved" exports
  if (~['name', 'prefix', 'engine', 'before'].indexOf(key)) continue;
```

[lib/utils.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/lib/utils.js#L93)

```
exports.normalizeType = function(type){
  return ~type.indexOf('/')
    ? acceptParams(type)
    : { value: mime.lookup(type), params: {} };
};
```

[examples/web-service/index.js](https://github.com/strongloop/express/blob/2f8ac6726fa20ab5b4a05c112c886752868ac8ce/examples/web-service/index.js#L35)

```
// key is invalid
if (!~apiKeys.indexOf(key)) return next(error(401, 'invalid api key'));
```

其中的秘密就在于[按位运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) `~` "Bitwise operators perform their operations on binary representations, but they return standard JavaScript numerical values." (按位运算符是将操作对象转换为二进制，在此基础上进行运算，但是**运算的结果是标准的JavaScript数字类型**)

**按位非运算会把-1转换为0， 而在JavaScript中，0可以被规范化为`false`:**

```
var someText = 'text';
!!~someText.indexOf('tex'); // someText contains "tex" - true
!~someText.indexOf('tex'); // someText NOT contains "tex" - false
~someText.indexOf('asd'); // someText doesn't contain "asd" - false
~someText.indexOf('ext'); // someText contains "ext" - true
```

#### String.prototype.includes()
ES6中引入了新的方法[`includes()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/includes)，可以用来检测一个字符串是否包含另外一个字符串。

```
'something'.includes('thing'); // true
```

在ECMAScript 2016 (ES7)中甚至可以将这项技术使用在数组中：

```
!!~[1, 2, 3].indexOf(1); // true
[1, 2, 3].includes(1); // true
```

**不幸的是，这仅仅在Chrome, Firefox, Safari 9+ 和 Edge中被支持， IE11- 都不支持。最好使用在受控的环境中**

>译注： 上面是指浏览器原生的对ES6的支持，可以使用Babel等转换工具将ES6代码转换为ES5的。

---

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips