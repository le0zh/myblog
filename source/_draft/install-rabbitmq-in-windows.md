title: 在windows环境中安装RabbitMQ
tags:
  - rabbitmq
categories:
  - service
date:  2016-01-20 10:40:12

### 下载
首先到官网[下载](http://www.rabbitmq.com/download.html)
这里我们下载的是windows版本的。

### 安装server
在安装rabbitmq之前，需要下载[Erlang Windows Binary File](http://www.erlang.org/download.html)并安装，大约会需要5分钟。

之后运行安装文件rabbitmq-server-3.6.0.exe，大概需要2分钟，将以默认配置设置RabbitMQ并以服务的形式运行。

### 启动server
安装完毕之后，默认RabbitMQ服务是自动启动的，当然你也可以通过开始菜单手动的启动服务。

### 配置文件
对于windows来说，RabbitMQ的配置文件存放在`%APPDATA%\RabbitMQ\`，名字为`rabbitmq.config`
改配置文件是表展的Erlang配置文件格式，具体可查看[Erlang Config Man Page](http://www.erlang.org/doc/man/config.html).

配置文件示例:
```
[
    {mnesia, [{dump_log_write_threshold, 1000}]},
    {rabbit, [{tcp_listeners, [5673]}]}
]
```
上面的配置首先修改了`dump_log_write_threshold`日志文件的的上限(默认是100)，同时将RabbitMQ的监听端口从5672改为5673。