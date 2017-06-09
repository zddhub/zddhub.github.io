---
layout: post
title: "Go语法简略 － web应用框架"
category: Memo
tags: "web框架 Go 编程语言"
---

> [Martini](https://github.com/go-martini/martini)是目前最流行的基于Go的web应用框架，借鉴了express和sinatra。其内核类非常简洁小巧，值得研究。

本文探索Go对web编程的支持。

<!-- more -->

# Go net/http包
Go对web编程的支持封装在net/http包中。以下是一个简单的例子：从服务器给客户端输出`Hello world`.

```go
package main

import (
  "fmt"
  "log"
  "net/http"
)

func main() {
  // 设置访问的路由
  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello world!")
  })
  err := http.ListenAndServe(":8080", nil)
  if err != nil {
    log.Fatal("ListenAndServe: ", err)
  }
}
```

build之后运行，用浏览器访问`http://localhost:8080`,可以看到浏览器成功输出`Hello world`。

# Martini框架
对比一下Martini对`Hello world`的实现:

```go
package main

import "github.com/go-martini/martini"

func main() {
  m := martini.Classic()
  m.Get("/", func() string {
    return "Hello world!"
  })
  m.Run()
}
```
默认的Martini服务绑定在3000端口上。

# 鸭子尝试
> 当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。

话说科学是一门猜测性的学问，计算机科学尤其是。太多的系统、软件、应用，我们没时间也没精力，有些更没办法搞清楚它的实现细节，只能靠猜测。如果按照我们猜测能实现相同的功能，性能也差不多的话，那就认为它是这样实现的，我把这种尝试叫鸭子尝试。现在来尝试一下，写个简单的Get实现：

```go
// duck_serve.go
// 先写Hello world程序
package main

import "github.com/zddhub/duck"

func main() {
  d := duck.Incubate() // 孵化出一只鸭子😋
  d.Get("/", func() string {
    return "Hello world!"
  })
  d.Run()
}

```

是不是和Martini的程序一毛一样？简单的封装一下net/http包如下：

```go
package duck

import (
  "fmt"
  "net/http"
)

type Duck struct {
  IP   string
  Port string
}

func Incubate() Duck {
  return Duck{"", "3030"} // 默认监听在3030端口上
}

func (d Duck) Run() {
  fmt.Println("[Duck] listening on", d.IP+":"+d.Port)
  http.ListenAndServe(d.IP+":"+d.Port, nil)
}

func (d Duck) Get(pattern string, f func() string) {
  http.HandleFunc(pattern, func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, f()) // 将f()的返回值写回到w
  })
}
```

运行`go run duck_serve.go`, 用浏览器访问`http://localhost:3030`, 成功输出`Hello world`。

这只是做了一个简单的封装，没有任何实际意义，因为我们固定了Get的第二个参数类型`f func() string`。根据`nodejs`的经验，controller一般会支持两个参数`request`和`response`：

```go
package main

import (
  "fmt"
  "github.com/zddhub/duck"
  "net/http"
)

func main() {
  d := duck.Incubate() // 孵化出一只鸭子😋
  d.Get("/", func() string {
    return "Hello world!"
  })
  // 放在不同的路由上
  d.Get("/zdd", func(res http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(res, "Can you do it?")
  })
  d.Run()
}
```

当然是不能直接运行的，这时候[反射]({% post_url 2015-07-03-go-reflect %})就派上了用场。现在让我们思考一个问题：运行时匿名函数的参数是谁传进去的呢？带着这个问题，改造一下我们的duck包：

```go
package duck

import (
  "fmt"
  "net/http"
  "reflect"
)

type Duck struct {
  IP   string
  Port string
}

func Incubate() Duck {
  return Duck{"", "3030"}
}

func (d Duck) Run() {
  fmt.Println("[Duck] listening on", d.IP+":"+d.Port)
  http.ListenAndServe(d.IP+":"+d.Port, nil)
}

//: 1
// func (d Duck) Get(pattern string, f func() string) {
//  http.HandleFunc(pattern, func(w http.ResponseWriter, r *http.Request) {
//    fmt.Fprintf(w, f()) // 将f()的返回值写到w
//  })
// }

//: 2
func (d Duck) Get(pattern string, i interface{}) {
  t := reflect.TypeOf(i)
  v := reflect.ValueOf(i)
  if v.Kind() != reflect.Func {
    return
  }

  var ret string

  if t.NumIn() == 0 {
    rt := v.Call([]reflect.Value{})
    ret = rt[0].Interface().(string)
  } else if t.NumIn() == 2 {
    // 这里可以得到匿名方法参数的类型, 但是不能得到方法的实参。
    // 方法只有在被调用之后，实参才是有效的，谁调用，谁就负责参数传递。
    fmt.Println(t.In(0), t.In(1))
    fmt.Println(reflect.ValueOf(t.In(0)), reflect.ValueOf(t.In(1)))
  }

  // 有两种方法可以实现我们的需求：
  // 1. 将 i interface{} 反射成func，作为参数直接传给http.HandleFunc. 
  //    然而reflect包并没有办法实现这种功能，该方法不可用。
  // 2. 将HandleFunc方法的参数w, r的值做为参数传递给Get中定义的匿名方法，并通过反射调用它。
  http.HandleFunc(pattern, func(w http.ResponseWriter, r *http.Request) {
    // Call中的匿名函数:
    // func(res http.ResponseWriter, req *http.Request) {
    //   fmt.Fprintf(res, "Can you do it?")
    // }
    // Call func(w, r)
    fmt.Fprintf(w, ret)
  })
}
```

按照上面的思路，我们重写一下Get方法：

```go
package duck

import (
  "fmt"
  "net/http"
  "reflect"
)

type Duck struct {
  IP   string
  Port string
}

func Incubate() Duck {
  return Duck{"", "3030"}
}

func (d Duck) Run() {
  fmt.Println("[Duck] listening on", d.IP+":"+d.Port)
  http.ListenAndServe(d.IP+":"+d.Port, nil)
}

func (d Duck) Get(pattern string, i interface{}) {
  t := reflect.TypeOf(i)
  v := reflect.ValueOf(i)
  if v.Kind() != reflect.Func {
    return
  }

  http.HandleFunc(pattern, func(w http.ResponseWriter, r *http.Request) {
    var in = make([]reflect.Value, t.NumIn())
    if t.NumIn() == 2 {
      in[0], in[1] = reflect.ValueOf(w), reflect.ValueOf(r)
    }
    ret := v.Call(in) // 通过反射，将w，r作为参数传递给Get中定义的匿名方法，并调用
    if len(ret) != 0 {
      fmt.Fprintf(w, ret[0].Interface().(string))
    }
  })
}
```

运行一下，访问`http://localhost:3030`和`http://localhost:3030/zdd`,都可以看到期望的输出。
像这种Get中的匿名方法依赖Get对它做参数传递，从匿名方法内部来看，参数就像从外部注入一样，我们把这种机制叫`依赖注入`。其实每个方法的调用，参数都是从外部传递过来的，到底有什么不同呢？请看下一篇:[依赖注入]({% post_url 2015-07-05-go-dependency-inject %})。

* 上一篇: [Go语法简略 － 反射]({% post_url 2015-07-03-go-reflect %})
* 下一篇: [Go语法简略 － 依赖注入]({% post_url 2015-07-05-go-dependency-inject %})