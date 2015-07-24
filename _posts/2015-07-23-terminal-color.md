---
layout: post
title: "终端颜色输出重定向"
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
| 0 － black | 1 - red | 2 - green | 3 - yellow | 4 - blue | 5 - purple | 6 - cyan | 7 - white |

<!-- more -->

当将带颜色的命令重定向到文件时，会打印颜色字符：

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

来自Tony的方案
------------

Tony提供了一个很好的方案，除去了命令行中多余的参数`--no-color`。非常感谢，么么哒。

```go
package main

import (
  "fmt"
  "github.com/mattn/go-isatty"
  "os"
)

func main() {
  fmt.Println(isatty.IsTerminal(os.Stdin.Fd()), isatty.IsTerminal(os.Stdout.Fd()))

  if !isatty.IsTerminal(os.Stdout.Fd()) {
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

现在，一切都变的统一了。

```sh
$ go run color.go # 带颜色
$ go run color.go > color.go # 重定向到文件时没有乱码
```

看了一下[isatty](https://github.com/robertkrimen/isatty)的实现，它使用了系统调用`syscall.Syscall6`来check不同的终端文件描述符:

```go
func check(fd uintptr) bool {
  var garbage [128]byte
  _, _, err := syscall.Syscall6(syscall.SYS_IOCTL, uintptr(fd),
    ioctlQuery,                        // Either TCGETS or TIOCGETA, basically request the termios struct
    uintptr(unsafe.Pointer(&garbage)), // A garbage slice to be filled with termios data
    0, 0, 0)
  return err == 0
}
```
目前支持Darwin、Linux、和FreeBSD。
