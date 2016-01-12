title: javascript中的Hoisting(变量提升)(吊装)
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-01-12 10:40:12
---

理解hoisting(变量提升)能帮助你更好的组织你的函数作用域。

**要时刻记住，变量的声明和函数定义会被提升(hoisting)到函数作用域的顶部**

变量的定义(definition)不会被提升，即使你是在同一行同时声明和定义一个变量。

变量声明(declaration)是让系统知道某个变量的存在，而变量定义(definiton)是给变量赋值。

```
function doTheThing() {
  // ReferenceError: notDeclared is not defined
  // 在全局和函数作用域内，变量notDeclared没有被声明
  console.log(notDeclared);

  // Outputs: undefined
  // 尽管definedLater是在该语句之后声明的，但实际上声明会被提到最顶部
  // 所以这里definedLater是已经被声明，但是还没被定义（赋值）
  console.log(definedLater);
  var definedLater;

  definedLater = 'I am defined!'
    // Outputs: 'I am defined!
    // 这时候已经被赋值了
  console.log(definedLater);

  // Outputs: undefined
  // 同时定义和声明的情况，注意最后还是被分开
  console.log(definedSimulateneously);
  var definedSimulateneously = 'I am defined!';

  // Outputs: 'I am defined!'
  console.log(definedSimulateneously);

  // Outputs: 'I did it!'
  // 函数的定义同样会被提升
  doSomethingElse();

  function doSomethingElse() {
    console.log('I did it!');
  }

  // TypeError: undefined is not a function
  // functionVar其实是一个变量，这样就好理解了
  // 这时候functionVar被声明了，但是还没被定义赋值
  functionVar();

  var functionVar = function() {
    console.log('I did it!');
  };
}
```

为了使代码可读性更强，将变量的声明都直接放在函数作用域的顶部，就比较清楚变量是来自哪个作用域。
在需要使用之前，才定义（赋值）变量。
在作用域的地步定义函数，减少对代码阅读的影响

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips