title: 为npm依赖包指定git仓库
tags:
  - npm
categories:
  - javascript
date: 2016-09-29 16:46:24
---

通常情况下，`package.json`中的`dependencies`是像下面这样定义：

```
"mocha": "^3.0.1"
```

即`包名 + 版本号`定义的，这种情况下，运行 `npm install` 时npm会在自己的服务器上搜索该包并下载到本地的`node_modules`目录。

**其实npm同时还提供了另外几种方式来指定包的来源**：
<!-- more -->
1. 指定下载的url地址
2. 指定git仓库信息

今天我们要记录的是，通过指定git仓库信息，让npm去我们的私有仓库下载包，一定程度上实现私有npm服务的效果。

> 这里采用了ssh的连接方式，需要先在私有git仓库上设置好ssh

1. 首先`npm init`创建一个空的项目和`package.json`文件；
2. 修改`package.json`中的`dependencies`, 增加
   ```
   "react-native-tv-focusable-view": "git+ssh://git@git.devops.letv.com:zhuxinliang/react-native-tv-focusable-view.git"
   ```
3. 运行`npm install`后
4. 检查`node_modules`会发现，`react-native-tv-focusable-view`已经安装好可以使用了。

