title: 写一个尽可能完善的Function.prototype.bind方法
tags:
  - javascript
categories:
  - javascript
date: 2017-06-26 17:02:12
---

首先，官方的文档说明在[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)


### 语法

```js
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

<!--more-->

### 用法和注意点

#### 1. 通过`bind`改变函数中的`this`指向，这是最常见的用法

```js
function foo(something) {
  this.a = something;
}

var obj1 = {};
var bar = foo.bind(obj1);

bar(2);

console.log(obj1.a); // 2
```

#### 2. 偏函数，使一个函数拥有预设的初始参数

```js
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// Create a function with a preset leading argument
var leadingThirtysevenList = list.bind(undefined, 37);

var list2 = leadingThirtysevenList(); // [37]
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
```

#### 3. 和`new`操作符结合使用

```js
function foo(something) {
  this.a = something;
}

var obj1 = {};

var bar = foo.bind(obj1);

bar(2);
console.log(obj1.a); // 2

// 使用new操作符调用
var baz = new bar(3);
console.log(obj1.a); // 2
console.log(baz.a); // 3
```

当`bind`出来的函数使用`new`操作符调用时，**就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数**

### 开始写polyfills

首先写一个架子：

```js
Function.prototype.bind = function(obj) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
  }
};

```

这里是在`Function.proptotype`原型链上增加了一个`bind`方法，先保证了`this`的类型必须是函数类型。


第一步，满足功能1： 通过`bind`改变函数中的`this`指向

```js
Function.prototype.bind = function(obj) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
  }

  var func = this;

  return function fBound() {
    return func.apply(obj, arguments);
  }
};

```

这时候，功能1就满足了，但是功能2是不能满足的，因为我们在`bind`方法中，仅仅接收了处理了第一个参数，而其他的参数都被抛弃了。


第二步，为了满足偏函数的功能，我们需要在`bind`内部，将除了第一个参数之外的其他参数，合并到最后返回的`fBound`函数中。

代码如下：

```js
Function.prototype.bind = function(obj) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
  }

  var func = this;
  var args = Array.prototype.slice.call(arguments, 1); // 拿到bind函数调用时，除了第一个参数之外的剩余参数

  return function fBound() {
    // 注意，这里将bind的剩余参数和当前的arguments合并了
    return func.apply(obj, args.concat(Array.prototype.slice.call(arguments)));
  }
};

```

第三步，处理`new`操作符调用的情况：

在判断到使用`new`操作符调用时，我们直接忽略`bind`提供的this，而使用函数本身的this即可。

```js
Function.prototype.bind = function(obj) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
  }

  var func = this;
  var args = Array.prototype.slice.call(arguments, 1); // 拿到bind函数调用时，除了第一个参数之外的剩余参数

  function fNop() {}

  function fBound() {
    // 通过this instanceof fNop判断是否为new，而之所以能这么判断，是因为下面的prototype指定语句
    // 注意，这里将bind的剩余参数和当前的arguments合并了
    return func.apply(this instanceof fNop ? this : obj, args.concat(Array.prototype.slice.call(arguments)));
  }

  fNop.prototype = this.prototype;
  fBound.prototype = new fNop();

  return fBound;
};

```

如果支持ES6的话，我们在函数内部可以直接通过判断`new.target`是否存在就能知道是否为`new`调用，代码会简化很多：

```js
Function.prototype.bind = function(obj) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
  }

  var func = this;
  var args = Array.prototype.slice.call(arguments, 1); // 拿到bind函数调用时，除了第一个参数之外的剩余参数

  return function fBound() {
    // 通过new.target判断是否为new，ES6新增
    // 注意，这里将bind的剩余参数和当前的arguments合并了
    return func.apply(new.target ? this : obj, args.concat(Array.prototype.slice.call(arguments)));
  }
};
```

这时候，我们写的polyfill已经基本满足所有需求了。
