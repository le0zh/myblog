title: (#16) 向回调函数中传入参数
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-04-07 09:03:12
---
默认来说，我们不能直接向回调函数中传入参数，比如：

```
function callback() {
  console.log('Hi human');
}

document.getElementById('someelem').addEventListener('click', callback);
```
但是，可以使用Javascript中的闭包作用域来向回调函数中传入参数，向下面这样:

```
function callback(a, b) {
  return function() {
    console.log('sum = ', (a+b));
  }
}

var x = 1, y = 2;
document.getElementById('someelem').addEventListener('click', callback(x, y));
```
<!-- more -->

#### 什么是闭包？

闭包是指向一些独立变量的函数，换句话说，定义在闭包中的函数可以“记住”被创建时的上下文环境。 
具体内容可以查阅[MDN 文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)。


所以呢，当回调函数被调用时，参数`x`和`y`是在他的作用域中的。

实现这种模式的另一种方式是使用`bind`方法，比如：

```
var alertText = function(text) {
  alert(text);
};

document.getElementById('someelem').addEventListener('click', alertText.bind(this, 'hello'));
```

两种方法之间有些细微的性能差别，具体请参看[jsperf](http://jsperf.com/bind-vs-closure-23)。

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips