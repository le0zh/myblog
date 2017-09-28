title: dva框架学习--Subscription订阅
tags:
  - javascript
  - dva
categories:
  - javascript
date: 2017-09-25 10:01:12
---

先看下[官方定义](https://github.com/dvajs/dva/blob/5741f8ce2d802a3b49bbacc0db28047f4d1d805b/docs/Concepts_zh-CN.md#subscription)

Subscriptions 是一种从**源**获取数据的方法，它来自于 elm。

Subscription 语义是订阅，用于订阅一个数据源，然后根据条件 dispatch 需要的 action。数据源可以是当前的时间、服务器的 websocket 连接、keyboard 输入、geolocation 变化、history 路由变化等等。

```js
import key from 'keymaster';
...
app.model({
  namespace: 'count',
  subscriptions: {
    keyEvent(dispatch) {
      key('⌘+up, ctrl+up', () => { dispatch({type:'add'}) });
    },
  }
});
```

> subscriptions要求内部所有成员都是函数.

<!-- more -->

#### 疑惑1：subscriptions里面都可以写什么

Q: 官方例子中是监听了按键事件，除此之外还可以些什么。

A: 任意自定义函数

`dva-core/src/checkModel.js`中可以清楚看到：

[code](https://github.com/dvajs/dva/blob/04ba752137017e6bfafb5cc2c6187d7d6db6dd0c/packages/dva-core/src/checkModel.js#L51)

```js
export default function checkModel(model, existModels) {
    // ...


    if (subscriptions) {
        // subscriptions 可以为空，PlainObject
        invariant(
        isPlainObject(subscriptions),
        `[app.model] subscriptions should be plain object, but got ${typeof subscriptions}`,
        );

        // subscription 必须为函数
        invariant(
        isAllFunction(subscriptions),
        `[app.model] subscription should be function`,
        );
    }
}
```

#### 疑惑2：subscriptions里的函数什么时候会被执行

首先subscriptions是定义在model里的，而一般加载model的方法是:

```js

const app = dva({
  history: browserHistory,
  onError(error) {
      // ...
  }
});

app.model(require('./models/app'));
```

所以看下dva中调用`model(...)`方法时做了什么:

[code](https://github.com/dvajs/dva/blob/d74d3d7ce1dafeb5e9d009aae4307b305305b288/packages/dva-core/src/index.js#L58)

```js
function model(m) {
    if (process.env.NODE_ENV !== 'production') {
        checkModel(m, app._models);
    }
    app._models.push(prefixNamespace(m));
}
```

可以看到基本就是处理了一下（加上命名空间前缀）后，保存在了内部的`_models`数组中。

> 同时可以注意到，dva在生产环境中是不做模型检查的，可以提高一点儿效率，但是在开发测试环境中是开启的，便于debug

真正执行Subscriptions的地方在`app.start()`中

[code](https://github.com/dvajs/dva/blob/d74d3d7ce1dafeb5e9d009aae4307b305305b288/packages/dva-core/src/index.js#L188)

```js
function start() {
    // ...

    // Run subscriptions
    const unlisteners = {};
    for (const model of this._models) {
      if (model.subscriptions) {
        unlisteners[model.namespace] = runSubscription(model.subscriptions, model, app, onError);
      }
    }

}
```

`runSubscription`方法定义在 [code](https://github.com/dvajs/dva/blob/master/packages/dva-core/src/subscription.js#L5)

```js
export function run(subs, model, app, onError) {
  const funcs = [];
  const nonFuncs = [];
  for (const key in subs) {
    if (Object.prototype.hasOwnProperty.call(subs, key)) {
      const sub = subs[key];
      const unlistener = sub({
        dispatch: prefixedDispatch(app._store.dispatch, model),
        history: app._history,
      }, onError);
      if (isFunction(unlistener)) {
        funcs.push(unlistener);
      } else {
        nonFuncs.push(key);
      }
    }
  }
  return { funcs, nonFuncs };
}
```

具体的调用代码如下：

```js
 const unlistener = sub({
        dispatch: prefixedDispatch(app._store.dispatch, model),
        history: app._history,
      }, onError);
```

所以subscription的方法签名为`({dispatch, history}, onError)`。

如果subscription的返回值是`function`，则认为是`unlistener`在model从app卸载时会被调用。

在`app.start`之后，还可以动态注入`model`，这时候`model`里面的subscription还是会被执行的。

首先看：[code](https://github.com/dvajs/dva/blob/d74d3d7ce1dafeb5e9d009aae4307b305305b288/packages/dva-core/src/index.js#L196)

```js
function start(){
    // ...

    // Setup app.model and app.unmodel
    app.model = injectModel.bind(app, createReducer, onError, unlisteners);
    app.unmodel = unmodel.bind(app, createReducer, reducers, unlisteners);
}
```

在app启动后，运行期间，再次调用`app.model(...)`其实就是调用`injectModel`:

[code](https://github.com/dvajs/dva/blob/d74d3d7ce1dafeb5e9d009aae4307b305305b288/packages/dva-core/src/index.js#L73)

```js
function injectModel(createReducer, onError, unlisteners, m) {
    model(m); // 首先调用model方法，将m加入app._models中

    // ...

    if (m.subscriptions) { // 如果定义了subscription，则执行
        unlisteners[m.namespace] = runSubscription(m.subscriptions, m, app, onError);
    }
}
```

