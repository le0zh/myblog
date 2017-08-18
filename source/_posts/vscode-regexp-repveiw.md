title: vscode插件：正则表达式预览
tags:
  - vscode
  - myproject
categories:
  - myproject
date: 2017-08-18 10:02:12
---

曾一度对正则表达式望而却步，直到遇到了神器 https://regexper.com/

在使用时，经常遇到下面的场景：
- 复制代码中的正则表达式，到网站中查看图形预览
- 跳出代码编辑器，在网址中编写正则表达式并查看图形预览（新手向）
- 网络抽风，神器打开很慢

既然使用了扩展性这么强的vscode，为什么不搞一个扩展呢？搜了一下还没有类似的，自己写一个(基于[regexper-static](https://github.com/javallone/regexper-static))。

> 本文就是个预(guang)热(gao)，需要的同学可以试用，关于开发记录会另开一篇文章。

#### 预览功能

![img](http://7xl2vf.com1.z0.glb.clouddn.com//vscode/preview.gif)

#### 编辑功能

![img](http://7xl2vf.com1.z0.glb.clouddn.com//vscode/editor.gif)

#### 安装方法

- 打开vscode，`ctrl` + `p`打开Quick Open并输入： 
  `ext install vscode-regexp-preivew`

- 或者直接在vscode扩展中搜索`regexp preivew`

#### 链接

- 源码链接： [github](https://github.com/le0zh/vscode-regexp-preivew)
- Marketplace链接： [marketplace.visualstudio](https://marketplace.visualstudio.com/items?itemName=le0zh.vscode-regexp-preivew)
