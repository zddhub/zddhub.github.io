---
layout: post
title: "Swift/SwiftUI 踩坑记"
category: Article
tags: "Swift;SwiftUI;iOS"
---

这里记录 Swift/SwiftUI 开发过程中踩过的坑。

<!-- more -->

注：目前的项目混合使用 SwiftUI 和 UIKit，不能保证纯 SwiftUI 也有类似问题。

## @State, @StateObject, @ObservedObject

这三兄弟是 SwiftUI 中最常用的三个属性，使用不当会得到奇怪的结果，也有可能导致性能问题。这里只记录如何正确的使用它们，来符合 SwiftUI 设计者的初衷。

### @State

@State 定义在使用它的 View 的最外层，当改变时，SwiftUI 刷新依赖它的视图。@State 只对当前视图和子视图可见，因此通常标记成 private，它们永远成对出现，如果你发现不能标记成 private，那么可能不应该使用 @State。@StateObject 也一样。用法如下：

```swift
1. 经典用法
@State private var library: Library = Library()
// @State 属性的初始化在其生命周期内只被赋值一次，但是右值 `Library()` 会被创建多次，（在当前视图的输入参数变化时，init 会被调用。
// 即使在 init 里设置 library = Library(name: "new"), SwiftUI 也只保证 library 只被赋值一次。
// View 的 id 变化时，生命周期结束

2. 延迟初始化, 对于频繁刷新的视图，为避免右值被创建多次，并且创建右值代价太大时，使用延迟调用
@State private var library: Library?

var body: some View {
  MyView()
    .task { // task 只调用一次
      library = Library()
    }
}
```

@State 自己改变时视图刷新，但是 @State 的属性改变时，视图并不会刷新，要做到这一点，就需要用 @StateObject

### @StateObject

使用 @StateObject 配合 ObservableObject 对象的 `@Published` 属性来检测属性的改变。SwiftUI 会保证 @StateObject 属性在当前视图生命周期内只被改变一次。@StateObject 的用法如下：

```swift
1. 经典用法
@StateObject private var model = DataModel()

2. 延迟初始化，和 @State 一样

@StateObject private var dataModel: DataModel?

var body: some View {
  MyView()
    .task { // task 只调用一次
      dataModel = DataModel()
    }
}

3. 在 Init 中初始化，即使 name 的变化会引起 init 被多次调用，但是 _model 也只被赋值一次。注意，这种写法会使得 DataModel 被创建多次，但没有赋值给 _model。所以，如果创建 DataModel 耗时，选择使用延迟初始化。
struct MyInitializableView: View {
  @StateObject private var model: DataModel

  init(name: String) {
      // SwiftUI ensures that the following initialization uses the
      // closure only once during the lifetime of the view, so
      // later changes to the view's name input have no effect.
      // State 和 StateObject 只能被初始化一次，但是 ObservedObject 每次都会被初始化
      // 如果想每次 name 更新时更新 _model, 需要更新 identifier，使用 .id(_) 方法。使用id有副作用，比如让动画失效。
      _model = StateObject(wrappedValue: DataModel(name: name))
      // model = DataModel(name: name) // 编译错误
  }

  var body: some View {
      VStack {
          Text("Name: \(model.name)")
      }
  }
}
```

### @ObservedObject

如果视图的输入 (参数)是 ObservableObject 对象（通常是 @StateObject），使用 @ObservedObject 接收它，不要在声明 @ObservedObject 的视图里初始化它。用法如下：

```swift
class DataModel: ObservableObject {
    @Published var name = "Some Name"
    @Published var isEnabled = false
}


struct MyView: View {
    @StateObject private var model = DataModel()

    var body: some View {
        Text(model.name)
        MySubView(model: model)
    }
}

struct MySubView: View {
    @ObservedObject var model: DataModel

    init(model: DataModel) {
      self.model = model // 只应该被这样用

      // 不要像以下这样用，和 State，StateObject 表现不一样
      // self.model = DataMoel() // 此时 model 被重新初始化
      // self._model = ObservedObject(wrappedValue: DataModel()) // 此时 model 被重新初始化
    }

    var body: some View {
        Toggle("Enabled", isOn: $model.isEnabled)
    }
}
```

Don’t specify a default or initial value for the observed object. Use the attribute only for a property that acts as an input for a view, as in the above example.

### 总结

- SwiftUI View 的 init 方法会被调用多次
- SwiftUI 在 View 的输入（View 的参数）改变时调用 init 方法
- 当状态改变或者 init 刷新时，body 会被调用
- body 执行时，里面的所有对象/View都会被重新创建，但不是所有的 View hierachy 会更新，SwiftUI 只重绘那些改变的部分
- 当 init 方法被重复调用时，@State, @StateObject 只会被初始化一次，@ObservedObject 每次都会被初始化
- @State, @StateObject 应该永远被标记成 private，永远在当前 View 内初始化
- @ObservedObject 只应该接受 ObservableObject 参数，不应该在当前 View 内被初始化
- 改变 View 的 id (identifier)，相当于创建一个全新的 View，会重置所有值。注意，动画可能也会收到影响而中断或者异常。

验证代码写在了 [Github Gist](https://gist.github.com/zddhub/77ffd275000970c9e99484ba6bdd8132) 上。

## 弹出键盘导致布局混乱

### 坑：

使用 push 弹出全屏幕的 View，该 View 中有一个 TextField，获得焦点后返回，主屏布局混乱，底部多了一块空白区域，所有元素事件响应失效，主屏是一个 ScrollView。逐步盘查后发现，注释掉子页面的 `view.becomeFirstResponder()` 后问题不复现。

### 合理的行为：

子屏幕不应该影响主屏幕的行为。

### 修复方案：

禁用键盘的安全区域。

```swift
.ignoresSafeArea(.keyboard)
```

### 启发

- 新开发的功能出问题后，应逐步盘查，用简单的 View 代替复杂的页面，逐步缩小范围，来定位问题。
- 如果确定是事件响应引起的问题，可能与键盘相关。
- 如果页面底部多了一块空白区域，可能是键盘收起后布局没有重新计算。


## @AppStorage 的 key 不能以 @ 开头

### 坑：

@AppStorage 能非常方便的存取 UserDefaults 中的数据，但是如果 key 使用 `@` 开头的话，会报 crash。

### 合理的行为：

不应该报 crash，或者文档中应该有类似的说明。

### 修复方案：

不使用 @ 开头的字符串做为 @AppStorage 的 key。如果是组合的字符串，保证 @ 之前的字符串不能为空。

### 启发

- `@` 是 objc 中字符串的关键字，目前 SwiftUI 的底层使用 UIKit，甚至 objc 的关键字
- 这个问题很难发现，只有踩坑才能知道
- 需要测试所有逻辑的路径，即使业务场景不存在

## 菜单 Menu

### 坑：

SwiftUI Menu 使用非常方便，但是在点开 Menu 菜单没有合上的时候，去点击/滑动同一个页面的其它元素 (使用 onTapGesture 响应)，这时页面卡死。

### 合理的行为：

此时，应该自动收起 Menu 的菜单，然后响应接下来的点击或者滑动事件。

### 修复方案：

当 Menu 打开时，给整个页面添加透明的蒙层，在 Menu 开启的时候，点击/滑动其它元素，隐藏蒙层。代码片段如下：

```swift
YourScreen {}
    .overlay {
      if menuIsOpen {
        Color.white.opacity(0.001)
          .ignoresSafeArea()
          .frame(maxWidth: .infinity, maxHeight: .infinity)
          .onTapGesture {
            menuIsOpen = false
          }
          .gesture(DragGesture(minimumDistance: 0)
            .onEnded { _ in
              menuIsOpen = false
            }
          )
      }
    }
```

### 启发

- 测试 Menu 时，记得在菜单打开的时候点击页面其它元素，或者滑动页面
- 测试 Menu 时，记得在菜单打开的时候点击页面上的另一个 Menu 元素
- 测试 Menu 时，记得旋转屏幕
- 透明蒙层时一个好的方案
