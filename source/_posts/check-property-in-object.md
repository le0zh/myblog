title: 检查对象中是否存在某个属性
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-01-11 10:40:12
---

当需要检查某个属性是否在一个对象中时，你可能会像下面这样子做： 
```
var myObject = {
    name: 'hello'
};

if(typeof myObject['name'] !== 'undefined') {...}

if(myObject['name'])｛...｝
```

上面的方法没有问题，但是你应该知道有两个原生的方法可以做这类事情，　`Object.in` 和 `Object.hasOwnProperty`， 每个对象都是Object的子类，所以都已经有了这两个方法

<!-- more --> 

### 看下区别
```
var myObject = {
    name: 'hello'
};

myObject.hasOwnProperty('name'); //true
'name' in myObject; //true

myObject.hasOwnProperty('valueOf'); //false, valueOf是从原型链中继承过来的

'valueOf' in myObject; //true
```

这两个方法在检查属性的深度上有所区别，就是说`hasOwnProperty`仅仅当待检查的属性是直接属于目标对象时，才会返回`true`，然而`in`操作符不会区分属性是对象直接创建的或者还是从原型链中继承来的。

下面是另一个例子：
```
var myFunc = function(){
    this.name = 'hello';
};
myFunc.prototype.age = '10 days';

var user = new myFunc();

user.hasOwnProperty('name'); //true
user.hasOwnProperty('age'); //false, 因为age是来自原型链
```
>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips