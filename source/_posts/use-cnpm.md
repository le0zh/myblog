title: 在国内使用cnpm代替npm
tags:
  - nodejs
categories:
  - nodejs
date: 2016-03-04 00:53:12
---

### 在国内，使用淘宝镜像，不需要VPN：

1. 安装：`npm install -g cnpm --registry=https://registry.npm.taobao.org`
2. 测试安装成功：`cnpm info connect`
3. 使用淘宝镜像安装：`cnpm install <name@version>`

>切身体会呀，就在刚才终于把electron-react-boilerplate的包都装好了。。