title: 刷些js coding面试题
tags:
  - javascript
categories:
  - javascript
date: 2016-04-22 15:04:12
---

###  问题：foo的值是什么
```
var foo = 10 + '20';
```

> 1020, typeof test = 'string'

扩展：
```
var foo= 20 - '10';
```

> 10, typeof test = 'number'

<!-- more -->

### 问题：如何实现以下函数？
```
add(2, 5); // 7
add(2)(5); // 7
```

```
var add = function add(x, y) {
	if (typeof y === 'undefined') {
		return function(y) {
			return x + y;
		}
	}

	return x + y;
}
```

### 问题：下面的语句的返回值是什么？？
```
var statement1 = "i'm a lasagna hog".split("").reverse().join("");
```

>  goh angasal a m'i

**注意！！split传入的是空字符，不是空格。这样会把每个字符分割开，这样可以巧妙的完成整个字符串的倒置**

```
 "i'm a lasagna hog".split("") = ["i", "'", "m", " ", "a", " ", "l", "a", "s", "a", "g", "n", "a", " ", "h", "o", "g"]
```

###  问题：window.foo1的值是什么？
```
(window.foo1 || (window.foo1 = "bar"));
```
> bar

###  问题：下面两个 alert 的结果是什么？？

```
var foo = "Hello";
(function() {
	var bar = " World";
	alert(foo + bar);
})();
alert(foo + bar);
```

>     1. Hello World
    2.  Uncaught ReferenceError: bar is not defined

### 问题：foo2.length的值是什么?

```
var foo2 = [];
foo2.push(1);
foo2.push(2);
```

> 2

### 问题：foo.x的值是什么？

```
var foo = { n: 1 };
var bar = foo;
foo.x = foo3= { n: 2 };
```

> undefined,  foo3: { n: 2 } 连等号的左结合律

### 问题：下面代码的输出是什么？
```
console.log('one');
setTimeout(function() {
	console.log('two');
}, 0);
console.log('three');
```

> one three two

