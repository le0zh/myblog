title: [RN自定义native view中，使用setNativeProps]
tags:
  - react-native
categories:
  - javascript
date: 2016-09-28 14:12:12
---

我们知道，可以使用`setNativeProps`直接修改原生view的属性，而不走react的渲染流程（依赖state/props改变），这在一些需要频繁修改view属性的场景（比如动画）可以避免大量的rerendering来提升性能。
官方文档： http://facebook.github.io/react-native/releases/0.32/docs/direct-manipulation.html
如何使用在官方文档已经很清楚了。

今天我们要讲的是，对于自定义的natvie view及其属性，如何使用`setNativeProps`。

<!--more-->

前提：已经会自定义native view啦。
比如我们有一个自定以的`LeView`，使用如下：

```
 <LeView {...this.props}  ref={com=>this.view = com} ...
```

通过`ref`拿到了`LeView`的引用，同时呢，`LeView`的native代码内部定义了一个react属性`isAnimating`

```
@ReactProp(name = "isAnimating", defaultBoolean = false)
public void setIsAnimating(LeView view, boolean isAnimating) {
    view.SetIsAnimating(isAnimating);
}
```

现在我们想通过`setNativeProps`直接修改其`IsAnimating`属性:

```
this.view.setNativeProps({isAnimating: true});
```

会发现，很不幸，出现readbox报错。。。

查了RN的源码，参考View.js的实现

https://github.com/facebook/react-native/blob/e8c8a06329070eef297fcc0d3aaf493e9d330f94/Libraries/Components/View/View.js


发现，需要在自定义view的js代码中增加

```
/**
 * `NativeMethodsMixin` will look for this when invoking `setNativeProps`. We
 * make `this` look like an actual native component class.
 */
viewConfig: {
  uiViewClassName: 'LeView',
  validAttributes: {IsAnimating: true}
},
```

done.