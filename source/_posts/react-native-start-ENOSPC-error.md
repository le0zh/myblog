title: react-native start后出现ENOSPC错误的解决办法
tags:
  - react-native
  - react
categories:
  - exception
date: 2016-12-06 09:23:12
---

>ubuntu系统

#### 操作
执行`react-native start`


#### 错误：
![img](http://7xl2vf.com1.z0.glb.clouddn.com//blog/enospc-error.png)

<!-- more -->

#### 解决办法：

运行

```
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

参考： http://stackoverflow.com/questions/16748737/grunt-watch-error-waiting-fatal-error-watch-enospc


据说安装watchman也可以解决该问题，但由于安装比较麻烦，暂时未验证。

watchman安装说明：

https://facebook.github.io/watchman/docs/install.html#installing-from-source
