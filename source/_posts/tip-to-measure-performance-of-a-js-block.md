title: 测量js代码块性能的小技巧
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-01-20 10:40:12
---

为了能据快速的测量javascript代码块的性能，我们可以使用`console`的两个函数：
[console.time(label)](https://developer.chrome.com/devtools/docs/console-api#consoletimelabel)
[console.timeEnd(label)](https://developer.chrome.com/devtools/docs/console-api#consoletimeendlabel)

<!-- more --> 

示例代码:
```
console.time('Array initialize');
var arr = new Array(100);
var len = arr.length;
var i = 0;

for (i = 0; i < len; i++) {
    arr[i] = new Object();
}
console.timeEnd('Array initialize');

// Outputs: Array initialize: 0.711ms
```

更多参考: [Console object](https://github.com/DeveloperToolsWG/console-object) [Javascript benchmarking](https://mathiasbynens.be/notes/javascript-benchmarking)

Demo: [jsfiddle](https://jsfiddle.net/meottb62/) [codepen](http://codepen.io/anon/pen/JGJPoa)

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips

