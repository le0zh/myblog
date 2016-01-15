title: react-native中DrawerLayoutAndroid的基本使用
tags:
  - javascript
  - react-native
categories:
  - react-native
date: 2016-01-15 13:40:12
---

本文记录在react-native中DrawerLayoutAndroid的基本使用，主要是为了解决导航菜单显示问题。

>本文做练习用的工程来自上一篇文章 [react-native中navigator的基本使用](http://le0zh.github.io/2016/01/14/navigator-in-react-native/)

首先，官方文档地址如下： 
http://facebook.github.io/react-native/docs/drawerlayoutandroid.html#content

`DrawerLayoutAndroid `组件通过其名字就能知道，只能在android系统上使用(好像是废话)。
该组件使用`renderNavigationView`方法来渲染Drawer(一般用来导航)。
该组件的直接子元素是主视图，用来显示内容。

在一开始，导航视图是被隐藏的，但是可以被从窗体的侧边拉出来(是的，就像抽屉一样)，具体的从窗体哪个方向拉，可以通过`drawerPosition`属性指定，同时拉出来的宽度(抽屉的深度)可以通过`drawerWidth`属性指定。

<!-- more --> 

示例code:
```
render: function() {
  var navigationView = (
    <View style={{flex: 1, backgroundColor: '#fff'}}>
      <Text style={{margin: 10, fontSize: 15, textAlign: 'left'}}>I'm in the Drawer!</Text>
    </View>
  );
  return (
    <DrawerLayoutAndroid
      drawerWidth={300}
      drawerPosition={DrawerLayoutAndroid.positions.Left}
      renderNavigationView={() => navigationView}>
      <View style={{flex: 1, alignItems: 'center'}}>
        <Text style={{margin: 10, fontSize: 15, textAlign: 'right'}}>Hello</Text>
        <Text style={{margin: 10, fontSize: 15, textAlign: 'right'}}>World!</Text>
      </View>
    </DrawerLayoutAndroid>
  );
}
```
重要属性说明：
+ `drawerWidth ` Drawer的宽度，类型为Number
+ `drawerPosition ` Drawer拉出的方向，类型为枚举，可选的有`DrawerConsts.DrawerPosition.Left`和`DrawerConsts.DrawerPosition.Right`
+ `renderNavigationView` 一个函数用来渲染导航的视图，当drawer被拉出来时显示

OK，现在开始改造我们上篇文章中创建的工程:

首先加入对`DrawerLayoutAndroid`的引用
```
var {
    AppRegistry, View, Navigator, Text, BackAndroid, StyleSheet, DrawerLayoutAndroid
} = React;
```

然后修改App组件的`render`方法如下，就是将原来的`Navigator`组件包裹在`DrawerLayoutAndroid `中：

```
render: function() {
    var navigationView = (
        <View style={{flex: 1, backgroundColor: '#fff'}}>
          <Text style={{margin: 10, fontSize: 15, textAlign: 'left'}}>I'm in the Drawer!</Text>
        </View>
    );
    return (
        <DrawerLayoutAndroid
            drawerWidth = {200}
            drawerPosition={DrawerLayoutAndroid.positions.Left}
            renderNavigationView={() => navigationView}>
            <Navigator
                initialRoute ={{name: 'home'}}
                configureScene = {this.configureScene}
                renderScene = {this.renderScene}>
            </Navigator>
        </DrawerLayoutAndroid>
    );
}
```

这时候，刷新JS，在模拟器中如下，可以从窗口左边拉出Drawer：

![截图](http://7xl2vf.com1.z0.glb.clouddn.com/blog/20160115/drawer-1.gif)

有了这个之后，我们就可以去掉原来home页面内的导航菜单了，并且每个页面也不需要返回按钮了
下面以home页面为例，其他页面的修改类似：
```
var HomeView = React.createClass({
    render: function() {
        return (
            <View style = {styles.container}>
                <Text>首页view</Text>
            </View>
        );
    }
});
```

下面，我们将导航按钮放进Drawer中，修改App组件中`render`方法内的`navigationView`:
+ `onNavPress`方法中使用了外部的`_navigator`变量来控制页面切换
+  使用了 `ref` 获取对 `DrawerLayoutAndroid` 的引用，在点击菜单之后关闭 `DrawerLayoutAndroid` 

```
onNavPress: function(target) {
    _navigator.push({
        name: target
    });

    //关闭drawer
    this.refs['DRAWER'].closeDrawer();
},

render: function() {
    var navigationView = (
        <View style={{flex: 1, backgroundColor: '#fff'}}>
            <Text style={styles.button} onPress={() => this.onNavPress('home')}>[首页]</Text>
            <Text style={styles.button} onPress={() => this.onNavPress('message')}>[消息]</Text>
            <Text style={styles.button} onPress={() => this.onNavPress('discover')}>[发现]</Text>
            <Text style={styles.button} onPress={() => this.onNavPress('user')}>[我的]</Text>
        </View>
    );
    return (
        <DrawerLayoutAndroid
            ref={'DRAWER'}
            drawerWidth = {200}
            drawerPosition={DrawerLayoutAndroid.positions.Left}
            renderNavigationView={() => navigationView}>
            <Navigator
                initialRoute ={{name: 'home'}}
                configureScene = {this.configureScene}
                renderScene = {this.renderScene}>
            </Navigator>
        </DrawerLayoutAndroid>
    );
}
```
如果没有错误的话，showtime：

![截图](http://7xl2vf.com1.z0.glb.clouddn.com/blog/20160115/drawer-2.gif)

最后，美化下我们的Drawer界面，这里我们直接使用了一个material-design组件库：
https://github.com/react-native-material-design/react-native-material-design

> 友情提醒：该组件库仍然处于开发阶段，API尚不稳定，文档也不全。

根据文档，安装之后，先在代码中添加引用
```
var MD = require('react-native-material-design');
var {
    Card, Button, Avatar, Drawer, Divider, COLOR, TYPO
} = MD;
```

修改app组件的代码如下:
+  使用了新的`Drawer`组件
+  使用了state来区分当前激活状态


```
onNavPress: function(target) {
    _navigator.push({
        name: target
    });

    this.setState({
        route: target
    });

    //关闭drawer
    this.refs['DRAWER'].closeDrawer();
},

getInitialState: function() {
    return {
        route: 'home'
    }
},

render: function() {
    var navigationView = (
        <Drawer theme='light'>
            <Drawer.Header image={<Image source={require('./img/nav.jpg')} />}>
                <View style={styles.header}>
                    <Avatar size={80} image={<Image source={{ uri: "http://facebook.github.io/react-native/img/opengraph.png?2" }}/>} />
                    <Text style={[styles.text, COLOR.paperGrey50, TYPO.paperFontSubhead]}>React Native</Text>
                </View>
            </Drawer.Header>
            <Drawer.Section
                items={[{
                    icon: 'home',
                    value: '首页',
                    active: !this.state.route || this.state.route  === 'home',
                    onPress: () => this.onNavPress('home'),
                    onLongPress: () => this.onNavPress('home')
                },
                {
                    icon: 'message',
                    value: '消息',
                    active: !this.state.route  || this.state.route  === 'message',
                    onPress: () => this.onNavPress('message'),
                    onLongPress: () => this.onNavPress('message')
                },
                {
                    icon: 'search',
                    value: '发现',
                    active: !this.state.route  || this.state.route  === 'discover',
                    onPress: () => this.onNavPress('discover'),
                    onLongPress: () => this.onNavPress('discover')
                },
                {
                    icon: 'settings',
                    value: '我的',
                    active: !this.state.route  || this.state.route  === 'user',
                    onPress: () => this.onNavPress('user'),
                    onLongPress: () => this.onNavPress('user')
                }]}
            />
        </Drawer>
    );
    return (
        <DrawerLayoutAndroid
            ref={'DRAWER'}
            drawerWidth = {200}
            drawerPosition={DrawerLayoutAndroid.positions.Left}
            renderNavigationView={() => navigationView}>
            <Navigator
                initialRoute ={{name: 'home'}}
                configureScene = {this.configureScene}
                renderScene = {this.renderScene}>
            </Navigator>
        </DrawerLayoutAndroid>
    );
}
```

最终的效果如下:
![截图](http://7xl2vf.com1.z0.glb.clouddn.com/blog/20160115/drawer-3.gif)



