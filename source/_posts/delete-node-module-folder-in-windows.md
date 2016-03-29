title: windows文件目录路径过长导致不能删除的问题
tags:
  - tips
categories:
  - nodejs
date: 2016-03-29 16:28:12
---

使用node之后，有时候需要删除node_module文件夹，经常出现不能完全删除的问题。
提示文件目录路径过长，神烦。。

#### 解决办法1：
使用命令
```
robocopy D:\tmp D:\SourceCode\Wikitec.Component\trunk\1.5\node_modules /purge
```
第一个目录是一个出在的临时目录，在哪儿无所谓，第二个目录是待删除的目录。

#### 解决办法2：
使用另外一个npm包`windows-node-deps-deleter`
https://www.npmjs.com/package/windows-node-deps-deleter#readme
使用：
先安装 `npm install -g windows-node-deps-deleter`
然后全局使用` wnddel dir1 `