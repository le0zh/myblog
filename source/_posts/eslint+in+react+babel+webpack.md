title: 在React+Babel+Webpack环境中使用ESLint
tags:
  - javascript
  - react
categories:
  - javascript
date: 2016-06-21 11:43:12
---

[ESLint](http://eslint.org/)是js中目前比较流行的插件化的静态代码检测工具。通过使用它可以保证高质量的代码，尽量减少和提早发现一些错误。使用eslint可以在工程中保证一致的代码风格，特别是当工程变得越来越大、越来越多的人参与进来时，需要加强一些最佳实践。

>本文假设您已经有一个react+babel+webpack的起始工程，可以参考[react-webapp-startkit](https://github.com/le0zh/react-webapp-startkit.git)

<!-- more -->

#### 首先，安装eslint包

在项目的跟目录下，运行
```
npm --save-dev install eslint
```

因为我们使用了webpack，所以必须要告诉webpack我们在构建时使用eslint，安装[eslint-loader](https://github.com/MoOx/eslint-loader)

```
npm --save-dev install eslint-loader
```

安装之后，我们可以再webpack配置中使用eslint加载器了。
webpack.config.js
```
...
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /node_modules/,
      loader: 'react-hot!babel'
    },
    {
      test: /\.js$/,
      exclude: /node_modules/,
      loader: 'eslint-loader'
    }
  ]
},
...
```

此外，我们既可以在webpack配置文件中指定检测规则，也可以遵循最佳实践在一个专门的文件中指定检测规则。我们就采用后面的方式。

在根目录下：
```
touch .eslintrc
```

.eslintrc

```
{
  "rules": {
  }
}
```

稍后我们可以在该文件中指定规则，但首先我们要在Webpack配置文件中引入该文件。

webpack.cofnig.js

```
...
devServer: {
  contentBase: './dist',
  hot: true,
  historyApiFallback: true
},
eslint: {
  configFile: './.eslintrc'
},
plugins: [
...
```

现在可以启动app了，在根目录下
```
npm run dev // 取决与package.json中的定义
```

你可能会看到`The keyword ‘import’ is reserved`的解析错误。这是因为eslint还不知道通过Babel使用的ES6特性(比如import)。


#### ESLint + Babel

之前，我们已经安装了[babel-loader](https://github.com/babel/babel-loader)(在起步工程中)来转换我们的代码。现在我们可以将它和eslint-loader一同使用。

webpack.config.js

```
...
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /node_modules/,
      loader: 'react-hot!babel'
    },
    {
      test: /\.js$/,
      exclude: /node_modules/,
      loaders: ['babel-loader', 'eslint-loader']
    }
  ]
},
...
```

**或者，使用webpack的preLoaders**

webpack.config.js

```
...
module: {
  preLoaders: [
    {
      test: /\.js$/,
      exclude: /node_modules/,
      loader: 'eslint-loader'
    },
  ],
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /node_modules/,
      loader: 'react-hot!babel'
    }
  ]
},
...
```

我们可以通过[babel-eslint](https://github.com/babel/babel-eslint)来检测ES6代码。

还是先安装
```
npm install --save-dev babel-eslint
```

修改`.eslintrc`

```
{
  parser: "babel-eslint",
  "rules": {
  }
}
```

现在应该可以启动app了，但是没有任何错误显示，不要高兴的太早，这只是因为我们还没有添加任何检测规则。


#### ESLint规则

我们来添加我们的第一条规则。

修改`.eslintrc`
```
...
"rules": {
  "max-len": [1, 70, 2, {ignoreComments: true}]
}
...
```

我们添加了一条规则来检查代码的单行长度，当单行代码长度大于70个字符时，检测会报错。

启动app,你可能会看到关于代码长度的错误，因为某些行多于70个字符了。我们可以修改规则来允许更多的字符。

`.eslintrc`
```
...
"rules": {
  "max-len": [1, 120, 2, {ignoreComments: true}]
}
...
```

如果还有错误，可能你就需要考虑修改代码了。

#### React的ESLint规则

现在来添加一些检测React的代码规则，使用[eslint-plugin-react](https://github.com/yannickcr/eslint-plugin-react)。

```
npm --save-dev install eslint-plugin-react
```

安装之后，我们可以使用react插件来指定我们关于react的第一条代码规则。比如我们要求组件指定`PropTypes`。

.eslintrc
```
{
  parser: "babel-eslint",
  "plugins": [
    "react"
  ],
  "rules": {
    "max-len": [1, 120, 2, {ignoreComments: true}],
    "prop-types": [2]
  }
}
```

当启动app后，你可能会看到PropTypes定义的错误，你可能想要修复这些错误。

另外，我们可以使用一些包含推荐规则的预设，但暂时我们先扩展自己的规则。

#### 扩展ESLint规则

我们不想每次都指定这些规则，所幸已经有很多符合最佳实践的规则。其中之一就是[Airbnb Style Guide](https://github.com/airbnb/javascript)，此外Airbnb还开源了他们自己的ESlint配置。

已经有一部分依赖包安装了，但还缺少一些：

```
npm --save-dev install eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y
```

Now we can add a one-liner to our ESLint configuration to use Airbnbs’ ESLint configuration. When you look back at the packages we installed, you can see that the configuration includes JSX and React rules.

接下来，通过一行代码的配置来让我们可以使用Airbnb的ESLint配置(你可以通过查看node_modules里面的包来查看，这个配置包含了jsx和React的规则)

.eslintrc
```
{
  parser: "babel-eslint",
  "extends": "airbnb",
  "rules": {
    "max-len": [1, 120, 2, {ignoreComments: true}],
    "prop-types": [2]
  }
}
```

我们可以看到可以很简单的使用别人的配置规则来扩展ESLint规则。我们还可以使用其他的扩展，但目前Airbnb代码规范和ESlint配置非常的受欢迎并被大多数开发者所接受。

#### 微调

有时候，为了迎合自己的项目需要，需要对某些特殊的规则微调。
比如我们不想看到`no-unused-vars`(为使用过的变量定义)的警告，可以

.eslintrc
```
{
  parser: "babel-eslint",
  "extends": "airbnb",
  "rules": {
    "no-unused-vars": 0,
    "max-len": [1, 120, 2, {ignoreComments: true}],
    "prop-types": [2]
  }
}
```

上面这种是全局的配置，如果是只想在某些文件中禁止检测，可以如下修改(通过注释的方式)

src/index

```
/*eslint-disable no-unused-vars*/
import SC from 'soundcloud';
/*eslint-enable no-unused-vars*/
import React from 'react';
import ReactDOM from ‘react-dom';
...
```

#### pre-commit钩子

如果项目使用了git,可以通过使用pre-commit钩子在每次提交前检测，如果检测失败则禁止提交。可以在很大一定程度上保证代码质量。

这里我们使用了`pre-commit`[git](https://github.com/observing/pre-commit)包来帮助我们实现这一目标。

首先在`package.json`中添加script命令：

```
"scripts": {
  "eslint": "eslint --ext .js src"
}
```

其次，安装`pre-commit`
```
npm install pre-commit --save-dev
```

最后，在package.json中配置pre-commit需要运行的命令：

```
"pre-commit": [
  "eslint"
]
```

完成之后，在每次提交之前，都会运行eslint命令进行检测，如果检测到有违反代码规则的情况，则会返回1，导致git commit失败。

### done.
