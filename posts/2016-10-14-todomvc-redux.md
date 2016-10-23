---
layout: post
title: "前端手记 TodoMVC 之 Redux 篇"
category: Memo
tags: "TodoMVC 前端手记"
---

React 将应用程序分解成不同的 component，每个 component 单独完成事件响应或者数据展示。把只负责数据展示的 component 叫做 Presentational Component，易复用。把与业务逻辑／业务处理密切相关的 component 叫做 Container Component，极难复用，通常作为 Presentational Component 的父 component 存在，比如我实现的 TodoMVC 中的 `TodoApp`。

<!-- more -->

React 很好的处理了 View 层的复用，在数据方面，采用单向流动的方式，减少了前端逻辑的复杂程度。它将数据划分为 props 和 state 两种类型，并强烈建议能用 props 就用 props 类型，在万不得已／无法避免的情况下，才使用 state 数据。这是因为 props 上的数据在向下传递的过程中不可变，而 state 上的数据是可变的，并且它通过 setState 方法异步更新。

在单页应用中，