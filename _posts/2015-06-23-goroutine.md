---
layout: post
title: "Go语法简略 － goroutine"
category: Memo
tags: "Go 编程语言"
---

> 优秀的程序员应该不限于只用一两种语言，对各种语言都要有猎涉，这样才能在合适的应用下选择最合适的语言。但优秀的程序员至少需要精通一到两种语言，在众多的编程语言中，我选择Go和Swift作为自己之后20年内的主要编程语言，并力争达到精通的境地。

本文主要记录[Go](https://golang.org/)并行相关语法，方便查询。本文为了便于表述加了行号，复制后可切断行号，命令如下：

```sh
$ pbpaste | sed -e 's/^ *[0-9]\{1,\} //g'
```
<!-- more -->

### 12. goroutine
goroutine 是由 Go 运行时环境管理的轻量级线程。
```go
package main

import (
  "fmt"
  "time"
)

func say(s string) {
  for i := 0; i < 5; i++ {
    time.Sleep(100 * time.Millisecond)
    fmt.Println(s)
  }
}

func main() {
  go say("world") //开启一个新的goroutine执行
  say("hello")
}
```

<!-- more -->

### 13. channel
```go
package main

import "fmt"

func sum(a []int, c chan int) {
  sum := 0
  for _, v := range a {
    sum += v
  }
  c <- sum // 将和送入 c, 箭头方向为数据流方向
}

func main() {
  a := []int{7, 2, 8, -9, 4, 0}

  c := make(chan int) // 使用前需要使用make创建
  go sum(a[:len(a)/2], c)
  go sum(a[len(a)/2:], c)
  x, y := <-c, <-c // 从 c 中获取

  fmt.Println(x, y, x+y)
}
```

* 上一篇: [Go语法简略 － 方法和接口]({% post_url 2015-06-23-go-interface %})
* 下一篇: [Go语法简略 － 面向对象]({% post_url 2015-06-29-go-orient-object %})