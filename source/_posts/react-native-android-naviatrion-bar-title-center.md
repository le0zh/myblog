title: react-native-android下NavigationBar的title居中
tags:
  - react-native
categories:
  - react-native
date: 2016-10-28 12:40:12
---

加一个内置的样式就可以了：
```
<Navigator.NavigationBar
  navigationStyles={Navigator.NavigationBar.StylesIOS}
  routeMapper={{
    LeftButton: this.navBarLeftButton,
    RightButton: this.navBarRightButton,
    Title: this.navBarTitle,
  }}
  style={{ backgroundColor: 'gray' }}
  />
```
就是`  navigationStyles={Navigator.NavigationBar.StylesIOS}`

参考：
https://github.com/facebook/react-native/pull/3028

相关讨论：
https://github.com/facebook/react-native/issues/3142
