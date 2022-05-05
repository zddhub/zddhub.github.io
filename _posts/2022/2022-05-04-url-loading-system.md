---
layout: post
title: "URL 加载系统（URL Loading System）"
category: Note
tags: "URL Loading System, URLSession, Swift, iOS"
---

[URL Loading System][URL_Loading_System]，通过 URLs，使用标准的网络协议，与服务器交换数据。

<!-- more -->

### 简介

iOS 应用开发少不了和服务器打交道，而 URL 加载系统用来加载 http- 和 URL-based 的资源。这种加载通常是异步的，异步的机制能使得你的应用在数据或者错误回来之前，仍然能响应。

和 URL 资源的交互依赖 `URLSession` 来实现。通过 `URLSession` 创建一个或多个 `URLSessionTask` 子类的实例来实现数据的访问、下载、上传，和对流媒体以及 web socket 的处理。

在创建 `URLSession` 时，可以设置 `URLSessionConfiguration` 来配置 `URLSession` 的行为和策略。通过 `URLSessionDelegate` 来检测 Session 的生命周期。

<img src="/assets/images/2022-05-04/url-loading-system.svg" width="100%" min-height="500px">

*注：上图是 svg 图片，可放大后观看*

### URLSession

`URLSession` 是整个 URL 加载系统的核心，创建 Session 后，通过方法创建 task，然后执行 task 来实现和后端服务的通信。虽然叫做加载系统 (loading system)，但是也支持写数据到服务器。一个应用程序可以有多个 Session，每个 session 可以创建多个 task。应用可以根据需要，尽可能少的使用 Session，来减少对资源的消耗。一般来说，session 可分为以下三类：

- 常规的 Session。使用 `URLSession` 的构造函数创建，使用 configuration 配置 Session 的权限和策略，使用 delegate 来监听 Session 的生命周期。
- 简单的 Session。使用默认的配置，不对配置做更改。或者直接使用 URLSession shared 实例，能满足大部分的需要。
- 后台 Session。需要配置，当应用挂起时，能在后台下载或者上传。

### URLSessionTask

`URLSession` 提供了多套 API 来创建不同的 `URLSessionTask`, 来执行和服务器交互的任务。`URLSession` 创建的 task 属于挂起状态，需要调用 `resume()` 方法来执行。从目前的 API (iOS 15) 来看, 有 5 类 Session Task：

- `URLSessionDataTask`，最常用的一种，dataTask 返回 Data 类型的数据，存在内存中。不能在后台运行。
- `URLSessionUploadTask`，能方便的设置 request body，上传到服务器。能在后台运行。
- `URLSessionDownloadTask`，下载内容到临时文件，支持后台运行。
- `URLSessionStreamTask`，支持流媒体
- `URLSessionWebSocketTask`，支持 WebSocket

每种 task 提供了多种使用方式（为什么要这样？Python 告诉我们解决问题最直接的方法应该有一种，最好只有一种），如下表所示：

| Task type                | Async API | Completion Handler API | Combine API | Normal API |
|:-|:-|:-|:-|:-|
|`URLSessionDataTask`      | ✅ | ✅ | ✅ | ✅ |
|`URLSessionUploadTask`    | ✅ | ✅ | - | ✅ |
|`URLSessionDownloadTask`  | ✅ | ✅ | - | ✅ |
|`URLSessionStreamTask`    | - | - | - | ✅ |
|`URLSessionWebSocketTask` | - | - | - | ✅ |

其中，

- Async API: 使用 `async/await` 关键字来调用 API
- Completion Handler API: 使用回调的方式，在 completionHandler 中处理结果
- Combine API: 使用 Combine publisher 来获取结果
- Normal API：只创建对应的 task，需要使用 delegate 获取结果

使用时可根据需要选择合适的 API，个人倾向于 Async API > Combine API >  Completion Handler API > Normal API。

[URL_Loading_System]: https://developer.apple.com/documentation/foundation/url_loading_system
