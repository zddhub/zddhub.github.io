---
layout: post
title: "前端手记 TodoMVC 之 Redux 篇"
category: Memo
tags: "TodoMVC 前端手记"
---

React 将应用程序分解成不同的 component，每个 component 单独完成事件响应或者数据展示。把只负责数据展示的 component 叫做 Presentational Component，易复用。把与业务逻辑／业务处理密切相关的 component 叫做 Container Component，极难复用，通常作为 Presentational Component 的父 component 存在，比如我实现的 TodoMVC 中的 `TodoApp`。

<!-- more -->

React 很好的处理了 View 层的复用，在数据方面，采用单向流动的方式，减少了前端逻辑的复杂程度。它将数据划分为 props 和 state 两种类型，并强烈建议能用 props 就用 props 类型，在万不得已／无法避免的情况下，才使用 state 数据。这是因为 props 上的数据在向下传递的过程中不可变，而 state 上的数据是可变的，并且它通过 setState 方法异步更新。

在复杂的单页应用中，可变的 state 的数据增多，加上异步更新方式，给开发和测试都带来了困难。对于 React 上的数据处理问题， Facebook 出了一个方案叫 [Flux](https://facebook.github.io/flux/)，然而 [Redux](http://redux.js.org/) 走的更远，并且得到了 Flux 作者的认可。Redux 把 state 数据存储在单独的对象 (store) 中, 绑定到顶层的 Container Component 上，使得 state 数据和 React 分离，React component 用到的所有数据都能从 props 上得到。 这一篇就用 Redux 改写我们的 TodoMVC demo。


### Redux 三大原则

* 单一数据源: 整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中;
* State 是只读的: 惟一改变 state 的方法就是触发 action，action 是一个用于描述已发生事件的普通对象。
* 使用纯函数来执行修改: 为了描述 action 如何改变 state tree ，你需要编写 reducers。

就是这样，现在你应该明白 Redux 是怎么回事了。Redux 将 state 数据限制为只读，只有通过写纯虚函数的 reducers 才能改变，这个应用程序的 state 数据被存储在单一的 object tree 上。


### Redux 三个基本概念

* Action

Action 是把数据从应用层传递到 store 的有效载体，它是 store 数据的唯一来源。Action 本质上是 JavaScript 普通对象。我们约定，action 内必须使用一个字符串类型的 type 字段来表示将要执行的动作。Action 仅仅表示某对象发生了什么行为，我们应该尽量减少在 action 中传递的数据。

TodoMVC demo 的 Action：

  ADD_TODO
  REMOVE_TODO
  COMPLETE_TOOD
  TRIGGER_ALL_TODOS
  FILTER_TODOS

异步 API 一般有三个关键时间点可以通知给 reducer：

  1. 请求开始
  2. 请求成功结束
  3. 请求失败

可以用三个不同的 Action 来通知给 reducer，或者用一个 Action，设置不同的 status，到底用哪种完全取决于你。

* Reducer

Action 只是描述了有事情发生了这一事实，并没有指明应用如何更新 state。而这正是 reducer 要做的事情。reducer 就是一个纯函数，接收当前 state 和 action，返回新的 state。函数形式如下：

  (previousState, action) => newState

只要传入参数相同，返回的 newState 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。保持 reducer 纯净非常重要。永远不要在 reducer 里做这些操作：

  1. 修改传入参数；
  2. 执行有副作用的操作，如 API 请求和路由跳转；
  3. 调用非纯函数，如 Date.now() 或 Math.random()。

记得不要修改 previousState 的值，创建一个新的对象返回给 newState。

* Store

使用 reducers 来根据 action 更新 state, 存储在 store 中。store 把之前创建的 Action 和 Reducer 联系在一起。store 的职责有：

  1. 持有应用的 state；
  2. 提供 getState() 方法获取 state；
  3. 提供 dispatch(action) 方法更新 state；
  4. 通过 subscribe(listener) 注册监听器;
  5. 通过 subscribe(listener) 返回的函数注销监听器。

有了 reducer 之后，创建 store 只需要一句话 `let store = createStore(todoApp)`。接下来看看 redux 的数据流吧。


### Redux 数据流

Redux 架构使用严格的单向数据流动方式，其生命周期分为以下四步：

  1. 应用调用 store.dispatch(action) 发送 Action
  2. Redux 根据传入的 action 调用对应的 reducer 方法
  3. 根 reducer 把子 reducer 的结果合并成一颗 state 树
  4. Redux store 保存根 reducer 生成的 state 树

得到的 state 树即为当前应用的下一个 state，所有订阅 store.subscribe(listener) 的监听器都将被调用。监听器可以调用 store.getState() 获得当前 state。


### Redux + todoMVC

首先，设计我们的 state 树，如下：

```json
{
  todos: {
    isFetching: false,
    didInvalidate: false,
    items: [
    {
        "title": "Start Redux",
        "completed": false,
        "createdAt": "2016-10-18T00:15:50.105Z",
        "updatedAt": "2016-10-20T00:35:04.858Z",
        "id": 1
    },
    {
        "title": "Start TodoMVC",
        "completed": false,
        "createdAt": "2016-10-18T11:20:26.200Z",
        "updatedAt": "2016-10-20T00:35:04.858Z",
        "id": 2
    }
    ]
  },
  filter: 'All'
}

```

其中，todos 为异步 action，数据来自服务器, isFetching 表示数据的状态，didInvalidate 标记数据是否过期。filter 为 同步 action, 数据来自客户端操作，默认为 'All'。

好了，现在开始实现你的异步 TodoMVC Demo 吧, 从文档到实现，还是有些坑，试试跨越吧，我们必须过这一关。我的实现在这里 [13a1c3748c7e39f77c04507971b0c13822a6d3a7](https://github.com/zddhub/todomvc), 之后有些整理代码的提交，欢迎补充。


### Props 数据验证

利用 Redux，应用中的 state 被统一存储在 store 中，只在 container 上通过 connect 的方式，把 state 数据 map 成 props，使得 React 中的 component 上，只有一种 props 数据， React 提供了一个验证 props 数据的方法, 使用 `PropTypes`。PropTypes 确保上层传给子 compoennt 的 props 数据符合期望。使用了 PropTypes 验证的代码在这里 [34324f5405b52abbef36e7991542d5a228c85403](https://github.com/zddhub/todomvc)。

注意，并不是说使用了 Redux 的应用中不能有 state， 对于底层的 component，上层 container 不应该了解其内部结构。

除了数据验证，代码测试同样重要，Action 和 Reducer 很容易撰写测试，略。


### 和大神作品比较

除了 [TodoMVC 自带的 demo](http://todomvc.com/examples/react/#/) 外， Redux 项目的例子里也有两个 [todoMVC 的例子](https://github.com/reactjs/redux/tree/master/examples), 我都看了哦，不过像我这种把 TodoMVC 的数据存储在服务器端的奇葩做法，还真不多见。


### 参考资料

[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.45qelzag9)
[React](https://facebook.github.io/react/)
[Redux](http://redux.js.org/)