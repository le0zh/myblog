title: ReactNative源码(js)中require路径问题
tags:
  - react-native
categories:
  - javascript
date: 2016-12-30 14:11:12
---

看ReactNative源码时，js部分中的require根常识中认为的加载方式不一样，真是困扰好久。
举个例子，在 `webView.ios.js` 文件的头部：

```
var ActivityIndicatorIOS = require('ActivityIndicatorIOS');
var EdgeInsetsPropType = require('EdgeInsetsPropType');
var React = require('React');
var ReactIOSViewAttributes = require('ReactIOSViewAttributes');
var StyleSheet = require('StyleSheet');
var Text = require('Text');
var View = require('View');
```

`ActivityIndicatorIOS`的实际位置是` Libraries/Components/ActivityIndicatorIOS/ActivityIndicatorIOS.ios.js`
而`EdgeInsetsPropType`的实际位置是`Libraries/StyleSheet/EdgeInsetsPropType.js`

这里怎么能直接require的，难道有隐藏的搜索路径设置？

<!--more-->

### 答案

RN的packager使用两种方式来搜索模块。

第一种就是基于头部的文档块，如果我们在文件的头部写了 `@providesModule X`，则就可以在其他地方直接使用 `require('X')` 加载模块；

第二种就是Node的标准方式；

所以react-native源码中的头部一般是下面这样:
```
/**
 * Copyright (c) 2015-present, Facebook, Inc.
 * All rights reserved.
 *
 * This source code is licensed under the BSD-style license found in the
 * LICENSE file in the root directory of this source tree. An additional grant
 * of patent rights can be found in the PATENTS file in the same directory.
 *
 * @providesModule NativeMethodsMixin
 */
```