---
layout: post
title: "前端手记 TodoMVC 之 CSS 篇"
category: Memo
tags: "TodoMVC 前端手记"
---

我一直以后端程序员自居，从不轻易涉足前端。有人说过，一个人最热爱的就是他所擅长的事。我没有设计师的品味和 UX 的眼力，对前端有着莫名的恐惧。这次由于工作原因，需要系统地学习前端知识，对我来说也是不小的挑战。

<!-- more -->

记得上周临时给同事们分享我的图形学知识，零零散散地讲了一大堆，还是觉得没讲到重点。特别是专业的词汇和概念，都忘得想不起来了，有种“只可意会，不可言传”的即视感，又没法从头念一遍计算机图形学再回来继续分享，非常尴尬。这次学前端，已经预见到会从头再来。不过这次学乖了，我要开始做笔记。


### TodoMVC Demo

尤其最近两年，前端百花齐放，各种框架层次不穷。虽然不在前端混，但也听过很多传说，像 Angular, React, Vue, ..., 等等。对于我等初学者到底该如何开始呢？万能的 github 上有个叫 [TodoMVC](https://github.com/tastejs/todomvc) 的项目，用不同的框架实现了一个简单的业务 —— Todo list，是居家学习前端的首选。相同的业务，不同的实现，有助于比较各种框架实现的异同，掌握其中精髓。


### css 是第一课

刚 clone 到 TodoMVC 的时候，我是蒙逼的。看着一坨坨的代码，不知道该从哪一句开始写，先从哪一个开始学。在这个问题上，我几乎花了一个月时间。最后发现，从 css 入手或许是一个很好的切入点，本篇会手写 TodoMVC 的前端页面来学习一下 css，整体上会强调开发流程而少基础知识描述。

在开始之前，花一个番茄工作时来学习一下 [html5](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/HTML5/HTML5_element_list) 和 [css](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started) 初级向导吧。

你还得再花一个番茄时间来学习 [css 布局](http://learnlayout.com/)，这个真的很重要，我在这里等你回来。

除了这些，你还得知道：

* css 选择器优先级：important > [ id > class > tag ]；最近的祖先样式比其他祖先样式优先级高。“直接样式”比“祖先样式”优先级高。
* 后面的 html 会覆盖前面的
* input-placeholder
* box-shadow

好了，囊中有了这些积累，我们就可以开始练手了。


## 第一个版本

### 第一步，做个 mockup 图

在开始 coding 前想好要做什么是很重要的，找个 UX 姐姐或者自己画吧。下图是我根据 [TodoMVC](http://todomvc.com/examples/react/#/) 的 demo 画的，当没有任何 todo 项时，只显示个输入框，非常简单。下面就开始动手实现吧。

![todomvc new-todo mockup]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/todoMVC_mockup_new_todo.png)


### 第二步，写 html 页面

对 mockup 进行划分，识别其中的元素，开始写 html 标签。以下是我的一个划分：

![todomvc new-todo mockup]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/todomvc_mockup_new_todo_html.png)

从文中的图片名能找到对应的 commit id，进而看到代码哦，代码托管在 [zddhub/todomvc](https://github.com/zddhub/todomvc) 上。

为了方便 css 选取，给主要标签都加了 class，这时的界面是如下所示的丑样子：

![todomvc new-todo html]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/7d080efa0a1528984616f038b901331224cfa0ff.png)


### 第三步，写 css 样式

通常这一步都不是一下子就能完成的，css 的语法虽然简单，但通常写起来会让人抓狂，除非你精通它。我尝试按以下步骤来完成一个网页的样式编写：

* 给元素定位（包括：字体，背景色等）

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/0da9805174394731428e8021976366988895d129.png)

* 调整每个元素的细节

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/a98d0ebb9f8e419a208876ea57936beb08dab9cc.png)

* 设置颜色和阴影

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/9b684ea31ad383ffcaddc91a7c059169658cc19e.png)

* 响应式布局

太简单没有发现问题

* 多浏览器支持

太简单暂时没发现问题

到目前为止，我们已经完成了简单的 new todo 界面。


## 第二个版本


### 第一步，做个 mockup 图

当存在 todo 时的界面

![todomvc new-todo mockup]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/todoMCV_mockup_todo_list.png)


### 第二步，写 html 页面

界面再一次变成如下所示的丑样子：

![todomvc new-todo html]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/5bc3f67db802da5c8b17ad1abd36f47974e40abb.png)


### 第三步，写 css 样式


* 给元素定位（包括：字体，背景色）

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/d051b7f4849815ddece4ee53bdfcb67ee16e9977.png)


* 调整每个元素的细节

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/96976034116027095374fb748b2e4644d6502f36.png)


* 设置颜色和阴影

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/0770d34cd9664268bf6c36c4e5961e6831733663.png)


* 设置动态效果

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/d3d6da9aee6cc74cff3d5feda1577bde87467b58.png)


* 响应式布局

太简单没有发现问题


* 多浏览器支持

我们的代码在最新版的 chrome 和 safari 上运行完美，在 firefox 上 checkbox 挂逼了。如下：

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/todoMVC_list_on_firefox.png)

稍微改了一下，在 firefox 上最终效果如下。

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/7eb2746852d08858d99228e7ce8883e77685c3f9.png)

追求完美的脚步是无止境的，但我们的 demo 就实现到这里，哈哈哈, 以下是最终的效果：

![todomvc new-todo css position]({{ post.img | replace: '..', site.url }}/asserts/images/2016-09-04/d3d6da9aee6cc74cff3d5feda1577bde87467b58.png)


所有代码都托管在 [zddhub/todomvc](https://github.com/zddhub/todomvc) 上，可以根据图片名字的 id checkout 代码，查看具体的实现。


### 和神作对比

这里有 TodoMVC 官方使用的 css 项目：[tastejs/todomvc-app-css](https://github.com/tastejs/todomvc-app-css)，比我多了 170 行代码，在 firefox 上的表现也优于我，取长补短吧。


好了，看得懂并且会写 css 是深入学习前端的基础。

欢迎来到前端新手村！胜败乃兵家常事，大侠请重新来过！
