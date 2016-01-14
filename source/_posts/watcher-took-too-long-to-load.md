title: react-native在windows下Watcher took too long to load的错误解决
tags:
  - javascript
  - react-native
categories:
  - react-native
date: 2016-01-14 10:40:12
---

### 问题:
在windows下，使用`react-native init projectname`之后，使用`react-native start`命令，过一段时间之后，报错误如下：

![截图](http://7xl2vf.com1.z0.glb.clouddn.com/656f3641-c4c4-4301-a393-45edd016637e.png)

### 解决办法：
参考： 
http://facebook.github.io/react-native/docs/linux-windows-support.html#content

打开文件
`node_modules\react-native\packager\react-packager\src\FileWatcher\index.js`

增加等待时间，我这里是加了个0。

![截图](http://7xl2vf.com1.z0.glb.clouddn.com/2b8e89d1-d53b-465e-8192-ed239f040fa6.png)

保存之后再运行`react-native start`

![截图](http://7xl2vf.com1.z0.glb.clouddn.com/15125506-cdf6-409e-bad7-9057f114c0a2.png)