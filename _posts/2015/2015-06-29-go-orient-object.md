---
layout: post
title: "Go语法简略 － 面向对象"
category: Memo
tags: "Go 编程语言"
---

> 优秀的程序员应该不限于只用一两种语言，对各种语言都要有猎涉，这样才能在合适的应用下选择最合适的语言。但优秀的程序员至少需要精通一到两种语言，在众多的编程语言中，我选择Go和Swift作为自己之后20年内的主要编程语言。

本文主要记录[Go](https://golang.org/)面向对象的语法，方便查询。

<!-- more -->

```go
package main

import "fmt"

// Go中没有类，只有struct，Go使用匿名域完成了struct的复用，使用接口完成了类似多态的功能。

type Human struct {
  name  string
  email string
  age   int
}

type Boss struct { // 定义结构
  Human   // 匿名字段， 复用Human结构
  company string
  url     string
  email   string // 重名属性
}

func (h Human) SayHello() {
  fmt.Printf("Hi, I am %s, %d years old, you can contact me via %s.\n", h.name, h.age, h.email)
}

func (h Human) Laugh() {
  fmt.Println("Hahaha ...")
}

func (b Boss) Laugh() { // 重写了SayHello方法
  fmt.Println("Hehehe ...")
}

type Sportor interface { // 定义接口, 接口中包含了一系列方法
  Walk()
  Run(km int)
  // Man string 错误，接口中只能包含方法
  // Method() {} 错误，不能有默认的方法实现
}

type Laugher interface {
  Laugh()
}

type Speaker interface {
  Laugher // 匿名字段，引用Laugher中的方法
  SayHello()
  SayGoodbye()
}

// Boss实现SayGoodbye()
func (b Boss) SayGoodbye() {
  fmt.Println("Goodbye, my employee!")
}

// Human实现Walk方法
func (h *Human) Walk() { // 使用指针可避免拷贝，并且可以修改接受者的值
  fmt.Println("I am walking")
}

// Human实现Run方法
func (h *Human) Run(km int) {
  fmt.Println("Run ", km, "mile")
}

func main() {
  fmt.Println("This is a go lang practice.")
  // 结构复用
  h := Human{"zddhub", "zddhub@gmail.com", 10}
  fmt.Println("Name:", h.name) // 访问属性
  fmt.Println("Age:", h.age)
  fmt.Println("Email:", h.email)
  h.SayHello()
  h.Laugh()

  b := Boss{Human{"daniu", "zddhub@gmail.com", 20}, "daniu.io",
    "www.daniu.io", "daniu@gmail.com"}
  // 不能这样初始化Boss
  // c := Boss{"daniu", "zddhub@gmail.com", 20, "daniu.io",
  //  "www.daniu.io", "daniu@gmail.com"}
  // 也不能这样初始化Boss
  // d := Boss{ {"daniu", "zddhub@gmail.com", 20}, "daniu.io",
  //  "www.daniu.io", "daniu@gmail.com"}

  fmt.Println("Name:", b.name) // 访问属性
  fmt.Println("Age:", b.age)
  fmt.Println("Email:", b.email)             // 重名属性，最外层优先使用
  fmt.Println("Human Email:", b.Human.email) // 用结构名明确访问内层变量
  b.SayHello()                               // 继承Human的SayHello方法
  b.Laugh()                                  // 重写Human的Laugh方法
  b.Human.Laugh()                            // 用结构名明确调用内层方法

  // Go没有像C++一样的继承，所以下面的方法是错误的:
  // var hh *Human
  // hh = &b
  // 这种情况在Go中由接口来实现

  // 接口
  var l Laugher
  l = h // l能存Human
  fmt.Println(l)
  l.Laugh()
  l = b // l能存Boss
  fmt.Println(l)

  x := make([]Laugher, 2)
  x[0], x[1] = h, b // 两个不同类型的元素，实现了同样的接口
  for _, value := range x {
    value.Laugh() // 多态的调用Laugh，显示不同的功能
  }

  var s Speaker
  s = b // b实现了Speaker接口中的所有方法
  s.SayHello()
  s.Laugh()
  s.SayGoodbye()
  // s.name 错误，接口只能调用方法，不能调用属性

  var spt Sportor
  spt = &h
  // spt = h 错误，接收者为指针
  spt.Run(10)
  spt.Walk()

  var a interface{} // a为空接口，可以引用任意类型的数据,如：
  a = h
  a = b
  a = 1
  a = spt
  fmt.Println(a)

  // interface中变量的类型
  list := make([]interface{}, 4)
  list[0], list[1], list[2], list[3] = h, b, 1, spt

  for index, element := range list {
    if value, ok := element.(Human); ok { // 使用类型断言判断
      fmt.Printf("list[%d] is an Human and its value is %v\n", index, value)
    } else if value, ok := element.(Boss); ok {
      fmt.Printf("list[%d] is an Boss and its value is %v\n", index, value)
    } else if value, ok := element.(int); ok {
      fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
    } else if value, ok := element.(*Human); ok {
      fmt.Printf("list[%d] is an *Human and its value is %d\n", index, value)
    }
  }

  // 也可以减少if else的使用，用switch来处理
  for index, element := range list {
    switch value := element.(type) { // element.(type)的语法只能在switch语句之内使用
    case int:
      fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
    case Human:
      fmt.Printf("list[%d] is a Human and its value is %v\n", index, value)
    case Boss:
      fmt.Printf("list[%d] is a Boss and its value is %v\n", index, value)
    default:
      fmt.Printf("list[%d] is of a different type\n", index)
    }
  }
}
```

* 上一篇: [Go语法简略 － goroutine]({% post_url 2015/2015-06-23-goroutine %})
* 下一篇: [Go语法简略 － 反射]({% post_url 2015/2015-07-03-go-reflect %})
