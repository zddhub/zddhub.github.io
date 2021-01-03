---
layout: post
title: "Go语法简略 － 反射"
category: Memo
tags: "反射 Go 编程语言"
---

> 反射是指计算机程序在运行时(Run time)可以访问、检测和修改它本身状态或行为的一种能力。
一般来说，静态语言都会经过源码—>编译->运行的过程，reflect反其道而行，在运行时访问、
检测或修改源码的行为。

本文主要记录[Go](https://golang.org/)的反射语法，方便查询。

<!-- more -->

```go
package main

import (
  "fmt"
  "reflect"
)

// reflect的用法, 包括访问、设置属性和方法调用。

func Info(i interface{}) {
  t := reflect.TypeOf(i)
  v := reflect.ValueOf(i)
  fmt.Println(t, v)

  // 检测不为struct类型时退出
  if k := t.Kind(); k != reflect.Struct {
    return
  }
  // Type:
  fmt.Println(t.PkgPath(), t.Name(), "")
  fmt.Println("- Field num:", t.NumField(), " Method num:", t.NumMethod())

  // 输出属性
  for j := 0; j < t.NumField(); j++ {
    f := t.Field(j)
    fmt.Println("-", f.Name, f.Type)
  }
  // 输出方法
  for j := 0; j < t.NumMethod(); j++ {
    m := t.Method(j)
    fmt.Println("+", m.Name, m.Type)
  }
}

// 获取struct属性的值
func GetValue(o interface{}) {
  v := reflect.ValueOf(o)
  fmt.Println(o)
  // 检测不为struct类型时退出
  if k := v.Kind(); k != reflect.Struct {
    return
  }
  for i := 0; i < v.NumField(); i++ {
    f := v.Field(i)
    if f.CanInterface() {
      // 读取未导出变量时会panic:
      // panic: reflect.Value.Interface: cannot return value obtained from unexported field or method
      // 用CanInterface过滤
      if k := f.Kind(); k == reflect.Struct {
        sf := f.Field(i)
        fmt.Println("-", sf.Interface(), sf.Type())
      } else {
        fmt.Println("-", f.Interface(), f.Type())
      }
    }
  }
}

// 设置属性的值
func SetValue(o interface{}, name string, value interface{}) {
  v := reflect.ValueOf(o)
  if k := v.Kind(); k != reflect.Ptr || !v.Elem().CanSet() {
    fmt.Println("Invalid set type:", o)
    return
  }
  v = v.Elem()
  f := v.FieldByName(name)
  if !f.IsValid() {
    fmt.Println("Invalid name:", name)
  }

  f.Set(reflect.ValueOf(value))
}

// 调用方法
func CallMethod(o interface{}, name string, args interface{}) {
  v := reflect.ValueOf(o)
  mn := v.MethodByName(name)
  fmt.Println(mn, mn.Kind())
  if mn.Kind() == reflect.Func {
    if mn.Type().NumIn() == 0 {
      mn.Call([]reflect.Value{})
    } else {
      mn.Call([]reflect.Value{reflect.ValueOf(args)})
    }
  }
}

type Human struct {
  Name string
  Age  int
  url  string
}

func (h Human) SayHello() {
  fmt.Println("Hi, I am ", h.Name)
}

func (h Human) SetAge(age int) {
  h.Age = age
  fmt.Println("I'm in Set Age:", h.Age)
}

func (h *Human) SetUrl(url string) {
  h.url = url
}

type Boss struct {
  Human
  money int
}

func main() {
  // 获取属性信息和方法
  Info(1)
  Info(11.15)
  Info("String-Value")
  //: int <int Value>
  //: float64 <float64 Value>
  //: string String-Value

  h := Human{"zddhub", 27, "www.zddhub.com"}
  Info(h)
  //: main.Human <main.Human Value>
  //: main Human
  //: - Field num: 3  Method num: 2
  //: - Name string
  //: - Age int
  //: - url string
  //: + SayHello func(main.Human)
  //: + SetAge func(main.Human, int)
  //: 接收者为指针的函数没有被输出

  b := Boss{Human{"Jobs", 30, "www.daniu.io"}, 1e10}
  Info(b)
  //: main.Boss <main.Boss Value>
  //: main Boss
  //: - Field num: 2  Method num: 2
  //: - Human main.Human
  //: - money int
  //: + SayHello func(main.Boss)
  //: + SetAge func(main.Boss, int)
  //: 接受者为指针的函数没有被输出

  // 返回属性的值
  GetValue(h)
  //: {zddhub 27 www.zddhub.com}
  //: - zddhub string
  //: - 27 int
  GetValue(b)
  //: { {Jobs 30 www.daniu.io} 10000000000}
  //: 双括号会和jekyll语法冲突，所以上一行中间加了一个空格
  //: - Jobs string

  // 设置属性的值
  fmt.Println(h.Age, h.Name)
  //: 27 zddhub
  SetValue(h, "Age", 10)
  //: Invalid set type: {zddhub 27 www.zddhub.com}
  SetValue(h, "Name", "wx")
  //: Invalid set type: {zddhub 27 www.zddhub.com}
  fmt.Println(h.Age, h.Name)
  //: 27 zddhub
  SetValue(&h, "Age", 200)
  SetValue(&h, "Name", "wx")
  fmt.Println(h.Age, h.Name)
  //: 200 wx

  // 方法调用
  CallMethod(h, "SayHello", nil)
  //: <func() Value> func
  //: Hi, I am  wx
  CallMethod(&h, "SetAge", 80) // 函数的接受者为copy的另一对象，设置不成功
  fmt.Println(h.Age, h.Name, h.url)
  //: <func(int) Value> func
  //: I'm in Set Age: 80
  //: 200 wx www.zddhub.com

  CallMethod(&h, "SetUrl", "www.daniu.io") // 接收者为指针类型，设置成功
  fmt.Println(h.url)
  //: <func(string) Value> func
  //: www.daniu.io
}

```

* 上一篇: [Go语法简略 － goroutine]({% post_url 2015/2015-06-23-goroutine %})
* 下一篇: [Go语法简略 － web框架]({% post_url 2015/2015-07-04-go-web-framework %})
