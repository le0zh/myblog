title: es6中的箭头函数
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-03-09 17:40:12
---

作为ES6的一个新特性，箭头函数(fat arrow function)作为一种可以方便的可以写更少的代码的语法出现了。它的名字来源于它的语法`=>`，就是一个“大箭头”（和“小箭头”`->`相比）。一些开发者或许已经从其他变成语言中遇到了这种类型的函数，比如在Haskell中，被称为“lamba表达式”，或者“匿名函数”。它被之所以被称为匿名函数，是因为这中箭头函数并没有一个显示声明的函数名。

<!-- more -->

### 有什么好处？
+ 语法：更少的代码行；再也不用一次次的输入`function` 关键字了
+ 语义：从周围的上下文环境中捕获 `this` 

### 简单的例子
看下下面两个代码片段的例子，他们的作用是一样的，然后你将会快速的明白箭头函数是干什么用的。

```
// 箭头函数的基本语法
param => expression

// 还可以加上括号
// 当有多于一个参数的时候，括号是必须的
(param1 [, param2]) => expression

// 使用普通函数的形式
var arr = [5,3,2,9,1];
var arrFunc = arr.map(function(x) {
  return x * x;
});
console.log(arr)

// 使用箭头函数的形式
var arr = [5,3,2,9,1];
var arrFunc = arr.map((x) => x*x);
console.log(arr)
```

我们可以看到，这种情况下箭头函数的形式我们就不用写大括号和函数的返回值，从而节省时间。这里建议总是将参数写在小括号里，因为当有多个参数的情况下，小括号是必须的，比如`(x,y) => x+y`，这仅仅是为了保持一致，防止一些情况下忘记了。但是上面的代码同样可以写成`x => x*x` 。目前来说，这仅仅是语法上的改进，可以少些一些代码同时提高可读性。

### Lexically binding this
另外还有一个很好的理由来使用箭头函数。关于`this` 的上下文环境总是有很多问题。使用箭头函数，你再也不用考虑`.bind(this)` 或者设置 `that = this` 了，箭头函数将从当前的lexical 记录当前上下文的`this`。看下面的例子：

```
// 全局地定义 this.i
this.i = 100;

var counterA = new CounterA();
var counterB = new CounterB();
var counterC = new CounterC();
var counterD = new CounterD()

// 不好的例子
function CounterA() {
  // 这里的this指向CounterA的实例
  this.i = 0;
  setInterval(function () {
    // 这里的this指向全局对象，而不是CounterA的实例
    // 因此该计数器将从100开始计数，而不是0（局部变量this.i）
    this.i++;
    document.getElementById("counterA").innerHTML = this.i;
  }, 500);
}

// 手动的绑定that = this
function CounterB() {
  this.i = 0;
  var that = this;
  setInterval(function() {
	//可以从0开始计数
    that.i++;
    document.getElementById("counterB").innerHTML = that.i;
  }, 500);
}

// 使用.bind(this)
function CounterC() {
  this.i = 0;
  setInterval(function() {
    this.i++;
    document.getElementById("counterC").innerHTML = this.i;
  }.bind(this), 500);
}

// 使用箭头函数
function CounterD() {
  this.i = 0;
  setInterval(() => {
    this.i++;
    document.getElementById("counterD").innerHTML = this.i;
  }, 500);
}
```

 关于箭头函数的更多内容，请参考文档 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)