---
layout: post
title: "终端输出颜色"
category: Knowledge
tags: "terminal color"
---

在用printf输出时，可加上颜色显示，让输出信息更加友好。
如下：

```sh
$ printf "\033[41;32m字体背景是红色，字是绿色\033[0m\n"
```
这种功能适用于任何形式的printf函数，或者变种。

其中4表示背景色，3表示前景色。颜色编码：
| 0 － black | 1 - red | 2 - green | 3 - yellow | 4 - blue | 5 - purple | 6 - cyan | 7 - while |

<!-- more -->

当将带颜色的命令重定向到文件时，会打印颜色信息：

```sh
$ printf "\033[41;32m字体背景是红色，字是绿色\033[0m\n" > a.log
$ vi a.log
^[[41;32m字体背景是红色，字是绿色^[[0m
```

出现乱码，非常不爽。目前的解决方案是，借助命令行传入全局参数，来打开或者关闭颜色的输出。

以下是Go的一种实现。

```go
package main

import (
  "flag"
  "fmt"
  "github.com/fatih/color"   //color 包对颜色输出进行了封装
)

func main() {
  var flagNoColor = flag.Bool("no-color", false, "Disable color output")
  flag.Parse()
  if *flagNoColor {
    color.NoColor = true // disables colorized output
  }

  // Print with default helper functions
  color.Cyan("Prints text in cyan.")

  // A newline will be appended automatically
  color.Blue("Prints %s in blue.", "text")

  // These are using the default foreground colors
  color.Red("We have red")
  color.Magenta("And many others ..")
  c := color.New(color.FgCyan)
  c.Println("Prints cyan text")
}
```

调用时：

```sh
$ go run color.go # 带颜色
$ go run color.go --no-color # 不带颜色
$ go run color.go --no-color > color.go # 重定向到文件时没有乱码
```

你有更优雅的解决方案吗？