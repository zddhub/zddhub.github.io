---
layout: post
title: "前端手记 TodoMVC 之 React 篇"
category: Memo
tags: "TodoMVC 前端手记"
---

保持专注越来越难了，感觉我的注意力也上了年纪，变的花心多了，很容易被打断。能有心情、有时间静坐下来，学学 Todo list 的内容，感觉是最幸福的了。 前端的框架很多，我先从 React 开始。

<!-- more -->


### 知识筹备

* ECMAScript 6.0 (ECMAScript 2015)

JavaScript (js) 是前端工程师绕不过的编程语言，非常灵活，目前 js 已经打通前后端，成为程序员必备技能之一。最近 [微信小程序](https://mp.weixin.qq.com/wiki) 又让 js 火了一把。ECMAScript 6.0 (简称 ES6)是 js 语言的下一代标准，已经在 2015 年 6 月正式发布了。它的目标是，使得 js 语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

熟读 ES6 feature 可以帮我们写出更现代的代码，这里有两本书必须精读：[es6features](https://github.com/lukehoban/es6features) 和 [《ECMAScript 6入门》](http://es6.ruanyifeng.com/)。如果还不够的话，读读[标准](http://www.ecma-international.org/ecma-262/6.0/)吧。

除了 ES6，你还需要了解以下工具：

* webpack: 模块打包工具，并且可以将某些语言编译成 js。
* JSX: 支持 js 和 html 混写
* babel: js 降级
* ...

后端 node.js v4.0＋ 已经支持大部分 ES6 feature。但是前端，各浏览器对 ES6 响应相对较慢，想用 ES6 的话，你需要 babel，将新 feature 编译成可以在浏览器中运行的原生 js 代码。我配置了一个 ES6 + React 的开发环境(2100ad886643344af77c103c995dfb5360a6e492)。 你可以这样运行它:

	webpack -w

用 chrome 打开 `react/index.html`, 如果能看到 `Hello, React! hahah ...` 说明环境搭建完成。当然，也可以运行：

	node server.js # 另打开一个终端

访问 `http://localhost:3000/`, 查看默认界面。接下来就可以专心写 React 啦。


### React

首先，你需要熟读 [官方文档](https://facebook.github.io/react/docs/getting-started.html)，其中有一篇 [Thinking in react](https://facebook.github.io/react/docs/thinking-in-react.html) 写的非常好，需要仔细研读。本文使用 Thinking in react 的步骤来实现 TodoMVC demo。

* 回顾一下 UI

在 [前端手记 TodoMVC 之 CSS 篇]({% post_url 2016-08-27-todomvc-css %}) 我们按步骤实现了 TodoMVC 的前端 UI，如下图所示：

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/assets/images/2016-09-04/d3d6da9aee6cc74cff3d5feda1577bde87467b58.png)

假设 JSON API 返回的数据如下：

```json
[
  {id: "1", title: "learn css first", completed: false, date: "2016-09-07T22:01:45Z"},
  {id: "2", title: "edit todo", completed: false, date: "2016-09-07T22:02:45Z"},
  {id: "3", title: "learn react", completed: true, date: "2016-09-07T22:03:45Z"}
]
```

* 第一步，根据 UI 划分 component 层级结构

React 将网页上的元素划分成不同模块的 component，每个 component 独立完成自己的显示和逻辑。component 可以重用。

划分组件时使用单一职责原则，保证每个组件只做一件事情。下图是我的一个划分：

![TodoMVC React Components]({{ post.img | replace: '..', site.url }}/assets/images/2016-09-06/todomvc-react-components.png)

划分了 7 个 component：

  1. TodoApp (red): 包含 TodoMVC App 的全部
  2. ToggleAll (green): 全选和反选
  3. NewTodo (orange): 添加一个新的 Todo
  4. TodoItem (blue): 操作(状态切换 / 删除)一个 Todo
  5. FilterTodo (purple): 过滤 Todo
  6. Footer (yellow): 包含 TodoMVC App 的脚注
  7. MainSection (black): 包括 TodoMVC 的主要 components

对应的层级结构如下：

```
* TodoApp
  * NewTodo
  * MainSection
    * ToggleAll
    * TodoItem
    * FilterTodo
  * Footer
```

* 第二步，用 React 实现静态页面

这是加上元素的效果，是不是和用 html 写的一样呢？

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/assets/images/2016-09-06/8dc5b7c94a93f2ae6a516ab119aa643bc8734d6d.png)

其中 style 的 class 和 js 有冲突，react 中用 className 定义样式，加上样式的 TodoMVC App 如下：

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/assets/images/2016-09-06/0352a2e9d04ce528c5f2632c794c42cf3b354a21.png)

只做了个微小的调整，感觉还不错，之前的 css 真的是没白写呀。

* 第三步， 确定 UI 上的状态

React 使用状态机来定义页面上的交互，组件在不同状态下显示不同的形式和内容。对于页面上的数据，React 把它分成两类: props 和 state。props 从父组件传入子组件，state 处理交互引起变化的数据。

想想 Todo list 上的数据：

	* 所有的 todo lists
	* 用户输入的新 todo 事项
	* Toggle/ToggleAll checkbox 状态
	* 剩余 item 个数

React 写在 props 上的数据从上层向下传递，并且在传递的过程中，值保持不变。写在 state 上的值发生变化时 (setState), 会引起 component 的重绘，一个应用的 state 数据越少越简单。那么如何区分数据应该用 props 还是 state 呢？有篇文章写这个事情: [Interactivity and Dynamic UIs](https://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html)。

state 是那些组件的事件响应会改变，并触发 UI 更新的数据。可以简单的问三个问题来确定一个数据是不是 state：

	* 是不是通过父组件的 props 传递下来？ 如果是，它应该就不是 state
	* 是不是随时间变化？如果是，它应该就不是 state
	* 可以通过其它 props 和 state 计算得到吗？如果可以，它应该就不是 state

所有的 todo lists 从父组件向下传递，所以不是 state。用户输入和 checkbox 值无法通过父组件传递，也没办法通过计算得到，需要根据用户的输入随时改变 UI 效果，所以是 state。剩余的 item 个数可以通过 props 上的数据计算得到，所以不是 state。那么最终看来，我们的 state 数据有：

	* 用户输入的新 todo 事项
	* Toggle/ToggleAll checkbox 状态

* 第四步， 确定 state 所在的组件

通过上一步，我们确定了最小的 state 数据集合，这一步我们确定这些 state 应该被哪些 components 持有。

React 是单向数据流，数据从父组件层层传递给子组件，究竟谁持有 state 会让程序更加的简单方便，对初学者来说是个难题。可以通过以下步骤来确定：

	* 确定哪些组件要根据 state 来绘制
	* 找到一个共同的父组件
	* 要么是共同父组件，要么是其他在层级里更高级的组件应该拥有这个state
	* 如果你不能找到一个组件让其可以有意义地拥有这个 state，可以简单地创建一个新的组件 hold 住这个 state，并把它添加到比共同父组件更高的层级上。

现在用这个策略梳理一遍我们的应用

	* MainSection 需要通过 state 更新 TodoItem。MainSection 需要通过 NewTodo 输入的值刷新 MainSection。
	* 共同的父组件是 TodoApp
	* 添加和删除 Todo 存在于 TodoApp，从概念上讲是有意义的。

好了，根据分析，我把所有 state 都写在组件 TodoApp 里，其它组件全部用 props 传递数据。

* 第五步，添加反响数据流

在实际应用中，子组件的交互通常会改变父组件的值，可以使用回调函数的方法，把子组件的操作更新到父组件上。子组件再通过 props 值获取更新的数据。这种单方向的数据更新，能减少应用的复杂性。


### 和神作对比

眼里过千遍不如手里过一遍，本例中有很多地方，在具体实现时，才可能会考虑到。所有代码都托管在 [zddhub/todomvc](https://github.com/zddhub/todomvc) 上。依然可以根据图片名字的 commit id checkout 源码，查看具体的实现。

![TodoMVC React Demo]({{ post.img | replace: '..', site.url }}/assets/images/2016-09-06/b122e36d02d72cf9ac7f7d7070b975ac1d0a9142.gif)

开源的 TodoMVC 程序大都结合了各种框架，找到一个纯 js 实现的例子 [tastejs/todomvc](https://github.com/tastejs/todomvc/tree/gh-pages/examples/react/js)，大家可以比较一下和自己实现的区别，相互学习。


### 总结一下

* React 只是 MVC 中的 view
* 页面由 component 组成，每个 component 包括数据展示和交互响应
* React 中的数据主要包括 props 和 state 两类，尽量使用 props，少用 state
* 尽可能的保持数据单向流动

