title: 在ES6函数中模拟必选参数
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-01-13 10:40:12
---

在多数编程语言中，函数的参数默认都是必选的，开发者必选要显示的声明参数的可选项。
在js中，所有的参数都是可选的。

但是我们可以利用[ES6中的参数默认值](http://exploringjs.com/es6/ch_parameter-handling.html#sec_parameter-default-values)的特性，在不改变函数体的情况下确保参数的必选性。

上代码：
```
 const _err = function( message ){
   throw new Error( message );
 }

 const getSum = (a = _err('a is not defined'), b = _err('b is not defined')) => a + b

getSum( 10 ) // throws Error, b is not defined
getSum( undefined, 10 ) // throws Error, a is not define
```

`_err`是一个立即抛出异常的函数，当没有给函数中的一个参数赋值时，默认值将被使用，所以`_err`会被调用同时抛出一个异常。

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips