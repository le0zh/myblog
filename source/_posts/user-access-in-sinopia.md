title: Sinopia中的用户控制
tags:
  - tools
categories:
  - tools
date: 2017-01-11 09:52:12
---

在[这里](http://le0zh.github.io/2016/10/19/private-npm-service-via-sinopia/)，我们介绍了如何使用sinopai来搭建自己的私有npm服务器。
工作中遇到了的一个常见需求，就是对于一些sdk包，需要控制其publish流程，同时只允许若干特定用户才能publish。

这就需要用到sinpia中关于用户控制的配置部分了。
来看看sinopia的配置文件中关于用户控制都有哪些。

<!-- more -->

#### 1. 用户注册

```yaml
auth:
  htpasswd:
    # 保存用户的账号密码等信息
    file: ./htpasswd   
    # 允许注册的最大用户数
    # 默认为1000，改为-1，禁止注册
    max_users: -1  
```

在`auth`配置项中, 首先指定了保存用户的账号密码等信息的`file`文件，其实这就是一个文本文件，使用文本编辑器就就可以打开。
其中保存了用户名、密码和创建时间等信息，如下：

```yaml
admin:{SHA}fEqNdfer3Yq9h5ZUglD3CZJT4lBs=:autocreated 2017-01-10T09:43:22.204Z
```

> 注：sinopia本身没有使用数据库，这也是为什么其配置非常简单。

如果通过`max_users: -1 `关闭了注册，则用户将不能再通过`npm add user`命令来注册用户。
但是管理员还是可以通过直接修改`htpasswd`文件来添加用户。

#### 2. 权限控制

控制用户对特定包的权限（访问和发布）
先说几个关键字:
- $all : 所有用户
- $authenticated ： 通过认证的用户（npm login）
- $anonymous : 匿名用户


```yaml
# 配置权限管理
packages: 
  'react-native':
    # react-native 覆盖public包，使用内部的包
    # 所有人都能访问（npm install）
    access: $all 
    # 只有用户amdin才能发布（npm public)
    publish: admin

  # 还可以可以通过前缀来区分，比如对于所有以local-开头的包
  # 只能admin可以访问和发布
  #'local-*':
  #  access: admin
  #  publish: admin
  #  # 同时，还可以单独指定storage（包的存放位置）
  #  storage: 'local_storage'

  '*':
    # 公共包
    # 允许所有的用户（包括没有认证的）来访问和发布包
    #
    access: $all
    publish: $authenticated
    # 如果请求的包在本地没有，则代理到npmjs源
    proxy: npmjs
```

主要的配置参考上面的注释即可。总的来说，就是通过对包名进行过滤，分别设置其`access`和`publish`。

我们的一个需求是需要对自己修改过的react-native包进行托管，所有用户都能访问，但是仅允许管理员发布。
只需要下面的配置就能满足:

```yaml
packages: 
  'react-native':
    # react-native 覆盖public包，使用内部的包
    # 所有人都能访问（npm install）
    access: $all 
    # 只有用户amdin才能发布（npm public)
    publish: admin
```