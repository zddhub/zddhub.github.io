---
layout: post
title: "Go语法简略 － 依赖注入"
category: Memo
tags: "依赖注入 Go 编程语言"
---

通过对[web应用框架]({% post_url 2015-07-04-go-web-framework %})背后原理的探索，引入了依赖注入的概念。如果你需要读懂或者写一个框架的话，依赖注入的思想绝对能帮到你。本文记录对依赖注入的探索。

<!-- more -->

# 区分依赖和宿主

现在我们把问题简化一下：

```go
package main

import "fmt"

func MethodA(name string, f func(a int, b string)) {
  fmt.Println("Enter MethodA:", name)
  f(3030, "zdd") // 给f注入参数
  fmt.Println("Exit MethodA:", name)
}

func MethodB(a int, b string) {
  fmt.Println(a, b)
}

func main() {
  d := MethodB
  MethodA("zddhub", d)
}
```

在上述的例子中，为了完成MethodA的功能，需要依赖MethodB方法的实现，把MethodB方法叫做MethodA的`依赖(Dependency)`, MethodB叫做`宿主(Host)`。上面的例子直接在Host中调用`f(3030, "zdd")`来给依赖注入参数。或者换一个角度理解，`将自定义的方法MethodB整体注入到MethodA的执行环境中`。为了通用性考虑，通常会实现一个依赖注入器。

# 依赖注入器
接下来实现一个依赖注入器(Injector)。在[web应用框架]({% post_url 2015-07-04-go-web-framework %})的例子中，我们发现，在Host中，只能拿到依赖方法的参数类型，而我们需要将参数类型和参数值建立联系，因此，在Injector中封装一个`map[Type]Value`是自然而然的选择。

```go
package main

import (
  "fmt"
  "reflect"
)

type Injector struct {
  mappers map[reflect.Type]reflect.Value // 根据类型map实际的值
}

func (inj *Injector) SetMap(value interface{}) {
  inj.mappers[reflect.TypeOf(value)] = reflect.ValueOf(value)
}

func (inj *Injector) Get(t reflect.Type) reflect.Value {
  return inj.mappers[t]
}

func (inj *Injector) Invoke(i interface{}) []reflect.Value {
  t := reflect.TypeOf(i)
  if t.Kind() != reflect.Func {
    panic("Should invoke a function!")
  }
  inValues := make([]reflect.Value, t.NumIn())
  for k := 0; k < t.NumIn(); k++ {
    inValues[k] = inj.Get(t.In(k))
  }
  ret := reflect.ValueOf(i).Call(inValues)
  return ret
}

func Host(name string, f func(a int, b string)) {
  fmt.Println("Enter Host:", name)

  inj.Invoke(f) // 利用注入器中的环境调用f
  // 这种使用方法，看起来就像把自定义的方法f里的执行语句放在Host中执行一样自然
  // 语句从f里穿透到Host方法中，这就是注入一词的由来。

  fmt.Println("Exit Host:", name)
}

func Dependency(a int, b string) {
  fmt.Println("Dependency: ", a, b)
}

var inj *Injector // 全局的注入器，保存注入环境

func main() {
  // 创建注入器
  inj = &Injector{make(map[reflect.Type]reflect.Value)}
  inj.SetMap(3030)
  inj.SetMap("zdd")

  d := Dependency
  Host("zddhub", d)

  inj.SetMap(8080)
  inj.SetMap("www.zddhub.com")
  Host("website", d)
}
```
这样，一个简易的注入器就完成了。突然发现它和装饰器有那么一丢丢的像呀。不断的在外围裹上一层层的衣服，而你所需要专心关注的，正是衣服里面的😜，啊哈哈。。。😄。

稍微改进一下我们的注入器，让它更通用些。

```go
package main

import (
  "fmt"
  "reflect"
)

type Injector struct {
  mappers map[reflect.Type]reflect.Value // 根据类型map实际的值
}

func (inj *Injector) SetMap(value interface{}) {
  inj.mappers[reflect.TypeOf(value)] = reflect.ValueOf(value)
}

func (inj *Injector) Get(t reflect.Type) reflect.Value {
  return inj.mappers[t]
}

func (inj *Injector) Invoke(i interface{}) []reflect.Value {
  t := reflect.TypeOf(i)
  if t.Kind() != reflect.Func {
    panic("Should invoke a function!")
  }
  inValues := make([]reflect.Value, t.NumIn())
  for k := 0; k < t.NumIn(); k++ {
    inValues[k] = inj.Get(t.In(k))
  }
  ret := reflect.ValueOf(i).Call(inValues)
  return ret
}

func New() *Injector {
  return &Injector{make(map[reflect.Type]reflect.Value)}
}

func Host(name string, i interface{}) { // 让注入的方法不受限制
  inj.Invoke(i) // 利用注入器中的环境调用f
}

func Dependency(a int, b string) {
  fmt.Println("Dependency: ", a, b)
}

var inj *Injector

func main() {
  inj = New()
  inj.SetMap(3030)
  inj.SetMap("zdd")

  d := Dependency
  Host("zddhub", d)

  inj.SetMap(8080)
  inj.SetMap("zddhub@gmail.com")
  inj.SetMap(10.24)

  Host("email", func(email string, money float64, number int) {
    fmt.Println(email, money, number)
  })

  Host("Get", func() {
    fmt.Println("Hello world!")
  })
}
```

# 依赖注入的另一种使用场景
依赖注入的另一种使用方法是用在依赖属性的构造上，通常在测试框架中应用广泛。通过下面的例子，理解一下吧。

```go
type Camera struct { // 定义一个单反相机
  Name string
  L    Lens
}

type Lens struct { 
  LensType int // 0: 普通镜头，1: 广角镜头，2: 长焦镜头
}

func (c Camera) Capture() { // 使用不同的镜头拍照
  m := make(map[int]string, 3)
  m[0], m[1], m[2] = "normal lens", "wide-angle lens", "telephoto lens"

  value := inj.Get(reflect.TypeOf(c.L))
  var index int
  if value.Kind() != reflect.Invalid {
    index = value.Interface().(Lens).LensType
  } else {
    index = 0
  }
  fmt.Println("capture with", m[index])
}

var inj *Injector // 持有一个全局的注入器，复用之前实现的注入器。

func main() {
  inj = New()

  c := Camera{"ZddCamera", Lens{0}} // 组装一个普通镜头的单反相机, 通常组装一个单反需要很长的时间，假设你有工匠情结，并且DIY成功的概率是100%，努力工作一个月就可以组装完一台。
  c.Capture() // 拍照
  // 如果想继续测试广角镜头和长焦镜头的拍照效果，那么是不是要这样？
  // d := Camera{"ZddCamera", Lens{1}} // 你又努力工作了一个月
  // e := Camera{"ZddCamera", Lens{2}} // 你又努力工作了一个月
  // 如果Boss告诉你，最近又进口了十几款镜头，你是不是感觉要死的心都有了呢。

  // 正常人只需要组装新的镜头
  wLens := Lens{0} // 努力工作十天
  tLens := Lens{1} // 努力工作十天

  inj.SetMap(wLens) // 复用之前的单反，只更换镜头
  c.Capture() // 拍照

  inj.SetMap(tLens) // 复用之前的单反，只更换镜头
  c.Capture() // 拍照

  // 是不是感觉棒棒嗒😄
  //: capture with normal lens
  //: capture with wide-angle lens
  //: capture with normal lens
}
```

# 测试框架猜测
比如Android的测试框架通常会出现形如`@Inject`的注释，它都做了些什么事情呢？下面给出一个完整可运行的例子程序来说明一下：

```go
package main

import (
  "fmt"
  "reflect"
)

type Injector struct {
  mappers map[reflect.Type]reflect.Value // 根据类型map实际的值
}

func (inj *Injector) SetMap(value interface{}) {
  inj.mappers[reflect.TypeOf(value)] = reflect.ValueOf(value)
}

func (inj *Injector) Get(t reflect.Type) reflect.Value {
  return inj.mappers[t]
}

func (inj *Injector) Invoke(i interface{}) []reflect.Value {
  t := reflect.TypeOf(i)
  if t.Kind() != reflect.Func {
    panic("Should invoke a function!")
  }
  inValues := make([]reflect.Value, t.NumIn())
  for k := 0; k < t.NumIn(); k++ {
    inValues[k] = inj.Get(t.In(k))
  }
  ret := reflect.ValueOf(i).Call(inValues)
  return ret
}

func New() *Injector {
  return &Injector{make(map[reflect.Type]reflect.Value)}
}

type Camera struct { // 定义一个单反相机
  Name string
  // @Inject // <-－你在这里说明了L是需要注入的，通常用注释来说明注入
  L Lens
}

type Lens struct {
  LensType int // 0: 普通镜头，1: 广角镜头，2: 长焦镜头
}

func (c Camera) Capture() { // 使用不同的镜头拍照
  m := make(map[int]string, 3)
  m[0], m[1], m[2] = "normal lens", "wide-angle lens", "telephoto lens"

  //* fmt.Println("capture with", m[c.L.LensType]) // 这是你在程序中的代码，框架看到你使用了c.L
  // 默默的给你做了如下替换：
  fmt.Println("capture with", m[getLens(c.L).LensType])
}

// 由框架在编译时插入的方法
func getLens(L Lens) Lens {
  value := inj.Get(reflect.TypeOf(L))
  var l Lens
  if value.Kind() != reflect.Invalid {
    l = value.Interface().(Lens)
  } else {
    l = Lens{0}
  }
  return l
}

var inj *Injector // 持有一个全局的注入器，复用之前实现的注入器。

func main() {
  inj = New()

  c := Camera{"ZddCamera", Lens{0}}

  wLens := Lens{1}
  inj.SetMap(wLens) // 你需要用某种方式进行注入

  c.Capture()
}
```

好了，以上解释了依赖注入的使用和原理，如果你要读懂或者写一个框架的话，反射和依赖注入都是很有用的特征，那就掌握它吧。或者用的时候来读读这篇文章吧。

# 上述注入器的bug

你一定发现了上述注入器的bug：

```go
func main() {
  inj := New()
  inj.SetMap("zdd")
  inj.SetMap("daniu")
  inj.SetMap("opensse")
  Host("Get", func(a string, b string, c string) {
    fmt.Println("Hello, ", a, b, c)
    // 本来想给三个人say hello，结果发现给最后一个人说了三次
    //: Hello,  opensse opensse opensse
  })
```

你会发现单用类型做拓扑的注入器，只会持有最新一次类型的值。那么怎么办呢？没有解决不了的问题，传个`[]string`也是极好的。如果参数的结构比较固定的话，封装成struct也是不错的选择，Go允许给struct打Tag，而reflect也支持对Tag的访问，如果对这部分感兴趣，推荐你去读一下`"encoding/xml"`, `"encoding/json"`包的源码。

什么? 还有bug!!!, 😱, 留言或者发[邮件](mailto:zddhub@gmail.com)告诉我吧, 欢迎各种骚扰。另外，赞赏作者里有微信号哦，一般人我不告诉他🍀。

理解依赖注入了吗？现在读[odegangsta/inject](https://github.com/codegangsta/inject)是不是感觉很溜呀？facebook还开源了一套基于有向图实现的注入器[facebookgo/inject](https://github.com/facebookgo/inject)感觉屌屌的。如果对Java情有独钟，那就研究一下[Dagger](http://square.github.io/dagger/)吧，相信你一定会有所收获。

* 上一篇: [Go语法简略 － web应用框架]({% post_url 2015-07-04-go-web-framework %})
* 下一篇: [Go语法简略 － Duck框架探索]({% post_url 2015-07-11-go-web-framework-2 %})