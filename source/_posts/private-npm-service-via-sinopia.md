title: 使用sinopia搭建私有服务器
tags:
  - tools
categories:
  - tools
date: 2016-10-19 09:53:12
---

![img](http://7xl2vf.com1.z0.glb.clouddn.com//blog/sinopia.png)

github: https://github.com/rlidwka/sinopia

![img](http://7xl2vf.com1.z0.glb.clouddn.com//blog/dict.png)

<!-- more -->

### 安装
```
# installation and starting (application will create default
# config in config.yaml you can edit later)
$ npm install -g sinopia
```

*如果需要https支持:*
```
# if you use HTTPS, add an appropriate CA information
# ("null" means get CA list from OS)
$ npm set ca null
```

### 启动

```
# 使用默认配置的方式
$ sinopia

# 指定配置文件和监听端口的方式
$ sinopia --config ./config/webl.yaml --listen 0.0.0.0:4873
```

启动后，访问 `http://localhost:4873/`  可以看到下面的页面,里面列出来了所有的【私有】模块

![img](http://7xl2vf.com1.z0.glb.clouddn.com//blog/web.png)

使用`pm2`启动:(守护进程， 部署服务器时一定要用这种启动方式)
如果没有安装pm2，先安装`npm install pm2 -g`

```
$ pm2 start which sinopia
```

### sinopia服务端配置

sinopia运行后，会打印出来默认的配置文件位置
```
le0zh@ProDesk:~/code/router-mock-server$ sinopia
 warn  --- config file  - /home/le0zh/.config/sinopia/config.yaml
 warn  --- http address - http://localhost:4873/
```

其中`config.yaml`就是主要的配置文件，同时其同级目录下会有`htpasswd` 文件（添加用户后，自动创建的，保存认证信息）。
另外还有一个`storage`文件夹，用于存放package，其目录在`config.yaml`中配置。

下面是修改过的`config.yaml`，注意注释：

```
#
# This is the default config file. It allows all users to do anything,
# so don't use it on production systems.
#
# Look here for more config file examples:
# https://github.com/rlidwka/sinopia/tree/master/conf
#

# path to a directory with all packages
storage: ./storage  //npm包存放的路径

auth:
  htpasswd:
    # 保存用户的账号密码等信息
    file: ./htpasswd   
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    # 默认为1000，改为-1，禁止注册
    max_users: -1  

# a list of other known repositories we can talk to
uplinks:
  npmjs:
    # 默认为npm的官网，由于国情，修改 url 让sinopia使用 淘宝的npm镜像地址
    url: http://registry.npm.taobao.org/  

# 配置权限管理

packages: 
  'react-native':
    # react-native 覆盖public包，使用内部的包
    access: $all
    publish: $authenticated

  '@*/*':
    # scoped packages 内部包
    # 表示哪一类用户可以对匹配的项目进行安装(install)
    access: $all
    publish: $authenticated

  '*':
    # 公共包
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs

# log settings
logs:
  - {type: stdout, format: pretty, level: http}
  #- {type: file, path: sinopia.log, level: info}

# you can specify listen address (or simply a port)
# 默认没有，只能在本机访问，添加后可以通过外网访问。
listen: 0.0.0.0:4873 
```

#### 覆盖public包

比如我们的需求是，覆盖react-native的public包，使用我们自己的包，配置的方法就是在上面package节点，增加一个入口，同时移除其proxy属性即可。
注意：有可能有缓存，导致publish失败，这时候可以修改version为更高的版本，或者到storage文件夹下删除对应的缓存包即可!!

### 客户端使用

> 开发人员其实只需要关注如何使用

首先需要修改npm配置的registry，有两种方式：

1.  永久的方式：
    ```
    # npm configuration
    $ npm set registry http://localhost:4873/
    ```
    可以使用命令`$ npm config get registry`检查自己当前的registry源配置。
    将本机的npm直接指向私有服务器，之后按正常的方式使用npm就可以了。

2. 临时的方式：
    在使用npm命令时，加上`--registry`参数，比如:
    ```
    npm install react --save --registry  http://localhost:4873/
    ```

但是推荐使用nrm管理npm的源：

```
$ npm install -g nrm # 安装nrm
$ nrm add XXXXX http://XXXXXX:4873 # 添加本地的npm镜像地址
$ nrm use XXXX # 使用本址的镜像地址
```

```
$ nrm --help  # 查看nrm命令帮助
$ nrm list # 列出可用的 npm 镜像地址
$ nrm use taobao # 使用`淘宝npm`镜像地址
```

比如：
```
$ nrm add webl http://10.57.177.232:4873/
$ nrm use webl
```
