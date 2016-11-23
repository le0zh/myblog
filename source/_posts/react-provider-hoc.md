title: React开发中的Provider和HOC模式
tags:
  - react-native
  - react
categories:
  - javascript
date: 2016-11-23 16:19:12
---

### Provider模式

许多的React库都需要在所有的组件树中传递数据。比如说，Redux需要传递他的store，而React Router需要传递当前的地址。

一个看似可行的方案时使用共享的可变状态，但这只能在客户端使用。当我们需要使用服务器端预渲染时，这种方案不可靠。


所幸，React提供了一种自上而下传递数据的途径： context。我们可以把它看做组件树的一个全局变量。


在app的最外部，我们可以提供一个Provider，它的唯一角色就是给当前的组件树的context增加数据，来提供给所有的子节点使用。

我们将使用"主题"的例子来解释这个模式： **我们需要将自定义的主题信息传递到app的每个地方**。

<!-- more -->

```jsx
import React, { Component, PropTypes, Children } from "react"

class ThemeProvider extends Component {
  static propTypes = {
    theme: PropTypes.object.isRequired,
  }

  // 必须要指定context中是什么
  static childContextTypes = {
    theme: PropTypes.object.isRequired,
  }

  getChildContext() {
   const { theme } = this.props
   return { theme }
  }

  render() {
    // Children.only使我们不需要为空的组件再添加一个<div />
    return Children.only(this.props.children)
  }
}

export default ThemeProvider
```

有了这个Provider，我们可以将`theme`传递给任何需要它的组件中。

```jsx
import React from "react"
import { render } from "react-dom"

import ThemeProvider from "ThemeProvider"
import App from "App"

const mountNode = document.querySelector("#App")

const theme = {
  color: "#cc3300",
  fontFamily: "Georgia",
}

render(
  <ThemeProvider theme={theme}>
    <App />
  </ThemeProvider>,
  mountNode
)
```

好了，现在`theme`被添加到了`context`中，我们还需要给组件一个简单的方法能够取到这个数据，这里我们将使用第二个模式。


### Higher-Order Component 模式（HOC）

可以说，每个需要使用`theme`的组件都要声明一个静态的`contextTypes`属性。

但这其实是一个不明智的做法，原因有下面两点：

- **可维护性**:  如果需要重构，整个app中都散布这这些`contextTypes`会给我们造成不小的麻烦。同时，当你想弃用某个属性时，很难方便的提供弃用说明；

- **复杂度**:  对于新手而言，还需要理解`context` API（而这可能会是痛苦的）。对于大多数库而言，组件不需要直接访问到`context`就可以。



还有一个潜在的解决方案是通过继承，但这也不是一个完美的方案，理由如下：


- **多余一层的继承是不明智的**: 多余一层的继承通常会导致方法冲突，当修改时，需要我们检查每个父类。

- **互通性**: 在React中，声明组件有三种方式，`class extends React.Component {}`, `React.createClass({})` 和无状态的函数 `({ props }) => …` 对于后两种方式来说，都不是继承自` React.Component`的。

因此，最好的方法是通过一个可复用的函数创建一个Higher-Order Component（HOC）。基本上来说，我们通过一个仅仅负责获取`context`的组件来包裹其他组件，同时将`context`作为`props`传递。

事实上，我们可以把HOC看做一个app的注入点，将原来的`context`注入到`props`，这么做有很多的好处：



- **隔离性**： 在自己的class中没有方法或者属性冲突的风险（相对于继承的方案）

- **互通性**： 不论组件是怎么被定义的，都可以配合使用

- **维护性**： Wrapper组件内只有一个功能，可以很容易调查（如果出了问题的话）


```jsx
const React, { Component, PropTypes } from "react"
const theme = (ComponentToWrap) => {
  return class ThemeComponent extends Component {
    // 定义想要从context中获取的属性
    static contextTypes = {
      theme: PropTypes.object.isRequired,
    }

    render() {
      const { theme } = this.context
      // 这里做的，仅是渲染ComponentToWrap，但是添加了theme属性
      return (
        <ComponentToWrap {…this.props} theme={theme} />
      )
    }
  }
}

export default theme
```

接下来，既可以使用`theme`方法来封装任何类型的组件了：

#### 无状态函数

```jsx
import React from "React"
import theme from "theme"

const MyStatelessComponent = ({ text, theme }) => (
  <div style={{ color: theme.color }}>
    {text}
  </div>
)

export default theme(MyStatelessComponent)
```

#### 类定义的组件

使用ES7中的decorator：

```jsx
import React, { Component } from "React"
import theme from "theme"

// using `theme` as a ES7 decorator
@theme
class MyComponent extends Component {
  render() {
    const { text, theme } = this.props
    return (
      <div style={{ color: theme.color }}>
        {text}
      </div>
    )
  }
}

export default MyStatelessComponent
```

直接调用`theme`方法：

```jsx
// or just calling it
import React, { Component } from "React"
import theme from "theme"

class MyComponent extends Component {
  render() {
    const { text, theme } = this.props
    return (
      <div style={{ color: theme.color }}>
        {text}
      </div>
    )
  }
}

export default theme(MyStatelessComponent)
```

注意`theme`仅仅是一个函数，我们可以使用一个简单的闭包来提供一些选项：

```jsx
const theme = (mergeProps = defaultMergeProps) =>
  (ComponentToWrap) => {
    // …
    render() {
      const { theme } = this.context
      const props = mergeProps(this.props, { theme })
      return (
        <ComponentToWrap {…props} />
      )
    }
    // …
  }
```

使用：

```jsx
// …
const mergeProps = ((ownProps, themeProps) =>
  ({…themeProps, …ownProps})

export default theme(mergeProps)(MyComponent)
```