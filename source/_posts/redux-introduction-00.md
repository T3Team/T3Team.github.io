---
title: Redux介绍
date: 2018-07-10 11:04:18
author: TFsky
tags:
    - Redux
    - React
    - 状态管理 
categories:
    - Redux
---

### 何时使用 Redux？

需要明确一点，Redux 是一个有用的架构，但不是非用不可。事实上，大多数情况，你可以不用它，只用 React 就够了。

React 早期贡献者之一 Pete Hunt 说：

> 你应当清楚何时需要 Flux。如果你不确定是否需要它，那么其实你并不需要它。

Redux 的创建者之一 Dan Abramov 也曾表达过类似的意思:

> 我想修正一个观点：当你在使用 React 遇到问题时，才使用 Redux。

简单说，如果你的UI层非常简单，没有很多互动，Redux 就是不必要的，用了反而增加复杂性。

- 用户的使用方式非常简单
- 用户之间没有协作
- 不需要与服务器大量交互，也没有使用 WebSocket
- 视图层（View）只从单一来源获取数据

上面这些情况，都不需要使用 Redux。

- 用户的使用方式复杂
- 不同身份的用户有不同的使用方式（比如普通用户和管理员）
- 多个用户之间可以协作
- 与服务器大量交互，或者使用了WebSocket
- View要从多个来源获取数据

上面这些情况才是 Redux 的适用场景：多交互、多数据源。

从组件角度看，如果你的应用有以下场景，可以考虑使用 Redux。

- 某个组件的状态，需要共享
- 某个状态需要在任何地方都可以拿到
- 一个组件需要改变全局状态
- 一个组件需要改变另一个组件的状态

发生上面情况时，如果不使用 Redux 或者其他状态管理工具，不按照一定规律处理状态的读写，代码很快就会变成一团乱麻。你需要一种机制，可以在同一个地方查询状态、改变状态、传播状态的变化。

最后需要说明的是：Redux 仅仅是个工具。它是一个伟大的工具，经常有一个很棒的理由去使用它，但也有很多的理由不去使用它。时刻注意对你的工具做出明确的决策，并且权衡每个决策带来的利弊。

## Redux是什么？

Redux是JavaScript状态容器，能提供可预测化的状态管理。

它认为：

- Web应用是一个状态机，视图与状态是一一对应的。
- 所有的状态，保存在一个对象里面。

我们先来看看“状态容器”、“视图与状态一一对应”以及“一个对象”这三个概念的具体体现。

![dw topic](https://tech.meituan.com/img/redux-design-code/%E7%8A%B6%E6%80%81%E5%AE%B9%E5%99%A8.png)

如上图，Store是Redux中的状态容器，它里面存储着所有的状态数据，每个状态都跟一个视图一一对应。

Redux也规定，一个State对应一个View。只要State相同，View就相同，知道了State，就知道View是什么样，反之亦然。

比如，当前页面分三种状态：loading（加载中）、success（加载成功）或者error（加载失败），那么这三个就分别唯一对应着一种视图。

现在我们对“状态容器”以及“视图与状态一一对应”有所了解了，那么Redux是怎么实现可预测化的呢？我们再来看下Redux的工作流程。

![dw topic](https://tech.meituan.com/img/redux-design-code/Redux%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.png)

首先，我们看下几个核心概念：

- Store：保存数据的地方，你可以把它看成一个容器，整个应用只能有一个Store。
- State：Store对象包含所有数据，如果想得到某个时点的数据，就要对Store生成快照，这种时点的数据集合，就叫做State。
- Action：State的变化，会导致View的变化。但是，用户接触不到State，只能接触到View。所以，State的变化必须是View导致的。Action就是View发出的通知，表示State应该要发生变化了。
- Action Creator：View要发送多少种消息，就会有多少种Action。如果都手写，会很麻烦，所以我们定义一个函数来生成Action，这个函数就叫Action Creator。
- Reducer：Store收到Action以后，必须给出一个新的State，这样View才会发生变化。这种State的计算过程就叫做Reducer。Reducer是一个函数，它接受Action和当前State作为参数，返回一个新的State。
- dispatch：是View发出Action的唯一方法。

然后我们过下整个工作流程：

1. 首先，用户（通过View）发出Action，发出方式就用到了dispatch方法。
2. 然后，Store自动调用Reducer，并且传入两个参数：当前State和收到的Action，Reducer会返回新的State
3. State一旦有变化，Store就会调用监听函数，来更新View。

到这儿为止，一次用户交互流程结束。可以看到，在整个流程中数据都是单向流动的，这种方式保证了流程的清晰。

## 为什么要用Redux？

前端复杂性的根本原因是大量无规律的交互和异步操作。

变化和异步操作的相同作用都是改变了当前View的状态，但是它们的无规律性导致了前端的复杂，而且随着代码量越来越大，我们要维护的状态也越来越多。

我们很容易就对这些状态何时发生、为什么发生以及怎么发生的失去控制。那么怎样才能让这些状态变化能被我们预先掌握，可以复制追踪呢？

这就是Redux设计的动机所在。

Redux试图让每个State变化都是可预测的，将应用中所有的动作与状态都统一管理，让一切有据可循。

![dw topic](https://tech.meituan.com/img/redux-design-code/redux%E4%B9%8B%E5%89%8D.png)

如上图所示，如果我们的页面比较复杂，又没有用任何数据层框架的话，就是图片上这个样子：交互上存在父子、子父、兄弟组件间通信，数据也存在跨层、反向的数据流。

这样的话，我们维护起来就会特别困难，那么我们理想的应用状态是什么样呢？看下图：

![dw topic](https://tech.meituan.com/img/redux-design-code/redux%E4%B9%8B%E5%90%8E.png)

架构层面上讲，我们希望UI跟数据和逻辑分离，UI只负责渲染，业务和逻辑交由其它部分处理，从数据流向方面来说, 单向数据流确保了整个流程清晰。

我们之前的操作可以复制、追踪出来，这也是Redux的主要设计思想。

综上，Redux可以做到：

- 每个State变化可预测。
- 动作与状态统一管理。

## redux与react-redux关系图

![react-with-redux](http://zhenhua-lee.github.io/img/react/redux.png)



### 参考资料

[Redux中文文档](https://cn.redux.js.org/)

[美团技术团队-Redux从设计到源码](https://tech.meituan.com/redux_design_code.html)

[阮一峰Redux相关文章](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)