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

和 URL 资源的交互依赖 `URLSession` 来实现。通过 `URLSession` 创建一个或多个 `URLSessionTask` 子类的实例来实现数据的访问、下载、上传，和对流媒体以及 web socket的处理。

在创建 `URLSession` 时，可以设置 `URLSessionConfiguration` 来配置 `URLSession` 的行为和策略。通过 `URLSessionDelegate` 来检测 Session 的生命周期。

![URLSession](/assets/images/2022-05-04/url-loading-system.svg)


[URL_Loading_System]: https://developer.apple.com/documentation/foundation/url_loading_system
