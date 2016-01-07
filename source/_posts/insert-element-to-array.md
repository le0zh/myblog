title: 向数组中插入一个元素
tags:
  - javascript
  - jstips
categories:
  - javascript
date: 2016-01-07 12:40:12
---

### 向现有数组中插入一个元素是经常会见到的一个需求。你可以：
使用`push`将元素插入到数组的尾部；
使用`unshift`将元素插入到数组的头部；
使用`splice`将元素插入到数组的中间；

### 上面那些方法都是常见的方法，但并不意味着没有性能更好的方法，比如：

#### 使用`push`很容易就能将元素插入到数组尾部，但是还有一个更快performant的方法：
```
var arr = [1, 2, 3, 4, 5];

arr.push(6);
arr[arr.length] = 6; // 43% faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1
```

两个方法都修改了原有的数组，不相信的话，可以去[jsperf](http://jsperf.com/push-item-inside-an-array)测试一下。

#### 现在我们尝试将元素添加到数组的头部

```
var arr = [1, 2, 3, 4, 5];

arr.unshift(0);
[0].contact(arr); //98% faster in Chrome 47.0.2526.106 on Mac OS X 10.11.1
```

这里需要注意的是，`unshift`修改了原有数组，但是`contact`是返回一个新的数组，测试在这[jsperf](http://jsperf.com/unshift-item-inside-an-array)

#### 将元素插入到数组中间使用`splice`，同时这是最快的方法了

```
var items = ['one', 'two', 'three', 'four'];

items.splice(items.length / 2, 0, 'hello');
```

`splice`会修改原有的数组

#### `splice`的参数说明：

`splice()` 方法向/从数组中添加/删除项目，然后返回被删除的项目。
```
arrayObject.splice(index,howmany,item1,.....,itemX)
```

+ index	必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。
+ howmany	必需。要删除的项目数量。如果设置为 0，则不会删除项目。
+ item1, ..., itemX	可选。向数组添加的新项目。

>本文是js tips系列，翻译自 https://github.com/loverajoel/jstips