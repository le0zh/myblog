title: RN混合release模式下NativeModule未定义问题记录
tags:
  - react-native
categories:
  - 问题记录
date: 2017-05-12 17:12:12
---

### 背景

1.  Android原生App集成RN
2. 自定义了若干NativeModule

###  现象

js端通过`NativeModules`访问自定义native module。

```
import { NativeModules } from 'react-native'

const Account = NativeModules.AccountNativeModule;
```

在Release模式下，`Account`总是为`{ }`一个空对象，然后调用里面的方法时，由于未定义导致app一致闪退。

<!--more-->

### 排查过程

1. 偶然发现debug模式下，没有问题
2. 定位到只是release模式问题
3. 排查代码没有问题后，定位到`build.gradle`

### 解决办法

`build.gradle`中，关于build类型有以下定义

```
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard.flags'
    }
    debug {
        minifyEnabled false
    }
}
```

两种模式不同的一个配置就是， release模式开启了`minifyEnabled`。
尝试关闭`minifyEnabled`后发现问题解决了。
为了优化，我们还是要在release模式下开启`minifyEnabled`的。

 > 我们怀疑是现在的优化规则把自定义的NativeModule优化掉了。

在`proguard.flags`最后加上下面的规则：

```
# 禁止混淆NativeModule
-keep class com.xxx.xxx.rn.reactModule.**  { *; }
```

done!