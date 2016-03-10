title: 使用webpack+babel开始es6开发
tags:
  - es6
  - jstips
categories:
  - javascript
date: 2016-03-11 00:52:12
---

es6中增加了许多好用的特性，可惜浏览器支持还是一个问题，还好出现了babel来将es6的代码转换为es5，既可以使用新特性来开发，同时不用担心浏览器支持问题。搭配webpack相当方便，这里我们来看一个最简单栗子。


> es6在线教程 http://es6.ruanyifeng.com/


<!-- more -->

### 1.准备项目

```
mkdir demo
cd demo
mkdir src
npm init
```

先创建了项目目录`demo`，然后建立了一个`src`目录，里面存放我们的源代码，使用`npm init` 命令初始化一个项目。

在项目根目录创建一个`index.html`:

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>ES6 webpack demo</title>
</head>

<body>
	<div id="container"></div>
	<script src="dist/bundle.js"></script>
</body>

</html>
```

在`src` 创建一个简单的源代码文件:

```
const welcomeMessage = 'ES6 is awesome';
const content = `hello, ${welcomeMessage}`;

let container = document.getElementById('container');
container.innerText = content;
```

使用了es6的`let` `const` 和 模板字符串。

### 2.安装babel

```
npm install --save-dev babel-loader babel-core --save

```
在Babel的6.x之前的版本，Babel默认包含了一些特定的转换，但是在6.x版本之后，将不再包含这些转换，也就说默认是没有任何转换的。需要我们显式的告诉Babel需要哪些转换。 最简单的办法是使用一个`preset` ,比如`ES2015 Preset`。 我们可以使用下面的命令安装：

```
npm install babel-preset-es2015 --save

```

安装完成之后，在项目根目录创建一个`.babelrc` 的配置文件来使用一些插件，比如上面安装的`ES2015 Preset`
```
echo '{ "presets": ["es2015"] }' > .babelrc
```

### 3.安装webpack和创建配置文件

```
npm install webpack --save
```

之后创建配置文件`webpack.config.js`

```
var path = require('path');
var webpack = require('webpack');

module.exports = {
    entry: [
        './src/index'
    ],

    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js'
    },

    module: {
        loaders: [{
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader'
        }]
    }
};

```

### 4.转换

运行命令:

```
webpack --progress
```

完成之后，查看`/dist/bundle.js` 检查代码，可以看到我们使用ES6编写的代码被转换成了标准的ES5版本的：

```
var welcomeMessage = 'ES6 is awesome';
var content = 'hello, ' + welcomeMessage;

var container = document.getElementById('container');
container.innerText = content;
```

### 5.在浏览器打开index.html

**It works!**