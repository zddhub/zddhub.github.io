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
└── .dit
    ├── HEAD
    ├── objects
    └── refs
        └── heads

3 directories, 1 file
```

文件创建使用 `package os` 来实现 (Dit 优选官方包，尽量不使用第三方库)。

Dit 基础类型
-----------

Dit 的基础类型有四种，blob，tree，commit 和 tag。blob 管理文件，tree 管理目录，commit 管理每次提交，tag 设置标签。为了统一，抽象出一个公共类 object，存储基本信息。

### blob

blob 记录项目文件快照。实际文件通过 [zlib](http://www.zlib.net/) 压缩后存储在本地仓库的 objects 目录下。快照文件目录为其内容 sha1 值的前两位，文件名为之后的38位。Dit 按内容存储文件快照，内容不同(sha1 值不同)的版本，对应不同的快照文件。

![blob]({{ post.img | replace: '..', site.url }}/asserts/images/2015-08-05/blob.png)

假如需要添加一个内容为 dit 的文件 a 到 Dit 仓库，则实际在 objects 中生成的快照为：

```sh
$ tree .dit/objects/
.dit/objects/
├── 8f
│   └── 2c96ad676d7423d2c319fffb78cfb87c78c3e2
.
```

blob 文件结构为 `"blob size\0实际内容"`，其中 size 为文件所占用的字节数。sha1 值按以下算法计算:

```sh
# 利用 git 命令计算
$ echo dit | git hash-object --stdin
8f2c96ad676d7423d2c319fffb78cfb87c78c3e2

# 利用 shasum 计算
$ echo -e "blob 4\0dit" | shasum
8f2c96ad676d7423d2c319fffb78cfb87c78c3e2  -

# 利用 openssl sha1计算
echo -e "blob 4\0dit" | openssl sha1
(stdin)= 8f2c96ad676d7423d2c319fffb78cfb87c78c3e2

# go 的实现: 我是试了很久才想到 \x00 的
fmt.Printf("%x\n", sha1.Sum([]byte("blob 4\x00dit\n")))
```

### tree

只有 blob 会保存文件的实际内容，tree 引用 blob 或者 tree，每个对象占一行。以下是某个tree的内容：

```sh
$ git cat-file -p af3d9d135f68fbec7e4ea7fde3ef19844746ae6f
100644 blob f0462300828522d14f9faeb75d80f66fa89457ae  blob.go
100644 blob 1b022329b3817b54c37e6704b7b9e4df49d8dee9  commit.go
040000 tree af8db5dbe85bc5f695ab3ddc1ad7a5fcf6617ae9  main
100644 blob 523a6539be6dc1a1d188965683091ce37e2f898e  object.go
```

同样加上头 `"tree size\0"` 后计算 sha1 值，并用 zlib 压缩后存储在 .dit/objects 目录下。

![blob]({{ post.img | replace: '..', site.url }}/asserts/images/2015-08-05/tree.png)

### commit

commit 只引用 tree, 每次提交时的文件信息都由 tree 来管理，commit 通过引用不同的 tree 来标记不同的版本。以下是某个 commit 的内容:

```sh
$ git cat-file -p 1029cd09aa6c72f142931d7cd0cb3fef7486ca14
tree 42477c2be645032c4dc8699fa4fa8acfcbc633af
author zdd <zddhub@gmail.com> 1439825985 +0800
committer zdd <zddhub@gmail.com> 1439825985 +0800

Add a

second line
```

加头 `"commit size\0"` 后计算 sha1 值，并用 zlib 压缩后存储在 .dit/objects 目录下。

![blob]({{ post.img | replace: '..', site.url }}/asserts/images/2015-08-05/commit.png)

### tag
TBD

缓存
---

Dit 在缓存 (cache) 中记录当前仓库的状态，缓存的状态存储在 .git/index 文件中，来保证不同命令之间信息的传递。缓存是 Dit 设计的主要思想, 搞清了它的数据结构和运作模式，Dit 的实现思路也就呼之欲出了。