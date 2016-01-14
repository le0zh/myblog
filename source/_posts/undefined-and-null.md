title: undefined和null的区别
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-01-06 10:40:12
---

+ `undefined`是说一个变量没有被声明，或者被声明了但是还没有被赋值

+ `null`是一个特定值(an assignment value )，代表“没有值”(no value)

+ JavaScript会将没有被赋值的变量默认设为`undefined`

+ JavaScript绝对不自动将一个变量设置为`null`值，也就是说`null`都是程序员手动设置用来说明一个变量没有值的
+ `undefined`的类型（typeof）是`undefined`
+ `null`的类型（typeof）是`object`
+ 两个都是基本数据类型

<!-- more -->

+ 判断一个变量是`undefined`的方法：
```
typeof(variable) === 'undefined'
```

+ 判断一个变量是`null`的方法:
```
variable === null
```

+ 相等性：(The equility operator consider them equal, but the identity doesn't):
```
null == undefined // true
null === undefined  //false
```

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips