---
layout: post
title: "PixelsMeasure 开发第二年总结"
category: Article
tags: "PixelsMeasure; iOS 开发；macOS 开发"
---

前年利用业余时间，开发了一款屏幕测量应用 —— [PixelsMeasure](https://apps.apple.com/app/pixelsmeasure/id1638740542)，搜索 PixelsMeasure 可以下载，同时支持 macOS 和 iOS。这篇文章总结一下今年的经历和感悟。

本文数据的统计时间从 07/22/2023 到 07/22/2024，和[开发第一年的总结](https://zddhub.com/article/2023/09/26/pixelsmeasure-development-summary.html) （从 07/22/2022 到 07/22/2023）做对比。

<!-- more -->

### 首先，它今年挣钱了吗？

直接上数据，总销售额为 $720, 预计收入为 $581，实际收到国内银行转账 4,064.59 元，这是所有收入。

![sales](/assets/images/2024-11-26/sales.png)

在扣除苹果税和各种税后，拿到了销售额的 80.69%。相比去年的 68.59% 有增长，应该是加了 App Store Small Business Program (主动申请才能加入)的缘故。

![proceeds](/assets/images/2024-11-26/Proceeds.png)

支出包括：
- 苹果开发者账号：688 元一年
- 域名续费两年：196 元，域名从去年的 69 元/年涨到了 98元/年，今年续了两年。
- ~~业余时间，不计入~~

**净收入**：4,064.59 - 688 - 196 = 3180.59 元。

### 时间成本

平均每周投入大概 4 小时，今年前半年在开发，后面中断了几个月，会续上。

### 今年的开发内容

今年将 PixelsMeasure 升级到了 3.0.0，最大的升级是应用内付费和跨平台支持（macOS 和 iOS），现在你可以在 Apple Store 搜索 PixelsMeasure 下载到手机端。

### 今年验证了那些观点？

#### 有没有必要支持 iOS？

iPhone + iPad 的销售占了总销售额的 37.91%，由于 PixelsMeasure 本身对 macOS 更友好，这个结果可以理解。

#### 有没有必要在国区发布？

PixelsMeasure 在中国大陆的销售占了总销售额的 1.25% （国区价格打七折），可以忽略不计。从今年开始，所有在国区上架的 App 都需要备案，至少对 PixelsMeasure 来说，完全没有必要。

#### 付费下载和应用内付费那个好？

应用内付费会更好。应用内付费给了消费者至少 7 天的时候去试用，对目标用户才有效，降低了退费率。

### 独立开发划得来吗？

如果是挣钱的话，铁定划不来。对技术提升的话，还是不错的选择。

### PixelsMeasure 还打算开发多久？

未来 2 年都会工作在它上。

### PixelsMeasure 未来开发计划

准备在未来几个月升级到 4.0.0，并支持应用内买断功能。
