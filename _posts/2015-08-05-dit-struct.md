---
layout: post
title: "独孤九剑 - 数据结构篇"
category: Fun
tags: "dit git"
---

> 穷则独善其身，达则兼济天下。一个人一旦能够喂饱自己并有个安身之所，剩下的就是为别人做点什么了。我现在所做的一切，还都是为了自己。

Dit 和 Git 使用相同的设计思路和数据结构，在表述上会混用两种，当出现差别时会特别说明。以下说明两种最基本的数据结构：文件系统和对象。

<!-- more -->

Dit 文件系统
-----------

Dit 和 Git 一样，存储文件快照，所谓快照，就是指文件本身(实际使用 zlib 压缩之后存储)，如果文件没有改变，那么只存储该文件的引用。你可以使用 `git init` 初始化一个 Git 仓库, 并查看生成的文件目录结构：

```sh
$ git init
Initialized empty Git repository in /Users/zdd/Projects/test_git_init/.git/
$ tree -a
.
└── .git # 在当前路径下生成 .git 目录，存放本地仓库
    ├── HEAD # 文件索引
    ├── config # 配置文件
    ├── description # 被 Git-Web程序引用，暂不使用
    ├── hooks # 钩子脚本，暂不使用
    │   ├── applypatch-msg.sample
    │   ├── commit-msg.sample
    │   ├── post-update.sample
    │   ├── pre-applypatch.sample
    │   ├── pre-commit.sample
    │   ├── pre-push.sample
    │   ├── pre-rebase.sample
    │   ├── prepare-commit-msg.sample
    │   └── update.sample
    ├── info # 暂不使用
    │   └── exclude
    ├── objects # 存储 Git 对象
    │   ├── info
    │   └── pack
    └── refs # 存放索引和 tags
        ├── heads
        └── tags

9 directories, 13 files
```

依样画葫芦，Dit 初始化仓库时也会创建9个目录，和4个文件（其中9个钩子脚本不生成）。初始版本我们只处理其中的几个文件：

```sh
# 永远只关心最重要的事
.
└── .git
    ├── HEAD
    ├── objects
    └── refs
        └── heads

3 directories, 1 file
```

文件创建使用 `package os` 来实现 (Dit 优选官方包，尽量不使用第三方库)。

Dit 基础类型
-----------

Dit 的基础类型有三种，blob，tree 和 commit。blob 管理文件，tree 管理目录，commit 管理每次提交。为了统一，抽象出一个公共类 object 。存储基本信息。