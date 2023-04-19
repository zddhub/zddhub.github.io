---
layout: post
title: "到底应不应该自定义第三方库"
journal: '2023年15周'
---

最近尝试用了 Debug Memory Graph 和 Xcode Instruments，不用不知道，一用吓一跳。我发现 [PixelsMeasure](https://apps.apple.com/cn/app/pixelsmeasure/id1638740542) 第一次测量后，创建了 30 多万个对象，然后驻留在内存中。导致即使不测量时，内存也占用过大。一翻研究发现，是自己自定义了一个第三方库出的问题 😄，改好之后成功地修复了问题。所以如果自定义了内容，一定要经过充分的测试。

同时，也发现了一些内存泄漏的问题 🤦，也一并修复了。后面也许会出一个内存相关的文章，来总结这次的实践。

这次的教训是，得时常看看 Debug Memory Graph 和 Xcode Instruments，一个项目可不只是能用这么简单呀。