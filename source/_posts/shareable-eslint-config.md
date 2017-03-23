title: 多项目共享eslint-config
tags:
  - react-native
categories:
  - react-native
date: 2017-03-23 10:40:12
---

> RN项目工程化之共享的eslint config

### Lint 

使用Lint对代码进行静态检查已经是前端工程化的标配了。

使用Lint，主要可以解决下面两个问题：
1. 代码格式（用分号？不用分号？2个空格？3个空格？Tab？ 个人喜好无所谓，在团队项目中，还是统一比较好）
2. 为声明变量错误（是的，简单的类型检查，使用了未声明的变量是会被发现的）

<!--more-->

### 背景

我们采用了比较流行的ESLint，然后在项目中根据实际情况，在airbnb共享的规则基础上做了定制化。

有新项目了，就把配置文件拷贝一份，又有新项目了，再拷贝一份 and so on.

一开始没什么问题，当项目进度没那么紧张了，我们开始还之前留下的技术债。

对于eslint，之前关闭的规则，是不是可以考虑打开（追求高了:)）

ok，说做就做，but, 没修改一个规则就得同步到其他项目中，是不是很烦。

时间长了，各个项目原来是一样的规则，慢慢差异大了，差异大了也懒得维护了。形成了恶性循环。

**能不能多项目之间共享eslint配置文件呢？**

答案当然是可以的，想想airbnb共享的规则，其实就是共享的规则，只不过我们在一开始，没有这个需求，也没有眼光高到一开始就认识到这点。

下面我们看如何共享。

### 共享eslint config

简答来说，就是将配置文件作为一个npm包管理，在各个项目中配置为eslint的扩展。（我们自己搭建了私有npm，所以实施起来非常简单）

#### 创建自己的npm包　eslint-config-mine

这里遵循eslint扩展的命名规则，将我们的包命名为 `eslint-config-mine` (mine部分是自定义的)

运行：

```
npm init
```

初始化npm包，根据提示输入配置。

新建`index.js`内容如下：

```
module.exports = {
  globals: {
    __DEV__: true
  },
  rules: {
    // 这里放自定义规则
    semi: [2, "always"]
  }
};
```

注意`index.js`就是个js文件，我们可以从其他文件中读取设置文件或者动态的生成。

#### 准备发布到npm

当eslint-config-mine准备好时，我们就可以准备发布了。

我们还应该在`package.json`中使用`peerDependencies`声明对`eslint`的依赖，推荐的做法是使用“>=”指定需要的eslint的最小版本：

```
peerDependencies: {
    "eslint": ">= 3"
}
```

发布之前，推荐使用`npm link`命令测试一下。做法如下：
1. 在eslint-config-mine中，运行`npm link`
2. 在需要使用共享配置文件的项目中，运行`npm link eslint-config-mine`

这样，在项目中就跟安装了eslint-config-mine一样直接使用了。


#### 使用共享配置文件

Shareable configs are designed to work with the extends feature of .eslintrc files. Instead of using a file path for the value of extends, use your module name. For example:

{
    "extends": "eslint-config-myconfig"
}
You can also omit the eslint-config- and it will be automatically assumed by ESLint:

{
    "extends": "myconfig"
}
在具体的项目中，通过`.eslintrc`文件的`extends`特性使用共享的配置，直接使用npm包名即可，比如：

```
{
    "extends": "eslint-config-mine"
}
```

我们也可以省略`eslint-config-`（这部分会自动被eslint补上）：

```
{
    "extends": "mine"
}
```

注意，这里可以使用数组指定多个扩展：

```
{
    "extends": ["airbnb", "mine"]
}
```

同时，我们可以在各个项目的`.eslintrc`文件中再定义某些具体的规则，覆盖共享的配置。

#### 一个npm包共享多个配置文件

同时支持在eslint-config-mine包里，同时存放多个配置文件，然后在`extends`中指定。

新建另外一个配置文件`my-special-config.js`:

```
module.exports = {
  rules: {
    quotes: [2, "double"];
  }
};
```

然后在`.eslintrc`文件中：

```
{
    "extends": "myconfig/my-special-config"
}
```

注意，我们可以省略`.js`文件名后缀，这样一来，就可以放任意多个的配置文件了。

但是强烈建议提供默认的配置文件(index.js)，来防止错误。

#### 发布npm

进入到`eslint-config-mine`:

```
npm publish
```