---
layout: post
title: "Swift/SwiftUI 踩坑记"
category: Article
tags: "Swift;SwiftUI;iOS"
---

这里记录 Swift/SwiftUI 开发过程中踩过的坑。

<!-- more -->

注：目前的项目混合使用 SwiftUI 和 UIKit，不能保证纯 SwiftUI 也有类似问题。

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
