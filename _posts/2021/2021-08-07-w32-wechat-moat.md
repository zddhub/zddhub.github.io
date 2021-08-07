---
layout: post
title: "微信的护城河"
journal: '2021年32周'
---

今天更新了一波博客，给加上了 SEO Tag，把 [gravid](https://rubygems.org/gems/gravid) 升级了一个版本。这个主题开发了 4 年多，截止今天为止，被下载了 15436 次。

又纠结于微信分享时缩略图丢失问题，浪费了很多时间。微信居然提供了 [JS-SDK](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html) 专门做这件事，而没有使用通用的解决方案，比如 Open graph Metadata `og:image`，真的是将围城进行到底。支持通用的解决方案，对腾讯能有什么影响呢？这种给别人找麻烦的公司，应该被用户尽早的淘汰掉。对于这种纯静态的博客来说，签名是个问题。最后看到这个图释然了，为啥我要支持呢？

![Share Apple to WeChat](/assets/images/2021-08-07/share-apple-to-wechat.png)
