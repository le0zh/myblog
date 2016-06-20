title: The React Way 1 (译)
tags:
  - javascript
  - react
categories:
  - javascript
date: 2016-06-14 09:40:12
---

> 原文链接 https://blog.risingstack.com/the-react-way-getting-started-tutorial/ 
本篇文章主要涵盖了react的主要设计思想和概念，并不涉及具体太多的语法，但个人觉得学习新的技术、框架时，其设计思想和概念往往比语法更需要深入了解和思考。


*占坑， 未完待续。。。*

现在，[React.js](https://facebook.github.io/react/)正在高速发展，同时很多有趣的东西不断的问世。我的朋友和同事不断的询问我关于如何开始使用React，以及如何用React的方式思考。

![img](https://risingstack-blog.s3-eu-west-1.amazonaws.com/2015/08/react-js-tutorial-1_eb46e0b8ae8aa21640332a1052069483.png)
> 上图是 React 在编程领域的google搜索趋势。React最初发布是再2013年5月29号，版本号是v0.3.0

然而，React并不是一个框架。是一些**概念、类库和原则**使其成为一个快速、简洁和优美的方式来开发客户端和服务器端的app。

<!-- more -->

在这个分为两部分的教程中，我将解释这些概念，同时建议使用哪些和如何使用。我们将涵盖下面这些技术：

+ ES6 React
+ 虚拟DOM
+ 组件驱动的开发
+ 不变性(Immutability)
+ 从上而下的渲染
+ 渲染路径及优化
+ 通用的工具类库，用来打包，转换ES6，发起请求，调试，路由等
+ 同构的React


当然，我们将会编写代码，我想让它尽量的切实可行。所有的代码都可以在[RisingStack GitHub repository](https://github.com/RisingStack/react-way-getting-started)中找到。


### 教程1：开始使用React.js

如果你已经对React熟悉了，理解了比如虚拟DOM、组件化思想的基础，那么本篇教程可能不太适合你。我们将在接下来的教程中讨论一些中级的话题，这些将会很有趣，我推荐你到时候回来再看。

#### React是一个框架么？

简单来说， **不，它不是一个框架**。
那么它究竟是个什么东西，为什么这么多人热衷于开始使用呢？


React是一个应用程序的**视图(View)**,他同时提供了不同的方式来组织我们的模板，使我们以组件化的方式思考。在React引用程序中，我们应该**分解**我们的站点、页面或者模块成**一些小的组件**。这些组件同时构建在其他不同的组件的组合之上。当一个问题比较棘手时，我们可以把它分解成一些更小的问题并解决。晚些时候，我们在其他地方可以**重用**这些小的组件。整个过程就像搭建乐高。我们在本篇文章的末尾将会深入探讨**组件驱动的开发模式**。


React还使用了虚拟DOM，这使渲染速度变得飞快，同时保持了易理解性和可控性。我们可以和组件化的思想结合起来，获得自上而下渲染的能力。我们将在第二篇文章中讨论这个话题。


好吧，我承认，到现在我还没有回答什么是React的问题。我们知道了组件化和快速渲染，但是为什么它就是一个游戏规则颠覆者呢？
因为**React首先主要是一个概念，其次才是一个类库**。有一些不同的类库已经在遵循这些思想，但有些许的差别。就像每一个编程概念一样，React也有自己的解决方案、工具和类库来形成一个生态系统。在这个生态系统中，你必须要选择自己的工具，构建自己的框架。我知道这听起来有些可拍，但是相信我，你已经知道这些工具中的大多数，我们将他们彼此链接起来，然后你将会惊奇的发现这一切是如此的简单。比如对于依赖处理，我们将不使用任何特殊的黑魔法，而是使用Node的`require`和npm。对于发布订阅，我们将使用Node的`EventEmitter`等等。


你是否已经兴奋了呢？咱们继续。


#### 简单的介绍虚拟DOM

为了能追踪模型的变化，并将他们应用于DOM(也成为渲染)，我们必须清楚下面两个重要的事情：

1. 什么时候数据发生了变化
2. 哪一个DOM元素需要被更新


对于变化监测，React使用了一个观察者模型代替脏检查(持续的模型变化检查)。这就是为什么它不必去计算什么发生了改变，它能立即知道改变发生了。这减少了计算量使应用更平滑。但是真正酷的东西是它如何管理DOM操作的：


对于DOM变化的挑战，React在内存中构建了代表DOM结构的树，并计算哪个DOM元素应该改变。DOM操作消耗很大，我们应该尽量减少。幸运的是，React试图尽可能的使DOM元素保持不变。考虑到很少的DOM操作可以被基于对象代表很快的计算出来，DOM改变带来的代价被很好的降低了。


既然React的差异算法使用了代表DOM的树结构，并且当父级被更改时，会重新计算所有的子树节点，我们应该注意模型的变化，因为所有的子树节点将被重新渲染。
不要悲观，稍后我们将优化这一行为。(剧透一下：使用`shouldComponentUpdate()`和`ImmutableJS`)

![img](https://risingstack-blog.s3-eu-west-1.amazonaws.com/2015/04/react-js-tutorial-2.png)

#### 如何在服务器端渲染?
Given the fact, that this kind of DOM representation uses fake DOM, it's possible to render the HTML output on the server side as well (without JSDom, PhantomJS etc.). React is also smart enough to recognize that the markup is already there (from the server) and will add only the event handlers on the client side.

既然这种DOM表示法使用的是假的DOM，那么就同样可以在服务端渲染出HTMl输出(不借助于JSDom, PhantomJS等)。React同样足够智能来识别已经存在的标记(从服务器端)，并将仅仅在客户端添加事件句柄。

有趣的事实： React渲染出来的HTML标记都包含一个`data-reactid`属性，用来帮助React追踪DOM节点。

#### 有用的链接，其他的虚拟DOM类库

+ [React’s diff algorithm](http://calendar.perfplanet.com/2013/diff/)
+ [The Secrets of React's virtual DOM](http://fluentconf.com/fluent2014/public/schedule/detail/32395)
+ [Why is React's concept of virtual DOM said to be more performant than dirty model checking?](http://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode)
+ [virtual-dom](https://github.com/Matt-Esch/virtual-dom)

### 组件驱动的开发

这曾经是我学习React时感觉最困难的部分之一。在组件驱动的开发中，你将不会再一个模板中看到整个站点。一开始你可能会认为这中方法很糟糕。但是我相信随后你会认识到分片和单一职责的力量。它使事情更容易理解和维护，和被测试所覆盖。
