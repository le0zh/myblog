title: React模式#1 无状态函数
tags:
  - react
categories:
  - react
date: 2017-01-13 09:57:12
---

[无状态函数(Stateless functions)](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions)是一种非常好的方式来定义可复用组件，它们内部不保存状态`state`或者引用`refs`，仅仅是函数。

```javascript
const Greeting = () => <div>Hi there!</div>
```

<!-- more -->

下面是一些说明：

#### 可以给无状态函数传入属性`props`和上下文信息`context`作为参数

```javascript
const Greeting = (props, contex) =>
  <div style={{color: context.color}}>Hi {props.name}!</div>
```

#### 可以定义局部变量(当使用了函数体时)

```javascript
const Greeting = (props, contex) => {
  const style = {
    fontWeight: "bold",
    color: contex.color,
  }

  return <div style={style}>{props.name}</div>
}
```

但是我们还可以使用其他函数来达到相同的目的：

```javascript
// 先定义了一个获取样式的函数
const getStyle = context => ({
  fontWeight: "bold",
  color: contex.color,
});

const Greeting = (props, contex) =>
  <div style={getStyle(context)}>{props.name}</div>
```

#### 可以定义`defaultProps`, `propTypes` 和 `contextTypes`

```javascript
const Greeting = (props, contex) =>
  <div style={getStyle(context)}>{props.name}</div>
```

```javascript
Greeting.propTypes = {
  name: PropTypes.string.isRequired
}
Greeting.defaultProps = {
  name: "Guest"
}
Greeting.contextTypes = {
  color: PropTypes.string
}
```
