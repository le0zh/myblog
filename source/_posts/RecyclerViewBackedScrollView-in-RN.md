title: RecyclerViewBackedScrollView的真实面目
tags:
  - react-native
categories:
  - javascript
date: 2016-12-14 14:33:12
---

### Intro

FB的官方ListView例子中，使用了一个`RecyclerViewBackedScrollView作为ListView`的内部scrollView。[ListViewExample.js](https://github.com/facebook/react-native/blob/9ee815f6b52e0c2417c04e5a05e1e31df26daed2/Examples/UIExplorer/js/ListViewExample.js#L67)

```
<ListView
  dataSource={this.state.dataSource}
  renderRow={this._renderRow}
  renderScrollComponent={props => <RecyclerViewBackedScrollView {...props} />}
  renderSeparator={this._renderSeparator}
/>
```
从名字上看很不错，“可重用的”，应该能解决性能问题。但是实际情况呢？
先直接告诉你，实际不是你想象的。

<!-- more -->

### iOS平台

首先看ios平台的实现：
[RecyclerViewBackedScrollView.ios.js](https://github.com/facebook/react-native/blob/9ee815f6b52e0c2417c04e5a05e1e31df26daed2/Libraries/Components/ScrollView/RecyclerViewBackedScrollView.ios.js)

```
'use strict';

module.exports = require('ScrollView');
```
是的，只有这两行，就是直接返回`ScrollView`，这应该是FB先自己挖了个坑，等自己或者社区来填了。

### Android平台

再看Android的吧，定位到 [RecyclerViewBackedScrollView.android.js](https://github.com/facebook/react-native/blob/9ee815f6b52e0c2417c04e5a05e1e31df26daed2/Libraries/Components/ScrollView/RecyclerViewBackedScrollView.android.js#L172)

```
var NativeAndroidRecyclerView = requireNativeComponent(
  'AndroidRecyclerViewBackedScrollView',
  RecyclerViewBackedScrollView
);
```
ok可以看到native层使用的是`AndroidRecyclerViewBackedScrollView`。再继续去java层，找到 [RecyclerViewBackedScrollViewManager](https://github.com/facebook/react-native/blob/9ee815f6b52e0c2417c04e5a05e1e31df26daed2/ReactAndroid/src/main/java/com/facebook/react/views/recyclerview/RecyclerViewBackedScrollViewManager.java)
根据其定义：
```
public class RecyclerViewBackedScrollViewManager extends
    ViewGroupManager<RecyclerViewBackedScrollView>
    implements ReactScrollViewCommandHelper.ScrollCommandHandler<RecyclerViewBackedScrollView> {
}
```

可以看到它管理的对应View为`RecyclerViewBackedScrollView`, 找到这个view的源码 [RecyclerViewBackedScrollView](https://github.com/facebook/react-native/blob/9ee815f6b52e0c2417c04e5a05e1e31df26daed2/ReactAndroid/src/main/java/com/facebook/react/views/recyclerview/RecyclerViewBackedScrollView.java)

```
/**
 * Wraps {@link RecyclerView} providing interface similar to `ScrollView.js` where each children
 * will be rendered as a separate {@link RecyclerView} row.
 *
 * Currently supports only vertically positioned item. Views will not be automatically recycled but
 * they will be detached from native view hierarchy when scrolled offscreen.
 *
 * It works by storing all child views in an array within adapter and binding appropriate views to
 * rows when requested.
 */
@VisibleForTesting
public class RecyclerViewBackedScrollView extends RecyclerView {
}
```
首先，确实是继承自`RecyclerView`。但是从其注释可以得到以下信息：
- 只支持竖向的滚动
- 视图**不会被自动回收**
- 当视图不再屏幕范围内时，会被从native视图层级中移除
- 将子视图保存在一个内部数组

所以，使用`RecyclerViewBackedScrollView`视图还是不会被自动回收的，但是会减少层级中的视图数目。这样就导致：
- 渲染速度还是很快的
- 内存会随着子视图的增加而增加

从其代码也能看出来, `RecyclerView`的三个主要方法 `onCreateViewHolder`（创建视图） `onBindViewHolder`（填充数据） `getItemCount`（获取数量）：
```
@Override
public int getItemCount() {
  return mViews.size();
}

@Override
public ConcreteViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
  return new ConcreteViewHolder(new RecyclableWrapperViewGroup(parent.getContext()));
}

@Override
public void onBindViewHolder(ConcreteViewHolder holder, int position) {
  RecyclableWrapperViewGroup vg = (RecyclableWrapperViewGroup) holder.itemView;
  View row = mViews.get(position);
  if (row.getParent() != vg) {
    vg.addView(row, 0);
  }
}
```
在`onBindViewHolder`中做了一件事情，传入`item`的`Position`，从`mViews`中获得这个row的view对象，而`mViews`是什么呢？
就是上面所提到的内部保存子视图的数组，看代码：
```
private final List<View> mViews = new ArrayList<>();
```

```
public void addView(View child, int index) {
  mViews.add(index, child);

  updateTotalChildrenHeight(child.getMeasuredHeight());
  child.addOnLayoutChangeListener(mChildLayoutChangeListener);

  notifyItemInserted(index);
}
```

### 结论

iOS平台下， `RecyclerViewBackedScrollView`形同虚设。
Android平台下，会减少层级中的视图数目，但不会自动重用。