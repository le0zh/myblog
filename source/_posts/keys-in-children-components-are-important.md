title: react中子组件的keys很重要
tags:
  - react
  - jstips
categories:
  - javascript
date: 2016-01-28 10:40:12
---

这里所说的[key](https://facebook.github.io/react/docs/multiple-components.html#dynamic-children)是指当使用数组动态创建组件时需要指定的属性。它是一个唯一并且不变的id，React用来区分每个组件的DOM，并且当和另一个组件比较时，可以知道是不同的组件还是相同的。

使用keys可以确保子组件的重用而不是重新创建，从而避免一些奇怪的事情发生。

<!-- more -->

 >Key其实和性能没有多大关系，它更关乎于组件的身份（当然，这会在一定程度上会带来更好的性能）。随机的赋值或者可变的值不能用来作为区分的条件。[ Paul O’Shannessy](https://github.com/facebook/react/issues/1342#issuecomment-39230939)

推荐
+ 使用一个对象中已经存在的唯一的值作为key
+ 在父组件中定义keys，而不是在子组件中

```
//bad 不好的做法
...
render() {
    <div key={{item.key}}>{{item.name}}</div>
}
...

//good 推荐的做法
<MyComponent key={{item.key}}/>
```

+ [使用数组的索引作为key是个不好的习惯](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318#.76co046o9)
+ 不要用`Random()`

```
//bad
<MyComponent key={{Math.random()}}/>
```

+ 我们可以创建自己唯一的id，但是要确保这个方法很快
+ 当子组件的数目很多，或者包含了很“昂贵”的组件（指占用资源多），使用keys来提升性能
+  [必须对`ReactCSSTransitionGroup`的子组件提供key属性](http://docs.reactjs-china.com/react/docs/animation.html)