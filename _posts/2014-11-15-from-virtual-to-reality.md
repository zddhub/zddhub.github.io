---
layout: post
title: 从虚拟到现实
category: Fun
tags: OpenSSE LineDrawings
---

> 假如你吃了鸡蛋觉得不错，何必认识那只下蛋的母鸡呢？我是一个想弄清楚下蛋过程的人。

获悉[Mathias](http://cybertron.cg.tu-berlin.de/eitz/)创办(co-found)了 DISDAR，真心希望他能做出改变世界的产品来。优秀的工作总是令人着迷，在第一次看了SIGGRAPH上的演示视频后，我就告诉自己，我要知道它的全部。大约两年之后，我才实现了 Mathias 论文里的算法流程，并开源了草图检索引擎[OpenSSE](https://github.com/zddhub/opensse)。这是一个基础的框架，目前只对外提供了一种特征提取算法。希望草图检索不再神秘，更希望感兴趣的朋友能不断的改进，一起把它做成产品。

<!-- more -->

### 初因

大学期间，在看过一个精美的3D Demo 之后，我就开始学习OpenGL，选择了计算机图形学这条道路，走进了3D 虚拟世界。大学二年级的时候，被一个学长拉去做内网检索的项目，又了解了lucene和检索的相关知识。这两点都对我决定做草图检索起到了积极的影响。

### 框架

完整的草图检索引擎(更确切的说是OpenSSE)需要包含三个部分：草图获得，特征提取和排序算法。

![Overview](/assets/images/2014-11-15/overview.png)

### 草图获得

用于检索的草图主要来源于手绘和线画图提取。手绘的草图噪声大，不稳定，却是最真实的存在，在用识别类算法训练时，手绘草图具有先天优势。手绘草图收集困难，数量有限。更多的草图来源于三维模型，得益于[Szymon Rusinkiewicz](http://www.cs.princeton.edu/~smr/)等人的奉献，我写了一个Qt版本的[trianglemesh](https://github.com/zddhub/trianglemesh)，并重新定义了线画图。

![Line Drawing](/assets/images/2014-11-15/drawinglines.png)

### 特征提取

最先研究的特征提取算法就是SIFT，那是最痛苦的时候。理解完之后，发表第一篇技术博客[《SIFT算法详解》](http://blog.csdn.net/zddblog/article/details/7521424)，希望帮助初学者跨越David论文里的鸿沟。结果收到了很多鼓励，直到现在为止，还时不时的会收到大家交流的邮件和赞助，真的非常感谢！

### 排序算法

开源的OpenSSE在排序时，使用了最简单的余弦定理，新的版本会使用流行排序算法。

![Manifold Sort](/assets/images/2014-11-15/manifold_sort.png)

### 当前工作

基于OpenSSE，实现了一个[在线的草图识别](https://online.opensse.com)版本。受训练草图的限制，在准确率和本土化上都打了折扣。打算实现一个在线的草图收集共享平台，希望得到web开发者的支援。

![OpenSSE Online](/assets/images/2014-11-15/opensse_online.png)

### 相机原理

在处理三维相关工作时，一直有一架相机，杵在我的键盘上，屏幕前。最近对实际相机起了兴趣，正好也有机会，从虚拟转向现实，可以好好研究一下摄像机。感觉又为自己挖了一个坑, 哈哈。
