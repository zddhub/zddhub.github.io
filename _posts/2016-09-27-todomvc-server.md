---
layout: post
title: "前端手记 TodoMVC 之 Server 篇"
category: Memo
tags: "TodoMVC 前端手记"
---

在上一篇 [前端手记 TodoMVC 之 React 篇]({% post_url 2016-09-06-todomvc-react %}) 中实现了单页应用 TodoMVC，直接把数据存在前端，当刷新或者重新打开网站时，所有数据都会丢失。虽然对于 TodoMVC 这种应用，把数据缓存在前端是一个不错的选择，但以练习为目的，我还是打算特意为之，实现一个简单的 Server，提供 RESTful API，把 TodoMVC 相关的数据存储在后台。

<!-- more -->

为了 focus 在前端，这里用一个比较重的 web 框架 [sails](http://sailsjs.org/) 来实现服务器端，并且尽量忽略实现细节。并根据实现的 RESTful API 来更新 [前端手记 TodoMVC 之 React 篇]({% post_url 2016-09-06-todomvc-react %}) 中的数据来源。


### RESTful API

REST (Representational State Transfer 即 [资源] 表现层状态变化) 是 Fielding 于 2000 年在其博士论文中提出的互联网软件架构原则。如果一套服务的 API 设计架构符合 REST 原则，就称它为 RESTful API。本文的 API 也使用 REST 原则实现。更多关于 RESTful API 的资料看[这里](https://github.com/aisuhua/restful-api-design-references)。

* RESTful API 设计思路

1. 每个 URI 代表一种资源。由于 URI 是资源，所以应该用名词，而不是动词；
2. 客户端和服务器之间，通过 HTTP/HTTPS 协议传递这种资源的某种表现层；
3. 客户端通过 HTTP 动词，对服务器端的资源进行操作，实现“表现层状态的转化”。

* 路径

路径 (url), 表示 API 的具体网址。在 RESTful 架构中，每个网址代表一种资源，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的“集合”（collection), 所以 API 中的名词也常用复数。TodoMVC server 的 url 应该设计成：

	https://api.example.com/v1/todos

本例把 url 简化为

	http://localhost:1337/todos

* HTTP 动词

对于资源的具体操作类型，由 HTTP 动词表示。

	GET（SELECT）：从服务器取出资源（一项或多项）。
	POST（CREATE）：在服务器新建一个资源。
	PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
	PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
	DELETE（DELETE）：从服务器删除资源。
	HEAD：获取资源的元数据。
	OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

* 状态码（Status Codes）

服务器向用户返回的状态码和提示信息，所有状态码的第一个数字代表了响应的状态:

	1xx 消息
	2xx 成功
	3xx 重定向
	4xx 客户端错误
	5xx 服务器错误

常见的状态码有以下一些（方括号中是该状态码对应的 HTTP 动词）。

	200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
	201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
	202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
	204 NO CONTENT - [DELETE]：用户删除数据成功。
	400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
	401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
	403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
	404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
	406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
	410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
	422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
	500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。

状态码的完全列表参见[这里](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。


### TodoMVC API Design

TodoMVC 只实现一个 API

	http://localhost:1337/todos{/todo_id}

只用 4 个最基本的 HTTP 动词 (GET、POST、PUT、DELETE) 来实现 Todos 的增删查改。

| API | GET | POST | PUT | DELETE |
|-----|-----|------|-----|--------|
| /todos | 获取 todos list | 新建 todo | 更新所有 todos 的状态 | 删除所有的 todos |
| /todos/todo_id | 获取当前 todo | - | 更新当前 todo | 删除当前 todo |

返回的数据格式

```json
[
  {
    "title": "What needs to be done？",
    "completed": "false",
    "createdAt": "2016-10-11T12:28:43.572Z",
    "updatedAt": "2016-10-11T12:28:43.572Z",
    "id": 1
  }
]

```

除了 `'PUT /todos'` 和 `'DELETE /todos'` 需要自己实现外，其它的 sails 已经帮你做到了。

注意：本例中实现的 API 通过 `http://localhost:1337/todos{/todo_id}` 对外提供服务。前端部署在 `http://localhost:3000` 上，会被同源策略所限制，为了简单，本例中实现的 API 的所有 HTTP 方法对所有网站开放 (Access-Control-Allow-Origin: *)。


### TodoMVC with Server

替换在上一篇 [前端手记 TodoMVC 之 React 篇]({% post_url 2016-09-06-todomvc-react %}) 中实现的 TodoMVC Demo 的数据来源。使用 AJAX (Asynchronous Javascript and XML) 从 API 服务器请求数据，设置到 state 上触发 React 重绘。

数据从 API 获取之后，通过 props 层层传递到各个子 compoent。只修改了 `TodoApp` 这个顶级 component， 其它负责展示的 componnet 保持不变。 这种单向的数据传递方式，使得事情变得非常简单。

现在试试添加或编辑 todo 后，重新刷新界面，数据终于不丢失了。

本文所有代码都托管在 [zddhub/todomvc](https://github.com/zddhub/todomvc) 上, 你最好自己实现一遍，如果时间不允许，可以 checkout 代码到 bb1ecf17c7ab2835a55b2bd4cbae331bc208ec33 ，试试效果吧。

	cd server
	sails lift

	cd react
	node server.js # or open index.html


