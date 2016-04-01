title: jsformat+collapse-preserve-inline保留小对象一行格式的配置
tags:
  - sublimetext
categories:
  - tools
date: 2016-04-01 13:40:12
---

使用SublimeText开发react的同学注意了，当使用jsformat插件格式化React代码的时候，会出现下面的情况：
import`组件的时候:
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/jsformat/50a1520d-2db0-4ce5-81ad-c36a1b9ff18e.png)

默认参数的情况:
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/jsformat/64458851-b01c-4327-8bc7-4dff730516c5.png)

而一般来说，我还是习惯下面的方式:
> 当然如果你更喜欢使用上面的方式，可以退出阅读啦。

![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/jsformat/6c4e8212-a115-4e27-8b5e-9f33ff3b4cec.png)

![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/jsformat/64e458b9-9ea4-4bb6-a61b-16e87ee1cbf4.png)

<!-- more -->

#### 调查
查阅了很多资料，定位到了jsformat里面使用的js-beautify同样有相同的问题，比如下面这个issue:
https://github.com/beautify-web/js-beautify/pull/853

并且js-beautify在最新的版本中，新加入了一个参数配置：
```
"brace_style": "collapse-preserve-inline"
```
来实现将字面量对象保留在一行的代码风格。

兴冲冲的修改了jsformat的config，发现报错，说`brace_style`没有`collapse-preserve-inline`的属性，ho---no!

大概查了下，跟设想的一样，jsformat没有更新js-beautify的版本，看了下jsformat的[github](https://github.com/jdc0589/JsFormat)，里面也有请求更新js-beautify版本的[issue](https://github.com/jdc0589/JsFormat/issues/165)，但是作者木有回复，可能是比较忙吧，木有办法，手工替换升级吧。

#### 解决办法
先放出升级后的jsforamt插件包：
[下载地址](http://7xl2vf.com1.z0.glb.clouddn.com/st3JsFormat.sublime-package)

使用方法：
1. 将下载到的文件放到Installed Packages文件中，比如我的windows版本的地址是`D:\IDE\Sublime Text Build 3083 x64\Data\Installed Packages`
2.  修改jsformat的配置如下：(关键是`brace_style`的设置)
```
//兼容jsx
"e4x": true,
// 自动保存
"format_on_save": true,
 // {}大括号不再强制换行
 "brace_style": "collapse-preserve-inline",
```
**done**

#### 下面看下遇到一个小问题：
当我把替换后jsformat插件包放到Installed Packages文件夹后，当时ST中是生效的，但是当重启之后，jsformat插件莫名就消失了。。。。

查看日志发现这个插件包被当做"orphaned package"被干掉了，google一番之后，发现了这个链接：
https://forum.sublimetext.com/t/sublime-keeps-deleting-my-loose-packages/15691


帖子的回复已经很清楚了，大概意思是ST会检测包文件里面的有没有`package-metadata.json`文件，
如果有，则Package Control认为它对该插件包有管理权，然后会和`installed_packages`中的数据比较（BTW，如果使用Package Contro安装的插件，都会记录在改文件中），当发现`installed_packages`中没有改插件信息时，就认为它是"orphaned package"，做清理操作，这个设计是为了用户在不同机器上同步插件时使用。

因为我们不是通过Package Control安装的，所以解决办法也很简单，将插件包里面的`package-metadata.json`文件删除即可。