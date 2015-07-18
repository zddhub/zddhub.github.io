---
layout: post
title: "Go语法简略 － 方法和接口"
category: Memo
tags: "Go 编程语言"
---

> 优秀的程序员应该不限于只用一两种语言，对各种语言都要有猎涉，这样才能在合适的应用下选择最合适的语言。但优秀的程序员至少需要精通一到两种语言，在众多的编程语言中，我选择Go和Swift作为自己之后20年内的主要编程语言，并力争达到精通的境地。

本文主要记录[Go](https://golang.org/)方法和接口语法，方便查询。本文为了便于表述加了行号，复制后可切断行号，命令如下：

```sh
$ pbpaste | sed -e 's/^ *[0-9]\{1,\} //g'
```

<!-- more -->

### 10. 方法
Go中没有类，但是可以为type 类型定义方法
{% highlight go linenos %}
package main

import (
  "fmt"
  "math"
)

type Vertex struct {
  X, Y float64
}

// 该方法内只对接受者进行了读操作，所以此处使用变量或者指针都不会影响最终结果，
// 但是使用指针能避免一次值拷贝
func (v Vertex) Abs() float64 { // 定义Vertex的Abs方法
  return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) { // 接收者使用指针时，对v的值写操作后生效
  v.X = v.X * f
  v.Y = v.Y * f
}

type MyFloat float64

func (f MyFloat) Abs() float64 { // 定义float64的Abs方法
  if f < 0 {
    return float64(-f)
  }
  return float64(f)
}

func main() {
  v := &Vertex{3, 4}
  fmt.Println(v.Abs())

  v.Scale(5)
  fmt.Println(v)

  f := MyFloat(-math.Sqrt2)
  fmt.Println(f.Abs())
}
{% endhighlight %}

<!-- more -->

### 11. 接口
{% highlight go linenos %}
package main

import (
  "fmt"
  "math"
)

type Abser interface { // 定义接口
  Abs() float64
}

func main() {
  var a Abser
  f := MyFloat(-math.Sqrt2)
  v := Vertex{3, 4}

  a = f  // a MyFloat implements Abser
  a = &v // a *Vertex implements Abser

  // v是Vertex变量不是*Vertex, 所以a=v错误。因为只实现了*Vertex的接口，
  // 但是若实现了Vertex的接口，则a=v和a=&v都可以使用
  // a = v

  fmt.Println(a.Abs())
}

type MyFloat float64

func (f MyFloat) Abs() float64 { // MyFloat实现Abser接口的方法
  if f < 0 {
    return float64(-f)
  }
  return float64(f)
}

type Vertex struct {
  X, Y float64
}

func (v *Vertex) Abs() float64 { // Vertex实现Abser接口的方法
  return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
{% endhighlight %}

Go中内置了一些常见的接口：
#### Stringers
{% highlight go linenos %}
package main

import "fmt"

type Person struct {
  Name string
  Age  int
}

/*
// fmt 包中默认的定义
type Stringer interface {
    String() string
}
*/

func (p Person) String() string {
  return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
  a := Person{"Arthur Dent", 42}
  z := Person{"Zaphod Beeblebrox", 9001}
  fmt.Println(a, z)
}
{% endhighlight %}

#### Errors
{% highlight go linenos %}
package main

import (
  "fmt"
  "time"
)

type MyError struct {
  When time.Time
  What string
}

/*
// error接口
type error interface {
    Error() string
}
*/

func (e *MyError) Error() string {
  return fmt.Sprintf("at %v, %s",
    e.When, e.What)
}

func run() error {
  return &MyError{
    time.Now(),
    "it didn't work",
  }
}
{% endhighlight %}

#### Web servers
```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}
```

#### Image
```go
package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
```

* 上一篇: [Go语法简略 － 基础篇]({% post_url 2015-06-22-go %})
* 下一篇: [Go语法简略 － goroutine]({% post_url 2015-06-23-goroutine %})
