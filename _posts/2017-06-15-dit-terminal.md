---
layout: post
title: "独孤九剑(0x05) - 终端篇"
category: Fun
tags: "dit git"
---

> 据说一个合格的程序员看终端的时间会比看女友的时间还多。

想开发一个受 类*nux 程序员普遍欢迎的程序，你就需要增加对终端的友好支持，包括良好的帮助文档、提示、颜色和自动补全。

<!-- more -->


# 创建自己的 man page

每当拿到一个新的命令，除了使用 `-h` 和 `--help` 查看用法之外，类*nux 用户还会自然而然的运行一个 `man` 命令，查看更详细的使用手册。

### Man page 内容

完整的 `man` 文档，一般都包含以下几部分：

```sh
NAME
SYNOPSIS
DESCRIPTION
OPTIONS
BUGS
AUTHOR
SEE ALSO
```

这 7 个部分几乎出现在了 90% 的系统文档中，你可能会看到其他的一些部分，但这 7 个是最常用的。

`NAME`: 程序的名称和一个简单的描述，例如：git - the stupid content tracker

`SYNOPSIS`: 程序的用法，描述你如何运行这个程序

`DESCRIPTION`: 更详细的描述你的程序

`OPTIONS`: 解释命令中的参数

`BUGS`: 如果你程序中存在 bug，或在哪种情况下会出现 bug，可以在这里告诉用户。比如 ls，就很诚恳的告诉我们:

```sh
BUGS
     To maintain backward compatibility, the relationships between the many options are quite complex.
```

`AUTHOR`: 程序的作者和邮箱地址

`SEE ALSO`: 链接与你程序相关的其它程序，帮助用户更加了解你的程序

### 撰写自己的 Man page

使用 `man` 去查看帮助文档时，它会调用一个命令叫 groff(GNU troff), `groff` 把标准的文本和特殊的命令翻译成格式化的输出，就像你在 `man` 手册页里看到的那样。

`groff` 为解析 man pages 定义了一批宏(详细列表参见[Groff 4.1.2 Usage](https://www.gnu.org/software/groff/manual/groff.txt)), 这些宏以 `.` 开始，如：

表示文档标题的宏：
```txt
.TH [name of program] [section number] [center footer] [left footer] [center header]
```

表示文档每部分标题的宏：
```txt
.SH [section name]
```

其它的不再详述，详细请看[groff 文档](https://www.gnu.org/software/groff/)。以下是 `dit` 的一部分 man page 内容：

```txt
.TH DIT 1 06/15/2017 "DIT 0.1" "DIT Manual"

.SH NAME
dit - the stupid content tracker

.SH SYNOPSIS

dit [--version] <command> [<args>]

.SH DESCRIPTION
Dit is a fast, scalable, distributed revision control system with an unusually rich command set that provides both high-level operations and full access
to internals.

.SH OPTIONS
--version Prints the Dit suite version that the dit program came from.
...
```

保存成 `dit.1`，可使用 `man ./dit.1` 查看格式化后的结果，或者将该文件复制到 `/usr/share/man/man1/` 目录下，使用 `man dit` 查看。

在实际项目中，通常先写成 markdown 文件，再通过工具加上特定的宏，最后用 `Makefile` 等工具将文档复制到对应的 `man` 目录下，完成 manual page 的构建。如果你仔细观察 `/usr/share/man`目录:

```sh
zddhub$ ls
man1   man2   man3   man4   man5   man6   man7   man8   man9
```

就会发现，这里有 9 个不同的 man 目录。这是因为 类*nux 把系统文档分成了 9 类，每个数字代表的含义如下：

```txt
1   Executable programs or shell commands
2   System calls (functions provided by the kernel)
3   Library calls (functions within program libraries)
4   Special files (usually found in /dev)
5   File formats and conventions eg /etc/passwd
6   Games
7   Miscellaneous (including macro packages and conventions), e.g.: man(7), groff(7)
8   System administration commands (usually only for root)
9   Kernel routines [Non standard]
```

在撰写自己的 man pages 时，也应该遵循这种分类。


# 终端颜色输出重定向

命令输出时，可加上颜色显示，让输出信息更加友好。

如下：

```sh
$ printf "\033[41;32m字体背景是红色，字是绿色\033[0m\n"
```
这种功能适用于任何形式的printf函数，或者变种。

其中4表示背景色，3表示前景色。颜色编码：
| 0 － black | 1 - red | 2 - green | 3 - yellow | 4 - blue | 5 - purple | 6 - cyan | 7 - white |

当将带颜色的命令重定向到文件时，会打印颜色字符：

```sh
$ printf "\033[41;32m字体背景是红色，字是绿色\033[0m\n" > a.log
$ vi a.log
^[[41;32m字体背景是红色，字是绿色^[[0m
```

出现乱码，非常不爽。目前的解决方案是，借助命令行传入全局参数，来打开或者关闭颜色的输出。

我的另一篇博客里有 Go 对终端颜色的实现,如果感兴趣请移步[这里](http://www.zddhub.com/knowledge/2015/07/23/terminal-color)。