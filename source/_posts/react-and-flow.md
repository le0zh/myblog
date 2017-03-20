title: React and Flow
tags:
  - react-native
categories:
  - react-native
date: 2017-03-20 15:35:12
---

React应用通常由嵌套的组件构成。随着应用的迭代，这些组件树和他们之间的依赖将会变得越来越复杂。

Flow的静态分析，能通过追踪检测`props`和`state`的类型，保证基于React构建的大型应用的安全性。
Flow能够理解什么样的`props`是合法的，并支持默认属性。

定义React组件有三种方法：
- `React.createClass`工厂方法
- `React.Component`子类
- 无状态的函数式组件

本文将讲述使用上面的三类方法时，如何定义强类型的组件，同时还会包括一个高阶组件（HOC）的例子。

<!--more-->

### 使用React.createClass方法定义组件

React本身提供了`PropTypes`，用来验证给组件提供的属性是否争取。但是，和Flow的静态分析不一样，`PropTypes`**仅仅在运行时检查**。

如果我们的测试用例没有覆盖到代码的所有路径，则可能遗漏一些程序中的类型错误。

FLow内置了对`PropTypes`的支持。当Flow看到使用`createClass`方法创建的组件指定了`PropTypes`时，能够静态的检查验证组件使用了正确的属性类型。

比如：

```javascript
const Greeter = React.createClass({
  propTypes: {
    name: React.PropTypes.string.isRequired,
  },
  render() {
    return <p>Hello, {this.props.name}!</p>;
  },
});

<Greeter />; // Missing `name`
<Greeter name={null} />; // `name` should be a string
<Greeter name="World" />; // "Hello, World!"
```

运行flow后的输出：

```shell
$> flow
10: <Greeter />; // Missing `name`
    ^^^^^^^^^^^ React element `Greeter`
2:   propTypes: {
                ^ property `name`. Property not found in
10: <Greeter />; // Missing `name`
    ^^^^^^^^^^^ props of React element `Greeter`

11: <Greeter name={null} />; // `name` should be a string
    ^^^^^^^^^^^^^^^^^^^^^^^ props of React element `Greeter`. This type is incompatible with
2:   propTypes: {
                ^ propTypes of React component
```

当属性默认值通过`getDefaultProps`提供了时，Flow能自动识别，并在属性没有被提供时，不产生错误。

注意，即使当提供了默认值时，最好也声明`isRequired`来避免`null`值的情况。 **React只会在属性值为`undefined`时使用默认值**。

```javascript
const DefaultGreeter = React.createClass({
  propTypes: {
    name: React.PropTypes.string.isRequired,
  },
  getDefaultProps() {
    return {name: "World"};
  },
  render() {
    return <p>Hello, {this.props.name}!</p>;
  },
});

<DefaultGreeter />; // "Hello, World!"
<DefaultGreeter name={null} />; // `name` should still be a string
<DefaultGreeter name="Flow" />; // "Hello, Flow!"
```

Flow能确保对`state`的读写和`getInitialState`返回的对象保持一致。

```javascript
const Counter = React.createClass({
  getInitialState() {
    return {
      value: 0,
    };
  },
  increment() {
    this.setState({
      value: this.state.value + "oops!", // flow-error: string. This type is incompatible with number
    });
  },
  decrement() {
    // Note: Typo below is intentional
    this.setState({
      valu: this.state.value - 1,
    });
  },
  render() {
    return (
      <div>
        <button onClick={this.increment}>+</button>
        <input type="text" size="2" value={this.state.value} />
        <button onClick={this.decrement}>-</button>
      </div>
    );
  },
});
```

### 使用`React.Component`子类定义组件

虽然`PropTypes`很好，但是也有局限性。比如，可以定义一个`prop`是某种类型的函数，但是没法具体到这个函数接收什么类型的参数以及返回什么类型的值。

Flow拥有更丰富的类型系统，能够表述这些限制，甚至更多。在使用基于类的组件时，我们可以定使用Flow的声明语法义组件的`props` `defaultProps`。

```javascript
// 定义类型
type Props = {
  title: string,
  visited: boolean,
  onClick: ()=> void,
};

class Button extends React.Component {
  props: Props; // 指定类型

  state: {
    display: 'static' | 'hover' | 'active',
  };

  static defaultProps: { visited: boolean }; // 定义默认属性的值

  onMouseEnter: () => void;
  onMouseLeave: () => void;
  onMouseDown: () => void;

  constructor(props: Props) {
    super(props);
    this.state = {
      display: 'static',
    };

    const setDisplay = display => this.setState({display});

    this.onMouseEnter = () => setDisplay('hover');
    this.onMouseLeave = () => setDisplay('static');
    this.onMouseDown = () => setDisplay('active');
  }

  render() {
    let className = 'button ' + this.state.display;
    if (this.props.visited) {
      className += ' visited';
    }

    return (
      <div className={className}
        onMouseEnter={this.onMouseEnter}
        onMouseLeave={this.onMouseLeave}
        onMouseDown={this.onMouseDown}
        onClick={this.props.onClick}>
        {this.props.title}
      </div>
    );
  }
}
Button.defaultProps = { visited: false };

function renderButton(container: HTMLElement, visited?: boolean) {
  const element = (
    <Button
      title="Click me!"
      visited={visited}
      onClick={() => {
        renderButton(container, true);
      }}
    />
  );
  ReactDOM.render(element, container);
}
```

### 无状态函数式组件

在JSX表达式中，任何返回React元素类型的函数都可以被用作一个组件类。

```javascript
function SayAgain(props: { message: string }){
  return (
    <div>
      <p>{props.message}</p>
      <p>{props.message}</p>
    </div>      
  );
}

<SayAgain message="Echo!" />;
<SayAgain message="Save the whales!" />;
```
无状态函数式组件，可以使用下面的方式指定默认属性值：

```javascript
function Echo({ message, times = 2 }: { message: string, times?: number }) {
  var messages = new Array(times).fill(<p>{message}</p>);

  return (
    <div>
      {messages}
    </div>
  );
}

<Echo message="Helloooo!" />; // times将使用默认值 2
<Echo message="Flow rocks!" times={42} />;
```

### HOC高阶组件

一些情况下，React组件中重复的模式可以被抽象出一个函数，将一个组件类转换为另外一个。

在下面的例子中，`loadAsync`作为一个高阶组件，需要指定一个特定的配置类型，和一个最终返回这个配置对象的Promise对象，返回一个用于处理加载中和数据的组件。

```javascript
function loadAsync<Config>(
  ComposedComponent: ReactClass<Config>,
  configPromise: Promise<Config>,
): ReactClass<{}> {
  return class extends React.Component {
    state: {
      config: ?Config,
      loading: boolean,
    };

    load: () => void;

    constructor(props) {
      super(props);

      this.state = {
        config: null,
        loading: false,
      }

      this.load = () => {
        this.setState({loading: true});
        configPromise.then(config => this.setState({
          loading: false,
          config,
        }));
      }
    }

    render() {
      if (this.state.config == null) {
        let label = this.state.loading ? "Loading..." : "Load";
        return (
          <button disabled={this.state.loading} onClick={this.load}>
            {label}
          </button>
        );
      } else {
        return <ComposedComponent {...this.state.config} />
      }
    }
  }
}

const AsyncGreeter = loadAsync(Greeter, new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve({ name: "Async World" });
  }, 1000);
}));

<AsyncGreeter />;
```
