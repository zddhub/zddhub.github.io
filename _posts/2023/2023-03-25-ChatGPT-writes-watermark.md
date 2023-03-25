---
layout: post
title: "ChatGPT 编程实现 Web 数字水印"
category: Article
tags: "Watermark; Web 数字水印; ChatGPT 编程"
---

文心一言发布后，界面的水印技术引起了我极大的兴趣。再加上想体验 ChatGPT 写代码的能力，所以借助它的能力，编程实现了类似文心一言界面的 Web 数字水印，并部署在了[我的网站上](https://zddhub.com/watermark/)。

<!-- more -->

在做这件事之前，定下了三个目标：

1. 只写描述，让 ChatGPT 写代码。
2. 不使用搜索引擎，例如谷歌，stackoverflow 等。
3. 录屏记录整个过程。

结果，ChatGPT 出色地完成了任务，大概写了 98% 的代码。其中跨文件的导入部分，由我来完成。整个过程都没有使用过搜索引擎。

整个过程也被我记录下来了，上传在了哔哩哔哩上。

[![Web 数字水印视频截图](/assets/images/2023-03-25/watermark-bilibli-video.png)](https://www.bilibili.com/video/BV1Ak4y147t6)

*注：在视频制作一周内，文心一言的水印技术也在不断进化，其中 div id 不再使用 `eb-watermark`。但是在我自己部署的网站上，仍然保留原 id。*

视频地址：https://www.bilibili.com/video/BV1Ak4y147t6
网站地址：https://zddhub.com/watermark/
代码地址：https://github.com/zddhub/watermark

### 视频内容简介

视频一共分为 6 部分：

1. 简介
2. 所用工具介绍
3. 实现水印效果
4. 水印防删除增强
5. 重构代码
6. 部署到 Github Page

### 心得

ChatGPT 极大地简化了开发时间，提升了开发效率，在多数情况下，几乎没有错误，值得拥抱。