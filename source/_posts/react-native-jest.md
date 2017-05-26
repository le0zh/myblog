title: 使用jest测试ReactNative应用
tags:
  - react-native
categories:
  - react-native
date: 2017-05-26 18:31:12
---

在FaceBook内部，就是使用jest测试RN程序的，其特有的snapshot功能，非常适用于RN应用测试。

![img](https://facebook.github.io/jest/img/content/female-technologist.png)

<!--more-->

### Setup

使用`react-native init`生成的项目，默认已经集成了jest。如果没有，可以按下面的step安装和集成jest。

修改`package.json`：

```json
"scripts": {
  "test": "jest"
},
"devDependencies": {
  "babel-jest": "19.0.0",
  "babel-preset-react-native": "1.9.1",
  "jest": "19.0.2",
  "react-test-renderer": "16.0.0-alpha.6"
},
"jest": {
  "preset": "react-native"
}
```

然后运行`npm install`安装。

### 添加测试

比如现在我们想给已经存在组件`Intro.js`添加测试。

首先，在项目根目录创建`__tests__`（如果没有的话），然后添加测试文件`Intro-test.js`：

```js
import 'react-native';
import React from 'react';
import Intro from '../Intro';

// Note: test renderer must be required after react-native.
import renderer from 'react-test-renderer';

test('renders correctly', () => {
  const tree = renderer.create(
    <Intro />
  ).toJSON();
  expect(tree).toMatchSnapshot();
});
```

### 运行测试

在根目录运行`npm test`。将会生成一个`Intro`组件的快照`__tests__/__snapshots__/Intro-test.js.snap`。

如果代码没有问题，将会有下面的输出：

```shell
PASS  __tests__/Intro-test.js
  ✓ renders correctly (10ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   1 passed, 1 total
Time:        1.604s
```

下次再运行测试的时候，新生成的快照将被用来和之前的快照做比较（以此来判断组件是否被正确渲染）。

快照文件应该和代码一起被提交，当快照测试失败时，开发人员需要根据失败的原因，来判断是因为组件更新还是代码错误导致的。如果是前者，则需要使用`jest -u`来更新快照。

更多关于`jest`测试脚本的编写和分析，将会陆续更新。
