---
layout: post
title: "MV* 软件设计架构"
category: Memo
tags: "MV; MVC; MVP; MVVM; Design architecture"
---

软件工程推荐设计**高内聚，低耦合**的程序，方便代码复用，测试和维护。除了微观上符合 [SOLID][SOLID] 原则外，宏观上良好的分层/分类也能极大的降低实体之间的耦合。客户端应用程序实体可以分成三大类：
- 模型(Model): 负责数据的操作(CRUD)
- 视图(View): 负责 UI 的绘制和交互
- *: 其它实体。负责在 MV 之间传递数据和业务逻辑，常见的抽象有 Controller，Presenter 和 ViewModel 等

根据对其它实体的抽象，出现了一系列 MV* 软件设计架构。本文整理并比较不同设计架构的优缺点和使用场景。

<!-- more -->

### MV

Model-View (MV) 是最简单的一种实现。View 拥有 Model 所有操作权限，负责根据用户输入更新 Model，刷新自己。

![MV diagram][MV]

#### 优点
- 代码简单，不需要中间层

#### 缺点
- Model 和 View 耦合度高，View 负责更新 Model
- 混有业务逻辑的 View 很难复用

#### 适用场景
- 简单原型项目
- View 只读 Model 数据，不更新

### MVC

Model-View-Controller (MVC) 是最常见的架构，[最原始的实现][MVC_wikipedia]如下：

![MVC diagram][MVC]

#### 优点
- Model 和 View 有绑定，但不直接通过 View 更新 Model。
- 使用广泛，有多个变种，针对领域问题有成熟的解决方案
- 易用性良好

#### 缺点
- Model 和 View 有绑定，没有完全解耦
- View 和 Controller 强耦合
- Controller 可以负责 View 的创建，可避免
- Controller 负责与 View 交互和业务逻辑，很容易写出臃肿的 Controller。在实际使用时，可将 Controller 进一步拆分。

#### 适用场景
- 多数客户端项目，保底选择

MVC 有很多变种，比如[Cocoa 的实现][Apple_MVC_Link]：Model 和 View 完全解耦。

![Apple MVC diagram][Apple_MVC]

还有一种后端常用的架构，直接从 Controller 开始，到 Render 结束：

![Backend MVC diagram][Backend_MVC]

### MVP

Model-View-Presenter (MVP) 是 MVC 的另一种演变。把构建 View 的代码从 Controller 中剥离出去(放在 View 中)，命名为 Presenter。View 获得用户交付事件后，把 event 发送到 Presenter 来处理。

![MVP][MVP]

#### 优点
- 不直接通过 View 更新 Model
- Presenter 不负责 View 的创建
- Presenter 只负责处理 View 的行为

#### 缺点
- Presenter 比较薄，导致 Model 和 View 比较重，一般使用 [Supervising Controller][SC] 给 Model 减负
- 允许 Model 和 View 有绑定，没有完全解耦

#### 适用场景
- 一般来说，能使用 MVC 的场景就能使用 MVP

### MVVM

Model-View-ViewModel (MVVM) 首次出现在微软的 WPF 和 Silverlight 项目中，是 MVP 的变种。

![MVVM diagram][MVVM]

如图所示，MVVM 和 Cocoa MVC 的实现非常接近，不用之处在于：Cocoa MVC 版本中的 View Controller 除了负责业务逻辑外，还负责创建 UI，和 View 的耦合更紧密。而 MVVM 中的 ViewModel 不创建任何 UI，只封装与 UI 相关的状态。ViewModel 相比于 ViewController 更加轻量。

MVVM 会利用绑定技术，View 变化时，ViewModel 会自动变化，反之亦然。所以，MVVM 模式又被称作：[model-view-binder 模式][model-view-binder]。

#### 优点
- Model 和 View 解耦，View 不直接操作 Model
- ViewModel 只负责 View 状态的改变，不创建 UI
- 业务逻辑独立于 Model 和 View，可单独测试

#### 缺点
- 数据绑定使得 Bug 很难被调试
- 对于过大的项目，数据绑定和数据转化需要花费更多的内存（成本）
- 给每个 View 设置一个 ViewModel 会增加代码量

#### 适用场景
- 对数据绑定友好支持的平台/项目
- 高模块化应用


### 比较和总结

据我理解，MV* 架构的不同点体现在 `Controller, Presenter, ViewModel` 的职责和与 MV 的协作方式上。比较如下：

|          | 职责 | 与 MV 的协作方式 |
|:---------|:--|:--|
|Controller| 操作数据并控制 View | 在 MV 中占主导地位，同时控制 MV |
|Presenter | 负责响应 View 的行为 | 连接 Model 和 View |
|ViewModel | 负责 View 的状态 并响应 View 的行为 | 和 View 强绑定 |


### 参考资料
- [iOS Architecture Patterns](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52)
- [GUI Architectures: MVP](https://www.martinfowler.com/eaaDev/uiArchs.html)
- [SwiftUI Architectures: Model-View, Redux & MVVM](https://quickbirdstudios.com/blog/swiftui-architecture-redux-mvvm/)
- [Migrating Swift iOS Apps from MVC to MVVM](https://www.twilio.com/blog/2018/05/migrating-swift-ios-mvc-mvvm.html)
- [Ending the debate: MVC vs MVP vs MVVM for iOS application development](https://www.simform.com/mvc-mvp-mvvm-ios-app-development/)
- [MVC vs MVVM: Key Differences with Examples](https://www.guru99.com/mvc-vs-mvvm.html)
- [浅谈 MVC、MVP 和 MVVM 架构模式](https://draveness.me/mvx/)


[SOLID]: https://en.wikipedia.org/wiki/SOLID
[MV]: /assets/images/2021-01-05/mv.png
[MVC]: /assets/images/2021-01-05/mvc.png
[MVC_wikipedia]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[Apple_MVC_Link]: https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html
[Apple_MVC]: /assets/images/2021-01-05/apple_mvc.png
[MVP]: /assets/images/2021-01-05/mvp.png
[SC]: https://www.martinfowler.com/eaaDev/SupervisingPresenter.html
[MVVM]: /assets/images/2021-01-05/mvvm.png
[model-view-binder]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel
