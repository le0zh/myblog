title: 你可能不知道的try...finally
tags:
  - javascript
  - you-dont-know-js
categories:
  - javascript
date: 2017-06-26 17:02:12
---

> 你(可能)不知道的系列，经典的you-dont-know-js读书笔记。

## 综述

在`finally`分支的代码，无论如何都将被执行，并且是在`try` `catch`（如果有的话）完成后，在其他代码运行之前，立即执行。

在某种意义上，我们可以将在`finally`分支中的代码想象为一个回调函数，不管其他部分如何，这个回调函数总是会被执行。

## 特殊情况

上面综述大家一般都知道，但是在特殊情况下的执行流程呢？

> 这里指的特殊情况是指，存在改变函数执行流程的语句，比如`return` `throw`等。

<!-- more -->

### 在`try`分支中有`return`

如果在`try`分支中有`return`语句的话，会怎么样呢？显然最后函数还是要返回一个值的。问题是：`fianlly`分支中的代码何时执行？

```js
function foo() {
    try {
        return 42;
    }
    finally {
        console.log( "Hello" );
    }

    console.log( "never runs" );
}

console.log( foo() );
// Hello
// 42
```

首先`return 42`会被立即的执行，它的作用是**设置**了调用`foo()`得到的返回值（*注意：并没有结束函数执行*）。这个动作结束了`try`分支，然后`finally`分支会立即接着执行，`finally`分支执行完后，`foo()`函数调用才算完成。所以`42`会被作为返回值给`console.log(...)`使用，最后打印出42。

### 在`try`分支中有`throw`

对于`try`中的`throw`语句，是同样的流程：

```js
function foo() {
  try {
      throw 42;
  }
  finally {
      console.log( "Hello" );
  }

  console.log( "never runs" );
}

console.log( foo() );
// Hello
// Uncaught Exception: 42
```

### 在`finally`分支中有`throw`

现在，如果在`finally`内部被抛出了一个异常（意外的或者故意的）的话，将会直接结束整个函数，同时如果在`try`分支中有`return xxx`语句设置了返回值，这个值将会被抛弃：

```js
function foo() {
    try {
        return 42;
    }
    finally {
        throw "Oops!";
    }

    console.log( "never runs" );
}

console.log( foo() );
// Uncaught Exception: Oops!
```

### 在`finally`分支有`return`

在`finally`中显示调用`return`语句的话，可以重写`try`或者`catch`分支中的`return`语句：

```js
function foo() {
    try {
        return 42;
    }
    finally {
        // no `return ..` here, so no override
    }
}

function bar() {
    try {
        return 42;
    }
    finally {
        // override previous `return 42`
        return;
    }
}

function baz() {
    try {
        return 42;
    }
    finally {
        // override previous `return 42`
        return "Hello";
    }
}

foo();    // 42
bar();    // undefined
baz();    // "Hello"
```

正常来说，如果函数中省略了`return`语句，那么最后的返回值将是`undefined`，但是在`finally`块中， 省略`return`并不等同于`return undefined`，如果没有`return`，则仅仅就是让前面的`return`生效。
