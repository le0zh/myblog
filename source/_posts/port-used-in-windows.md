title: windows下使用命令查看端口占用情况
tags:
  - windows
  - cmd
categories:
  - windows
date: 2016-02-29 09:30:12
---

### 使用下面的命令查看端口占用情况：

比如查看3000端口的占用情况
```
netstat -ano|findstr 3000
```
运行后，结果如下：

<!-- more -->

![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/windowsport/1264528b-c33f-4cbb-af68-819d568ba157.png)

可以看到进程号为9692的占用了该端口，使用下面的命令查看是哪个任务：

```
tasklist|findstr 9692
```
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/windowsport/0103c59f-0d06-4f96-a2ad-13600c0b490a.png)

使用下面的命令关闭进程:

```
taskkill /f /pid 9692
```
![img](http://7xl2vf.com1.z0.glb.clouddn.com/blog/windowsport/64289a29-f248-4a73-8621-000a42b72945.png)