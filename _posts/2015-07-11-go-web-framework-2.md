---
layout: post
title: "Go语法简略 － Duck框架探索"
category: Memo
tags: "Martini Go 编程语言"
---

> 这个周末，《奔跑吧，兄弟》没有了，《那时那你》看完了，《万万没想到》剧组也没有新的项目，挠人的《花千骨》也要等到周日才更新，这是一个剧荒的周末，没办法，只能看看代码了。

本文打算参考[Martini](https://github.com/go-martini/martini)进一步实现我的[Duck框架]。

# Duck 计划

想想，一个经典的web框架，都有什么行为呢？web应用是通过http协议实现的，客户端通常由浏览器实现，web框架处在服务器端，主要完成对服务器请求的应答任务。首先，我希望它能自动解析路由，最好支持REST风格API的实现，可以方便的添加丰富的中间件，具有日志功能。能完美的支持MVC/MVVC结构，能解析静态文件，支持session功能和数据库访问能力。

打算支持路由，中间件处理，日志。其它使用中间件注入。

先研究一下Go的http包是如何实现的。

# Go http实现

Go在net/http包里实现了http协议，http使用TCP实现：在服务器端监听，请求到来时新建一个线程，响应客户端的请求。想知道更多的细节点这里:[build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/03.3.md)。

有两点需要说明一下：

* Go的变量和结构可以重名。😒，好吧，第一次知道这个，请原谅我的无知。

```go
// A Server defines parameters for running an HTTP server.
// The zero value for Server is a valid configuration.
type Server struct {
  Addr           string        // TCP address to listen on, ":http" if empty
  Handler        Handler       // <- 看这里，看这里 handler to invoke, http.DefaultServeMux if nil
  ...
}
```

* 在处理每个request时，调用的hander的ServeHTTP方法

```go
serverHandler{c.server}.ServeHTTP(w, w.req)

// 具体实现如下，其中sh.srv.Handler就是我们传给http.ListenAndServe的第二个参数
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
handler := sh.srv.Handler
if handler == nil { // 为空时默认使用DefaultServeMux
  handler = DefaultServeMux
}
if req.RequestURI == "*" && req.Method == "OPTIONS" {
  handler = globalOptionsHandler{}
}
handler.ServeHTTP(rw, req) // 实际处理请求的地方
}
```

DefaultServeMux是ServeMux对象的实例，ServeMux就是一个内置的路由器，最后绕了一圈:

```go
// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
  f(w, r) // 调用我们传过来的匿名方法
}
```

可以这样理解web路由：通过给定路由的pattern，把用户定义的匿名方法，注入到合适的服务环境中(w,r)环境，来实现用户希望的操作。

接下来就开始实现我们的Web框架吧。

# 最简单的实现

```go
package main

import (
  "fmt"
  "net/http"
  // "reflect"
)

type Handler interface{}

type Duck struct {
  *Injector
  handlers []Handler // handler all fanc
  IP       string
  Port     string
}

func Incubate() *Duck {
  return &Duck{Injector: New(), IP: "", Port: "3030"}
}

func (d *Duck) Run() {
  fmt.Println("[Duck] listening on", d.IP+":"+d.Port)
  http.ListenAndServe(d.IP+":"+d.Port, d) // 加上我们的
}

// 实现ServeHTTP方法，作为一个http.Handler被http server调用
// 每次请求调用一次
func (d *Duck) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  // 一次请求后的响应环境
  fmt.Println("[Duck] start ServeHTTP")

  // 设置环境
  d.SetMap(w)
  d.SetMap(r)

  // 根据request的信息，调用不同的handler方法，实现用户定义的功能
  for i := 0; i < len(d.handlers); i++ {

    if ret, err := d.Invoke(d.handlers[i]); err == nil {
      if len(ret) != 0 {
        fmt.Fprintf(w, ret[0].Interface().(string))
      }
    } else {
      fmt.Println(err)
    }
  }

  fmt.Println("[Duck] end   ServeHTTP")
}

func (d *Duck) Get(pattern string, handler Handler) {
  fmt.Println("[Duck] Get")
  d.handlers = append(d.handlers, handler)
}
```

上面的内容很简单，但是原理清楚，接下来的事情就容易了，通过不同的pattern和requst method来选用不同的handler，整个框架的核心部分就有了。



