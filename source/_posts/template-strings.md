title: ES6 template string模板字符串
tags:
  - ES6
  - jstips
categories:
  - javascript
date: 2016-01-27 10:40:12
---
在ES6中，JS除了传统的字符串，提供了新的template string模板字符串。

Ex：普通的字符串:

```
var firstName = 'Jake';
var lastName = 'Rawr';
console.log('My name is ' + firstName + ' ' + lastName);
// My name is Jake Rawr
```

template string模板字符串:
```
var firstName = 'Jake';
var lastName = 'Rawr';
console.log(`My name is ${firstName} ${lastName}`);
// My name is Jake Rawr
```

<!-- more --> 

>注意，注意，template string是用符号 ` 包括起来的，不是单引号也不能是双引号。

我们可以在template string的`${}`中实现
+ 不用`\n`实现多行
+ 简单的逻辑(比如: `1+2`)


同时，我们可以使用一个函数修改template string的输出，这些函数被称为[tagged template strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings#Tagged_template_strings)


你可以通过这边[文章](https://hacks.mozilla.org/2015/05/es6-in-depth-template-strings-2)来了解更多关于template string的内容。

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips