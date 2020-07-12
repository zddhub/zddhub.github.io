---
layout: post
title: "学习一个新命令需要多久?"
category: Mics
tags: "VS Code; VS Code extensions; Change language mode; Format Document"
---

> 你还记得最近学会的新命令，一共花了多久吗？这里记录了我学习 `Change language mode` 的路线图，总共花了5个小时。

<!-- more -->

### 问题发生的背景

在大公司待过的人一定非常能理解，跟日志打交道的时间远远大于写代码的时间。很长的一段时间里，我都是在 Jira 和 log 里度过的。因为经常要从日志里复制出一段 json，通过它来找到程序的 bug。但不便利的是，每次从日志里拿到的 json 是没有格式化的，如果想格式化（我用 `VS Code` ），必须需要做以下几步：

```
痛苦的步骤：
新建一个标签 -> 复制内容到新标签 -> 保存为 .json 文件 -> 格式化 -> 用完后删除文件
```

通常来说，拿到一段 log，格式化后看上几个关键字段的值，接着就不用了。按上面的流程，总是要保存文件，并且几分钟后又要删除它（尤其对我这种强迫症患者），很难受。

### 需要解决的问题

为什么要保存立马要删除的文件呢？我理想的流程应该像下面这样子：

```
新建一个标签 -> 复制内容到新标签 -> 格式化 -> 用完后关闭标签
```

首先去掉保存文件的步骤，其次，关闭标签比删除文件个人觉得省力不少。那么，要解决的问题是，如何在不保存文件的情况下，格式化它呢？

### 现实很骨感

用 vscode 的人应该都知道，新建的标签，在没有命名时，会有个默认的名字，叫做 `Untitled-1`(..`Untitled-n`), 它是一个文本文件(`plaintext`)。如果你在这个时候运行 `Format Document`, VS Code 会非常温馨的提示你 `There is no formatter for 'plaintext' files installed.`, 希望你安装插件。

![Format-Untitled-File](/assets/images/2020-07-12/Format-Untitled-File.png)

### 解决方案

1. 寻找插件

按照 VS Code 的提示，最便捷的方案就是寻找并安装插件，如果能有人遇到同样的问题，并且解决了的话, 那安装个插件是最简单的招数了。

一番寻觅，也安装了若干插件，结果都没能解决我的问题。我想，没有人有想我这么变态的需求吧。

2. 开发一款自己的插件

在**静默了一周**之后，我终于忍不了了。像这种问题，如果你没有觉得它有问题，大可创建一个 `json` 文件，专门用来格式化 json，如果一但发现有问题，在使用时就会一遍遍的抗拒，直到非解决不可为止。

自己开发一个格式化的插件有技术含量吗？VS Code 已经有强大的 `Format Document` 了，按照它运行的流程可以想见，`Format Document` 先通过文件后缀名判断语言，然后根据特定语言进行格式化操作。那么我们要做的只是把判断语言的操作给替换掉就可以了。VS Code 是开源的，格式化的代码应该也能找见。我理想的命令是这样的 `Format JSON`格式化 json，`Format JavaScript`, 格式化 JavaScript, ...。说干就干。

因为是第一次开发 VS Code 插件，照着[官网](https://code.visualstudio.com/api/get-started/your-first-extension)先来了个 HelloWorld，大概学了一下 VS Code 的基本概念和目录结构，大概花了**20分钟**。

学完 Hello World，命令和架子都打好了，现在需要的就是填入格式化代码的部分了，搜了一下 VS Code 源码，看了 [`FormatDocumentAction`](https://github.com/microsoft/vscode/blob/846aec810ede01f4c262cbffe59a7ac7e33c5b4b/src/vs/editor/contrib/format/formatActions.ts#L209)的实现，确实有些复杂，直接调用的可能性不大，如果复制一份到插件里，VS Code 的更新不能及时同步到插件里，头大。

在 FormatDocumentAction 里最大的发现就是它的 id [`editor.action.formatDocument`](https://github.com/microsoft/vscode/blob/846aec810ede01f4c262cbffe59a7ac7e33c5b4b/src/vs/editor/contrib/format/formatActions.ts#L213), 如果能直接用它调 format document 是再好不过了，搜了一下 API 文档，还真的找到了一个命令 `vscode.commands.executeCommand`，写到 Helloworld里试了一把，还真的可以。这一过程大概又花了**半个小时**。

现在的问题是如何把语言的信息告诉 vscode，又继续搜了一下 API，找到了一个只读属性，`languageId`, 文档是这样描述的 `The identifier of the language associated with this document`。赶紧试了一下，untitled 文件输出的 `languageId=plaintext`, JSON 文件的 `languageId=json`。是它，真的是它。一分钟后我就找到了设置它的方法 `setTextDocumentLanguage`。到此为止，我的七巧板凑齐了。这个过程大概花了**十分钟**。

接下来就简单了，把七巧板组装一下，一个私人定制的 VS Code 插件就出来了。源码在[这里](https://github.com/zddhub/format-code)，有兴趣的同学可以去看看。为了能在公司的电脑上使用，我当然是选择发布在 [Marketplace](https://marketplace.visualstudio.com/items?itemName=zddhub.format-code)上了,可以去[下载](https://marketplace.visualstudio.com/items?itemName=zddhub.format-code)，或者在插件里搜索 `format-code` 找到它。发布的过程很顺滑，官网的文档写的真的是相当的好呀。

其实可以支持更多的语言，但是懒人如我，只支持了自己常用的几种，在命令面板输入 `Format JSON` 就可以格式化 JSON 文件了。

![Format-Code](/assets/images/2020-07-12/Format-Code.png)

至此，我的问题解决了，整个人都舒服多了。

3. 遇到新命令

从写代码到发布，大概用了**一个小时**，之后完善了一下 Readme，又读了读文档，发现一篇文章写的很不错，名字叫 [Creating a Formatter Extension](https://code.visualstudio.com/blogs/2016/11/15/formatters-best-practices)，里面讲到一些开发 Format 类插件的最佳时间，其中最重要的一条就是，更多的使用 `Format Document` 和 `Format Selection` 这样通用的命令，尤其应该避免使用像 `Format Foo File.` 这种名字。

如果不用 `Format JSON` 这样的命令的话，我的插件应该只设置 `languageId`，然后再调用 `Format Document`格式化代码。等等，难道 VS Code 不能设置语言？

![what-the-fuck](/assets/images/2020-07-12/what-the-fuck.jpg)

靠，还真有呀!!!

`Change language mode`

喜人的名字加上好看的 language list。


### 写在最后

比较一下两种方案：

- 使用插件：CMD+SHIFT+P -> 输入 Format JSON -> 格式化完成
- 使用原生命令: CMD+SHIFT+P -> Change language mode -> 输入JSON选择语言 -> 输入 Format Document -> 格式化完成

从流程上，插件还是短了一截的，我是一定会用的，你呢？