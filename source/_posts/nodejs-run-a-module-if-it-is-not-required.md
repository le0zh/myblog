title: (#17) nodejs中判断模块的被调用方式
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-04-08 17:13:12
---

在node中，我们可以判断自己的代码是通过下面哪种方式调用的：

+ 通过node直接调用的方式 `node something.js`
+ 通过require调用的方式 `require('./something.js')`

这在我们想要针对不同的情况做不同的处理时很有用，比如下面的情况。

```
if (!module.parent) {
    // ran with `node something.js`
    app.listen(8088, function() {
        console.log('app listening on port 8088');
    })
} else {
    // used with `require('/.something.js')`
    module.exports = app;
}
```

更多信息，请参考[module](https://nodejs.org/api/modules.html#modules_module_parent)的文档。

---

`module`的几个属性：

| 属性名        | 类型           |注释   |
| ------------- |:-------------:|:-----|
| `module.filename` | String | module的完整文件名 |
| `module.id`      | String      | module的唯一标识，一般来说就是module的完整文件名   |
| `module.loaded` | Boolean      | 标识module是否加载完成 |
| **`module.parent`** | Object      | **第一次加载本module的模块** |

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips