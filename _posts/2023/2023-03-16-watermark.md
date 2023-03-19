---
layout: post
title: "Web 数字水印探究"
category: Article
tags: "Watermark; Web 数字水印"
---

文心一言发布后，除了训练数据规模、训练结果和 ChatGPT 有差异外，界面上还多了水印。这篇文章主要探究文心一言使用的 Web 数字水印技术。

<!-- more -->

### 缘起

半年来，ChatGPT 3.5 彻底颠覆了人们对语言模型技术的认识，一时风光无两，瞬间碾压了 Web3、区块链和元宇宙。2023 年 3 月 14 日 ChatGPT 4.0 重磅发布，进化速度之快令人乍舌。百度也不甘寂寞，两天后发布了自家研发的知识增强大语言模型——文心一言。发布会上录屏演示，略显不自信，一时间股票暴跌 10%。原以为邀请码会遥遥无期，结果当天晚上就收到了短信，拿到了体验号。今天的故事就从这里开始。

体验了一番文心一言后发现，发布会上说的除了生成视频之外，基本可用。通过比较，效果和模型距 ChatGPT 还有很大差距，请飞速进化。其中，作画功能很有趣，让人忍不住有分享的冲动。比如下面这幅：

![ERNIE-Bot-draw-image](/assets/images/2023-03-16/ERNIE-Bot-draw-image.png)

刚要发朋友圈分享一下，突然发现背景模模糊糊的。凑近一看，一串代码，盲猜是邀请码。才反应过来，这是水印啊！一个体验的产品，要什么水印！要用来干什么？这是在防谁？(此处省略腹黑脑补一万字……)

分享一个带水印的图片，总觉得不安心。管它呢，删掉就可以了，这能难得到我！

### 文心一言界面水印去除

立马打开了 Console。

呵呵，嘴角微微上扬，这原来只是一段 DOM。

```html
<div id="eb-watermark" style="pointer-events: none !important; display: block !important">
    #shadow-root(open)
    <div id="mask_div_id00" style="display: block; ...">KFCVW50</div>
    ...
    <div id="mask_div_id100" style="display: block; ...">KFCVW50</div>
</div>
```

意外的是，它并不像我想的那么容易删除。以下是一些失败的尝试：

1. 删除 DIV 之后，又被加了回来。
2. 设置 `display: none !important;` 或者 `visibility: hidden !important;` 后, 又被加了回来。
3. 设置 `width: 0; height: 0;` 后，又被加了回来。
4. 编辑属性，或者编辑 HTML 后，又被加了回来。

这些尝试已经耗尽了我所有的前端知识储备。

又经过一番尝试，最后发现：DOM 删除和修改都不行，但是 `innerText` 是可以替换的。那么，去除文心一言水印的方法就来了:

打开 Console 运行：

```js
document.getElementById("eb-watermark").shadowRoot.childNodes.forEach(node => node.innerText = "")
```

只要不调整浏览器大小或者刷新，就能够保持住。这时，就可以安心地截图分享了。

### 这种水印是怎么做的？

我是老网民了，有空也会上网，是见过水印的。知道图片、文档在二进制中暗藏水印。但像这种 Web 上的水印还是头一次见，或者说是头一次吸引了我的注意。

那它到底是怎么实现的呢？HTML 没有魔法，要有，那一定是 JavaScript 干的。在 DOM 修改或删除后，又被重新添加了。那么是如何监听 DOM 修改和删除的呢？

常言道，外事不决问谷歌，内事不决问百度。人工智能时代，万事都问 ChatGPT。

> 问：如何监听 dom 的修改和删除

> ChatGPT：要监听 DOM 的修改和删除，可以使用 JavaScript 中的 MutationObserver 对象。MutationObserver 是一个异步 API，用于监视 DOM 树中的变化。当监视的元素或子元素的属性发生变化、添加或删除时，MutationObserver 会接收到通知并触发回调函数。
>
> 以下是一个简单的示例，演示如何使用 MutationObserver 来监听 DOM 的修改和删除：
>
> ```js
> // 选取需要监听的元素
> const targetNode = document.getElementById('target-element');
>
> // 创建一个 MutationObserver 对象
> const observer = new MutationObserver((mutationsList, observer) => {
>   // 遍历所有变化
>   for (const mutation of mutationsList) {
>     if (mutation.type === 'childList') {
>       // 监听子元素的添加或删除
>       console.log('A child node has been added or removed.');
>     } else if (mutation.type === 'attributes') {
>       // 监听属性的修改
>       console.log('The ' + mutation.attributeName + ' attribute was modified.');
>     }
>   }
> });
>
> // 配置观察选项
> const config = { attributes: true, childList: true, subtree: true };
>
> // 开始观察目标节点
> observer.observe(targetNode, config);
> ```

哦，原来答案是 `MutationObserver`，监听到 DOM 变化后删除它，再添加一个新的，就能实现文心一言的水印效果了。

什么？你也想问文心一言这个问题？好的！

![DOM-Observer](/assets/images/2023-03-16/ERNIE-Bot-answer-Screenshot-2023-03-17-at%2021.06.29.png)
*注：AI 随时在进化，这是文心一言发布当天的测试结果。*

此时此刻，我想吟诗一首：

> 手持两把锟斤拷
>
> 口中疾呼烫烫烫
>
> 脚踏千朵屯屯屯
>
> 笑看万物锘锘锘

哎，希望进化的快一点吧。

### 堵住文心一言水印的漏洞

上面提到，修改 `innerText` 可以去掉文心一言的水印，那么 `innerText` 的修改能监听到吗？要是有个配置就好了，再问 ChatGPT：

> 问：Config 能配置监听 innerText 的修改和删除吗？

> ChatGPT：是的，MutationObserver 的观察选项（config）可以配置监听 innerText 的修改和删除。具体来说，可以通过设置观察选项的 characterData 属性为 true 来仅监听文本内容的变化，而忽略元素节点的增加、删除、属性修改等其他变化。
>
> 设置观察选项的 characterData 属性为 true，表示仅监听节点的文本内容变化，而忽略其他类型的变化，如节点添加、删除、属性修改等。当目标节点的 innerText 属性发生变化时，MutationObserver 会调用回调函数，并将变化信息传递给它。

是的，可以通过设置 `const config = { characterData: true, ...};` 来监听 `innerText` 的改变。

齐活儿，漏洞堵上了!

咦，怎么突然就从人工智能跳到水印了呢？不说了，我要赶紧去学习了，要不焦虑感就会一直留在我的心里，就像那无法去除的水印一样。