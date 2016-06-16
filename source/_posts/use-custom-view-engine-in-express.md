title: 在Express中使用自定义视图模板引擎
tags:
  - express
categories:
  - javascript
date: 2016-06-16 18:04:12
---

众所周知，Express阵营中视图模板大多是Jade或则ejs，使用起来简单高效。既然是模板，那就需要规定好模板语言的格式，比如jade的缩进格式，ejs的`<% %>`。

那么，什么时候需要自定义视图模板引擎呢？对了，就是当模板语言是自定义的时候。不幸，我们在项目中就遇到了这种需求，所以需要自己实现一个模板引擎。所幸，在Express中使用一个自定义视图引擎是非常简单。往下看。

<!-- more -->

### 相关API：

[app.engine(ext, callback)](http://www.expressjs.com.cn/4x/api.html#app.engine)
[app.set(name, value)](http://www.expressjs.com.cn/4x/api.html#app.set)

这里简单概括下：
+ 使用`app.engine(ext, callback)` 创建视图引擎，其中`ext`是模板文件的**后缀名**, `callback`的函数签名为`function(filePath, options, callback)`,其中
  + `filePath`是模板文件的路径
  + `options`是数据对象，常用来替换模板中的占位符
  + `callback`回调的签名是`function(err, renderedHtml)`

+ 使用`app.set('view engine', 'ext')`配置Application Settings，注册自己的视图引擎，其中ext是模板文件的**后缀名**

### Show Me The Code

> 先准备好一个Express的骨架工程，删除关于view engine的配置

删除app.js中下面的代码（如果有，或者jade）:

```
// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');
```

替换为自定义的视图引擎:

```
// 自定义 视图引擎
var fs = require('fs'); // 读取模板文件内容时，需要fs模块

// 自定义模板文件后缀名为.abc
app.engine('abc', function(filePath, options, callback) {
    // 定义我们自己的视图引擎  
    fs.readFile(filePath, function(err, content) {
        if (err) {
            return callback(new Error(err));
        }

        // 这是一个非常简单实现。。。    
        var rendered = content.toString()
            .replace('#name#', options.name)
            .replace('#age#', options.age);
        return callback(null, r endered);
    })
});

// 指定视图目录
app.set('views', path.join(__dirname, 'views'));
// 注册自定义模板引擎
app.set('view engine', 'abc');
```

注册路由：
```
app.use('/', function(req, res, next) {
    res.render('index.abc', {
        name: 'le0zh',
        age: 31
    });
});
```

在views文件夹中添模板文件index.abc:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>自定义模板引擎测试</title>
</head>
<body>
    <h3>自定义模板引擎测试</h3>
    <p>姓名: #name#</p>
    <p>年龄: #age#</p>
</body>
</html>
```

完成之后，启动Express：
```shell
node app.js
```

使用浏览器访问，可以看到渲染后的页面啦。

### 更进一步
上面的例子中，我们的模板引擎非常简单，当它复杂起来时，一大坨的代码在app.js中，影响阅读。
下面我们模块化，将模板引擎提出来，新建一个`myViewEngine.js`文件，把引擎的逻辑放进去：

```
var fs = require('fs');

var myViewEngine = function(filePath, options, callback) {
  // ...具体略
};

module.exports = myViewEngine;
```

现在，在app.js中只需要：
```
var myViewEngine = require('./myViewEngine');
app.engine('abc', myViewEngine);

// 指定视图目录
app.set('views', path.join(__dirname, 'views'));
// 注册自定义模板引擎
app.set('view engine', 'abc');
```

剩下的工作，就是丰富`myViewEngine.js`的功能了。


