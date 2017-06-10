---
layout: post
title: "Gem-based Jekyll theme 开发小记"
category: Fun
tags: "Jekyll theme gravid"
---

Github pages 和 Jekyll 搭配，让一大批静态博客火了起来，方便了不少人，用 `markdown` 写文章也非常优雅。

<!-- more -->

美中不足的是，Jekyll 把 theme 和博文混在了一起。把博客内容和 theme 分开，让内容写作者只关注内容本身，我觉得这才是一种最好的体验。我最理想的写作平台是只写 `markdown` 外加一个 `assets`（用来保存文章中的素材）。Jekyll 引入的 `Gem-based theme` 几乎完美的解决了这一问题。

Jekyll 在 3.2.0 版本引入了 `Gem-based theme`。 把网站的样式封装成 gem 包，整个静态博客离理想只差一个 `Gemfile`。看到这个，加上我对前博客臃肿的样式，缓慢的速度已经容忍到了极限，顿时觉得干劲十足，那就行动吧。


名字
---

首先它得有个名字，叫什么呢？对我来说起名字就像公鸡下蛋，憋的再久，也下不出来。三步成诗、七步成文，这是需要才气的。钱老先生说怀才就像怀孕，时间久了才能看得出来。我已经快熬到了老先生写《围城》的年纪，也没发现自己有什么才气。既然不能 `being talented`，那就退而求其次， `being gravid` 吧。

> Being talented just like being gravid , must be known with a long time past.

所以，一个用形容词做名字的 Jekyll theme 诞生了：[gravid](https://github.com/zddhub/gravid)。


样式
---

博客应该长什么样子？首先应该便于阅读，其次应该足够简单。[小鱼同学](https://github.com/sofish) 的 [typo.css](https://typo.sofi.sh/) 一直是我认为的最完美的中文排版样式，[gravid](https://github.com/zddhub/gravid) 的样式，就全照搬他的[博客](https://sofi.sh/)了。


烦恼
---

开发出奇的顺利，并且 push 了生平第一个 [gem](https://rubygems.org/gems/gravid)，暗喜。本地运行完美，添加之前的博客，push 到 Github pages 上，静待成功喜悦。几秒后收到一份邮件：

```
You are attempting to use a Jekyll theme, "gravid", which is not supported by GitHub Pages. Please visit https://pages.github.com/themes/ for a list of supported themes. If you are using the "theme" configuration variable for something other than a Jekyll theme, we recommend you rename this variable throughout your site. For more information, see https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/.
```

脑后一凉，仔细一看，Github pages 只[支持有限的几个 theme](https://pages.github.com/themes/): [Architect, Cayman, Dinky, Hacker, Leap day, Merlot, Midnight, Minima, Minimal, Modernist, Slate, Tactile, Time machine]，有黑幕呀。


求助 Github
----------

如何才能加入到这个 supported list 里呢？找 Github 试试看吧。向 Github support 请教了一把，一个小时不到就收到了回复（北京时间上午11:46），我万万没想到这么及时。顿时对 Github 肃然起敬，好感多了不少。

![Github reply](/assets/images/2017-06-10/github-reply.png)

结果在意料之中，回复非常得体，又非常专业，切中要害，连一句废话都没有。即成功了拒绝了我的妄想，又贴心的提供了两个备选方案。顿时对 Github 的程序员肃然起敬，好感多了不少。


折中的方案
--------

我从一开始就反感混合 theme 和博文做法，又偏执的认为代码仓库只应该托管代码，而不应该托管中间产物。所以对 Github 给出的两个方案，我只能会心一笑，采用 `git submodule` 来曲线救国了。详细的做法在[这里](https://github.com/zddhub/zddhub.github.io)。

中间遇到了一个坑，Jekyll 限定 `_includes` 和它的子文件不能是链接文件，解决方案在[这里](https://help.github.com/articles/page-build-failed-file-is-a-symlink/)。

为了方便使用，封装了一个命令，每次写完文章，或者更新完 theme 之后，执行 `./publish_blog` 就可以把文章发表在 Github pages 上啦！


总结
---

这次更新了一个新的 Github theme，并且尝试了一下 gem，感觉程序员的世界会越来越美好！

Gem-based 已经实现了博文和 theme 的分离，我想，除了是亲生的几个 theme，可能也是出于安全和资源两方面考虑，Github 暂时不支持自定义的 Gem-based theme。

把博客和 theme 通过 `submodule` 的方法加到 Github pages 中，能专注的更新 blog 或者 theme，解决了一部分问题。但提交博文后需要手动（或者自动）在 Github pages 上 push 一次。

期待 Github pages 早点支持自定义 theme!

这已经是我第三次更新个人博客了，下一次会在什么时候呢？