title: react-native中navigator的基本使用
tags:
  - javascript
  - react-native
categories:
  - react-native
date: 2016-01-14 17:40:12
---

本文记录在react-native中navigator的基本使用，解决了页面间如何切换的问题。

>注： 本文是windows环境下开发android

首先，官方文档地址如下：
http://facebook.github.io/react-native/docs/navigator.html#content

从官方文档可以看到navigator的基本用法如下：
```
<Navigator
  initialRoute={{name: 'My First Scene', index: 0}}
  renderScene={(route, navigator) =>
    <MySceneComponent
      name={route.name}
      onForward={() => {
        var nextIndex = route.index + 1;
        navigator.push({
          name: 'Scene ' + nextIndex,
          index: nextIndex,
        });
      }}
      onBack={() => {
        if (route.index > 0) {
          navigator.pop();
        }
      }}
    />
  }
/>
```
从中可以看到两个最基本和必须的属性如下：
+ `initialRoute ` : 该属性指明了一开始的初始路由，即app加载时第一次要展现什么内容
+ `renderScene ` : 渲染场景函数，根据`route`参数确定显示哪些内容

另外还经常会用到的可选属性有：
+ `configureScene `: 场景配置函数，允许配置场景相关的动画和手势

<!-- more --> 

### 下面我们写一个最基本的demo:

将代码直接都写在： `index.android.js`中了。

首先是引用：
```
'use strict';

var React = require('react-native');

var {
    AppRegistry, View, Navigator, Text, BackAndroid, StyleSheet
} = React;

var App = require('./index.js');
```

然后为了在不同页面跳转，模拟创建几个不同的页面（场景），分别是首页、消息、发现和我的页面。
```
var HomeView = React.createClass({
    render: function() {
        return (
            <View>
                <Text>首页view</Text>
            </View>
        );
    }
});

var MessageView = React.createClass({
    render: function() {
        return (
            <View>
                <Text>消息view</Text>
            </View>
        );
    }
});

var DiscoverView = React.createClass({
    render: function() {
        return (
            <View>
                <Text>发现view</Text>
            </View>
        );
    }
});

var UserView = React.createClass({
    render: function() {
        return (
            <View>
                <Text>我的view</Text>
            </View>
        );
    }
});
```

因为android设备存在返回按钮，所以为了保证用户体验的一致性，当用户点击了后退按钮时，需要返回上一个页面：
```
var _navigator; //用来保存navigator

//监听后退按钮的press事件，用来回退页面
BackAndroid.addEventListener('hardwareBackPress', () => {
    if (_navigator.getCurrentRoutes().length === 1) {
        return false;
    }
    _navigator.pop();
    return true;
});
```

接下来定义app的入口：
```
var App = React.createClass({
    configureScene: function(route, routeStack) {
        return Navigator.SceneConfigs.FloatFromRight;
    },

    renderScene: function(router, navigator) {
        var Component = null;
        _navigator = navigator;

        switch (router.name) {
            case 'home':
                Component = HomeView
                break;
            case 'message':
                Component = MessageView;
                break;
            case 'discover':
                Component = DiscoverView;
                break;
            case 'user':
                Component = UserView;
                break;
        }

        //注意这里将navigator作为属性props传递给了各个场景组件
        return <Component navigator = {navigator} />;
    },

    render: function() {
        return (
            <Navigator
                initialRoute ={{name: 'home'}}
                configureScene = {this.configureScene}
                renderScene = {this.renderScene}>
            </Navigator>
        );
    }
});

AppRegistry.registerComponent('Survey', () => App);
```
这时候在模拟器运行如下：

![截图](http://7xl2vf.com1.z0.glb.clouddn.com/f47f972c-f80d-42ef-91c8-78a4fb66e64b.jpg)

OK，默认显示的首页视图，但是没有办法跳转，这就是对了。。因为我们还没有加任何可以跳转的代码。

下面修改首页的视图：
```
var styles = StyleSheet.create({
    container: {
        backgroundColor: '#FFF',
        flex: 1, //flex-grow:1 等分剩余空间
        justifyContent: 'center', //定义了项目在主轴上的对齐方式
        alignItems: 'center', //定义项目在交叉轴上如何对齐
    },
    button: {
        borderRadius: 5,
        marginTop: 20
    }
});

var HomeView = React.createClass({
    onNavPress: function(target) {
        this.props.navigator.push({
            name: target
        });
    },

    render: function() {
        return (
            <View style = {styles.container}>
                <Text>首页view</Text>
                <Text style={styles.button} onPress={() => this.onNavPress('message')}>跳转到 [消息]</Text>
                <Text style={styles.button} onPress={() => this.onNavPress('discover')}>跳转到 [发现]</Text>
                <Text style={styles.button} onPress={() => this.onNavPress('user')}>跳转到 [我的]</Text>
            </View>
        );
    }
});
```

增加了简单的居中样式，然后增加了几个”按钮“，`onNavPress`方法中根据传入的参数，跳转到相应的页面。
下面以消息页面为例，看在各个子页面中，如何回退:
```
var MessageView = React.createClass({
    goBack: function() {
        this.props.navigator.pop();
    },
    render: function() {
        return (
            <View style = {styles.container}>
                <Text>消息view</Text>
                <Text style={styles.button} onPress ={this.goBack}>后退</Text>
            </View>
        );
    }
});
```

是的，具体就在`goBack`方法中。

最终效果：

![gif](http://7xl2vf.com1.z0.glb.clouddn.com/navigator.gif)