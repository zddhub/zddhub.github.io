---
layout: post
title: "URL 加载系统（URL Loading System）"
category: Note
tags: "URL Loading System, URLSession, Swift, iOS"
---

[URL Loading System][URL_Loading_System]，通过 URLs，使用标准的网络协议，与服务器交换数据。本文帮助理解 URL 加载系统，并通过示例代码练习如何使用它，示例代码已开源。

<!-- more -->

### 简介

iOS 应用开发少不了要和服务器打交道，而 URL 加载系统就是用来和 http- 和 URL-based 资源通信的。虽然叫做加载系统，但是也包括更新数据到服务器。这种加载通常是异步的，异步的机制能使得你的应用在数据或者错误回来之前，仍然能响应，不出现卡顿。

和 URL 资源的通信依赖 `URLSession` 来实现。通过 `URLSession` 创建一个或多个 `URLSessionTask` 来实现数据的访问、上传、下载，和对流媒体以及 Web socket 的处理。

在创建 `URLSession` 时，可以设置 `URLSessionConfiguration` 来配置 `URLSession` 的行为和策略。通过 `URLSessionDelegate` 来处理 Session 级别的事件响应，例如监听 Session 生命周期的改变并做相应的处理。

<img src="/assets/images/2022-05-04/url-loading-system.svg" width="100%" min-height="500px">

*注：上图是 svg 图片，可放大后观看*

### URLSession

`URLSession` 是整个 URL 加载系统的核心，创建 Session 后，通过方法创建 `URLSessionTask`，然后执行 Session Task 来实现和后端服务的通信。一个应用程序可以有多个 Session，每个 Session 可以创建多个 Session Task (Session Task 只能使用 Session 创建)。应用可以根据需要，尽可能少的创建 Session，来减少对资源的消耗。一般来说，Session 可分为以下三类：

- 常规的 Session。使用 `URLSession` 的构造函数创建，使用 configuration 配置 Session 的权限和策略，使用 delegate 处理 Session 级别的事件响应。
- 临时的 Session。使用默认的配置，不对配置做更改。或者直接使用 URLSession shared 实例，能满足大部分的需要。
- 后台 Session。需要配置，当应用挂起时，能在后台下载或者上传。

### URLSessionTask

`URLSession` 提供了多套 API 来创建不同的 `URLSessionTask`, 来和服务器交换数据。`URLSession` 创建的 `URLSessionTask` 属于挂起状态，需要调用 `resume()` 方法来执行。从目前的 API (iOS 15) 来看, 有 5 类 Session Task：

- `URLSessionDataTask`，最常用的一种，dataTask 返回 Data 类型的数据，存在内存中。不能在后台运行。
- `URLSessionUploadTask`，能方便的设置 request body，上传到服务器。能在后台运行。
- `URLSessionDownloadTask`，下载内容到临时文件，支持后台运行。
- `URLSessionStreamTask`，支持流媒体
- `URLSessionWebSocketTask`，支持 WebSocket

`URLSession` 为每种 Session Task 提供了多种创建/使用方式（为什么要这样？Python 告诉我们解决问题最直接的方法应该有一种，最好只有一种），如下表所示：

| Task type                | Combine API | Async API | Completion Handler API | Normal API |
|:-|:-|:-|:-|:-|
|`URLSessionDataTask`      | ✅ | ✅ | ✅ | ✅ |
|`URLSessionUploadTask`    | - | ✅ | ✅ | ✅ |
|`URLSessionDownloadTask`  | - | ✅ | ✅ | ✅ |
|`URLSessionStreamTask`    | - | - | - | ✅ |
|`URLSessionWebSocketTask` | - | - | - | ✅ |

其中，

- Combine API: 使用 Combine 风格调用 API
- Async API: 使用 `async/await` 关键字来调用 API
- Completion Handler API: 使用回调的方式，在 completionHandler 中处理结果
- Normal API：只创建对应的 Task，需要使用 Task Delegate 获取结果

每种 API 都提供了 URL 和 URLRequest 两个版本，当需要配置 Request 时选用 `URLRequest` 版本。

使用时可根据需要选择合适的 API，个人倾向于 Combine API > Async API > Completion Handler API > Normal API。

### 示例程序

由于 `URLSessionDataTask` 支持所有的四类使用方式，本文以 `URLSessionDataTask` 为例，来展示 URL 加载系统的使用。

#### 准备工作

用来示例的程序是一个展示个人信息的卡片，个人信息和头像地址使用 JSON 格式，存储在服务器上。如下图所示：

![profile view](/assets/images/2022-05-04/profile-view.png)

View 的实现如下所示：

```swift
var body: some View {
    VStack {
      profileCard
    }
    .background(Color(UIColor.systemGroupedBackground).edgesIgnoringSafeArea(.all))
    .onAppear {
      viewModel.loadData(loadingMethodType)
    }
    .onChange(of: loadingMethodType) { newValue in
      viewModel.loadData(newValue)
    }
  }

  private var profileCard: some View {
    HStack {
      Image(uiImage: viewModel.avatar)
        .resizable()
        .frame(width: 120, height: 120)
        .cornerRadius(80)
        .background(.white)
        .clipShape(Circle())
        .padding(8)

      VStack(alignment: .leading) {
        Text(viewModel.name)
          .font(.title)

        HStack {
          Image(systemName: "mail")
          Text("\(viewModel.email)")
        }

        HStack {
          Image(systemName: "link")
          Link("\(viewModel.blog)", destination: viewModel.blogUrl!)
        }
      }
    }
    .frame(maxWidth: .infinity, alignment: .leading)
    .background(backgroundColor)
    .cornerRadius(8)
    .shadow(
      color: shadowColor,
      radius: 4,
      x: 0.0,
      y: 1.0
    )
    .padding()
  }
```

所用的数据来自 JSON 格式的 API. 查看 JSON payload：`curl https://zddhub.com/assets/profile.json`，如下所示：

```json
{
  "name": "zddhub",
  "avatar": "https://zddhub.com/assets/zddhub_big.png",
  "email": "zddhub@gmail.com",
  "blog": "www.zddhub.com",
  "bio": "Just for fun!"
}
```

对应的 Model 为：

```swift
struct Profile: Decodable {
  let name: String
  let avatar: String
  let email: String
  let blog: String
}
```

`ProfileViewModel` 持有一个 Profile 的 model，加载完 API 后，将服务器端的值赋值给 model，View 将自动完成刷新。

```swift
class ProfileViewModel: ObservableObject {
  private var url: URL
  @Published private var model: Profile?
}
```

这里使用一个简单的 MVVM 的架构。示例代码已开源，去 [zddhub/url-loading-system ](https://github.com/zddhub/url-loading-system) 查看完整示例代码。

让我们来看看四种 API 的使用方式：

#### Combine API

Combine API 目前只支持 `URLSessionDataTask`，使用 SwiftUI 的话可以优先选用。

```swift
  URLSession.shared
      .dataTaskPublisher(for: url)
      .tryMap { (data: Data, response: URLResponse) in
        return data
      }
      .decode(type: Profile.self, decoder: JSONDecoder())
      .receive(on: DispatchQueue.main)
      .sink {_ in } receiveValue: { model in
        self.model.send(model)
      }
      .store(in: &cancellable)
```

#### Async API

`async/await` 要求的 iOS 版本比较高，使用前需确认 iOS 版本不小于 15。

> When it was originally announced, Swift concurrency required at least iOS 15, macOS 12, watchOS 8, tvOS 15, or on other platforms at least Swift 5.5.

```swift
  let (data, response) = try await URLSession.shared.data(from: url)

  guard let httpResponse = response as? HTTPURLResponse,
        (200...299).contains(httpResponse.statusCode) else {
    print ("Service error \(String(describing: response))")
    return
  }

  if let mimeType = httpResponse.mimeType, mimeType == "application/json",
      let model = try? JSONDecoder().decode(Profile.self, from: data) {
    self.model.send(model)
  }
```

#### Completion Handler API

这里要注意创建的 `task` 是挂起的状态，需要手动 `resume` 后才能执行。

```swift
  let task = URLSession.shared.dataTask(with: url) { data, response, error in
    if let error = error {
      print("Client error \(error)")
      return
    }

    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
      print ("Service error \(String(describing: response))")
      return
    }

    if let mimeType = httpResponse.mimeType, mimeType == "application/json",
        let data = data,
        let model = try? JSONDecoder().decode(Profile.self, from: data) {
        self.model.send(model)
    }
  }
  task.resume()
```

####  Normal API

这里展示了创建常规的 Session 的方法，虽然 configuration 使用了默认值，什么都没改，但是可以改。由于这里的 `dataTask` 只创建了 task，所以需要设置 delegate，才能拿到 loading 后的值。

```swift
  let configuration = URLSessionConfiguration.default
  let session = URLSession(configuration: configuration)
  let task = session.dataTask(with: url)
  task.delegate = coordinator

  task.resume()

  class Coordinator: NSObject, URLSessionDataDelegate {
    var loadingMethod: LoadingMethod
    private var data: Data? = nil

    init(_ loadingMethod: LoadingMethod) {
      self.loadingMethod = loadingMethod
    }

    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
      guard error == nil, let data = data, let model = try? JSONDecoder().decode(Profile.self, from: data) else {
        self.data = nil
        return
      }

      self.loadingMethod.model.send(model)
      self.data = nil
    }

    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
      if self.data == nil {
        self.data = data
      } else {
        self.data?.append(data)
      }
    }
  }
```

还需要注意的是 `func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data)` 这个方法会被调用多次，如果 data 过大，会分批传送，需要自己拼接数据后再处理。

#### Loading Avatar

上例中都使用 url 作为参数来创建 task，这里展示一种使用 URLRequest 创建 task 的例子，来抓去头像。

正如你在下述代码中看到的那样，可以配置缓存策略和超时时间，下述代码忽略了缓存，每次从服务器请求 API。当然，你也可以对 request 进行更改。

```swift
let request = URLRequest(url: avatarUrl, cachePolicy: URLRequest.CachePolicy.reloadIgnoringLocalAndRemoteCacheData, timeoutInterval: 5.0)

URLSession.shared
  .dataTaskPublisher(for: request)
  .tryMap { (data: Data, response: URLResponse) in
    return data
  }
  .receive(on: DispatchQueue.main)
  .sink { _ in
  } receiveValue: { data in
    self.avatarData = data
  }
  .store(in: &cancellable)
```

### 总结

URL 加载系统是 App 使用最频繁的模块，值得反复练习。主要使用 `URLSession` 创建 Session Task 执行不同的操作。所有 Session Task 中，`URLSessionDataTask` 最常用，所以本文选取 `URLSessionDataTask` 为例，需要熟练掌握。

本文示例完整源码地址 [zddhub/url-loading-system ](https://github.com/zddhub/url-loading-system)。

### 练习

**网上得来终觉浅，绝知此事要躬行**。读完本文后，可以用类似的方法，练习上传 `URLSessionUploadTask` 和下载 `URLSessionUploadTask` 的创建和使用。练习对 `URLSessionConfiguration` 和 `URLRequest` 的配置。

[URL_Loading_System]: https://developer.apple.com/documentation/foundation/url_loading_system
