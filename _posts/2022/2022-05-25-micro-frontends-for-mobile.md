---
layout: post
title: "Micro Frontends for Mobile"
category: Article
tags: "Application architecture; Micro Frontends; Mobile"
lang: en
---

On web, We have seen significant benefits from including [Micro Frontends][Micro_Frontends], which breaks up frontend monoliths into many smaller, more manageable pieces, and makes many teams to work simultaneously on a large and complex product. But Micro Frontends for mobile seems stucks these days. In this article we will describe a comprehensive guide to bring these benefits to native mobile apps, as well as cover some of the implementation options that are available for mobile. And finally we will dive deep into a full example application to show the technique.

<!-- more -->

[Micro Frontends][Micro_Frontends], aka MFEs, have continued to gain in popularity since they were marked as 'Adopt' in the Thoughtworks technology radar in 2019 [1][Technology_Radar_Micro_Frontends]. We've seen many projects use this architecture in the last few years, and results turn out very well. So that for many of our colleagues this is becoming the sensible default pattern for building modern enterprise web applications.

In the meantime, Micro frontends for mobile is not going well like Web. It's brought in the Thoughtworks technology radar since 2020, but still stops at 'Trial' stage [2][Technology_Radar_Micro_Frontends_For_Mobile]. Many mobile developers continue to struggle with monolithic codebase. Sadly, there's not much guide that outlines what the Micro Frontends for mobile is and how to do it.

Lately mobile has been becoming the first-class citizen for business requirement shipping. Some companies start with mobile App directly instead of Web App, especially in unicorn companies. And more and more corporations are paying more attention to mobile platform, and the mobile team goes from a dozen people to scores of people, even hundrens. At the same time, declarative UI (SwiftUI, Jetpack compose) makes mobile development become analogous as web development. A lot of web frontend technologies and solutions are crowding into mobile, Micro Frontends are no exception.

Micro Frontends are defined as "An architectural style where independently deliverable frontend applications are composed into a greater whole" [3][Micro_Frontends]. Unlike web application, mobile app has more limitations. For example, we deliver one app to Apple App store or Google Play store instead of several applications. Hotfix is disallowed by mobile platform. So we refined definition of micro frontends for mobile as below:

> An architectural style where independently deliverable frontend components with the same cohesive user experience are composed into a greater whole.

The MFE for mobile is a component that addresses on the same cohesive user experience, owns by a single small team, can be built, tested and deployed to internal application store with an example app independently. The Shell app will integrate all MFEs and be released to public.

[Digram here for MFE mobile]

### Benefits

Even if there are more restrictions, Some of key benefits that still are appealing to mobile developers. And seperating MFE by user experience makes business contexts in a small team highly cohesive.

#### User experience oriented

Each MFE contains one or more components, each component can be big like a screen or small like a button, but all components should address the same user experience. For instance, the Maps MFE only provides a map experience, and the payment MFE only serves payment experience to our users.

Each MFE only serves one cohesive user experience, doesn't and can't rely on other MFEs. User experience is the mininal unit to split MFEs. We still can reserve some interfaces to switch user experience, but the integration work should be done on shell application.

#### Incremental upgrades

For a big legacy codebase, the new option is to split single user experience and rewrite it in a single MFE. Once the first piece succeed, the team will have confidence to rewrite all experiences with new technique.

#### Simple, decoupled codebases

We can split our codebase to smaller ones via user experience boundary from a monolitic codebase. The smaller codebases tend to be simpler and easier for developers to work with. Additionally, the smaller codebases mean less build time and less test time.

It's easy to push some code's button under delivery pressure when working on big codebase. Sometimes just for temporary convenience, the complexity would be brought in two components that should not know about each other. Simple and smaller codebases can help us write high cohesion and low coupling code.

#### Independent deployment

Due to platform limitations, we can't deploy several mobile applications and compose them to an one on app store. But we can use example app to deploy MFE components to internal app library. Each MFE example app should have its own pipline, which builds, tests and deploys it to internal app library. Example app should try to simulate the real app environment.

[Digram here for composing]

#### Autonomous teams

Each MFE has one user experience, so the custodian team has all controls for it without collaborating with other experience teams. And each time can choose their own architectures, tech stack or code styles.

#### Release with a shell app

A shell app is a central app, that integrates all user experiences via MFEs to deliver the whole solution to our users. This central app is just a shell, no more business logic is involved. Each team has a responsibility to integrate their own MFEs to shell app, This is only one mess place that every team can work on. It's not a big deal due to its lightweight implemation. In the stable team, the changes on this repository is just upgrading the version of MFEs.


### The example

[Feed me][Feed_Me] application, a website where users can order food, is a good example which Cam Jackson used to demonstrate MFEs in his [Micro Frontends article][Micro_Frontends]. Here we use the same application but designs it for mobile, hope it can be a standard applicaton that shows MFEs architecture, like [TodoMVC][TodoMVC] does for MV* framework. The business is described as below [4][Feed_Me] :

- There should be a restaurant screen where users can search, filter and browse for restaurants.
- Each restaurant needs its own screen that shows its menu items, and allows a user to order.
- Users should have a profile page where they can see their order history, track delivery, and customise their payment options.

[Design for mobile]

For appearance, we uses a similar design with website. In addition, It supports dark mode.

Throughout the rest of this article, we'll be using this example application wherever we need example code or scenarios.

### Integration approaches

Before integration, we need to split MFEs out first. Given the business above, there are many reasonably approaches to split MFEs out. We recommend spliting MFEs via different user experiences. There are three user experiences, which are located in three tab views:

- Users can search, filter and browser restaurants  - `Browse MFE`
- Users can order food in each restaurant - `RestaurantOrder MFE`
- Users should have an about screen that shows user profile, order history and payment options. - `About MFE`

It will be split to 3 MFEs and 1 shell app.

[Diagram for integration]

By the way, Our demo app is an iOS app, that is built via SwiftUI and Swift package. If you like you can also use UIKit or podfile. It's worth pointing out that this architecture is suitable for Android as well. We choose iOS here because we are familar with them.

#### Build-time integration

It is a major, somehow a only approach to integrate MFEs to a shell app. It's to publish each MFE as a Swift package, and have shell app that include all MFEs as its dependencies. Here is your `Package.resolved` might look:

```swift
{
  "pins" : [
    {
      "identity" : "about",
      "kind" : "remoteSourceControl",
      "location" : "https://github.com/micro-frontends-mobile/About",
      "state" : {
        "revision" : "e96fdbada870a92fccf918b8b250f9c9d4751149",
      }
    },
    {
      "identity" : "browse",
      "kind" : "remoteSourceControl",
      "location" : "https://github.com/micro-frontends-mobile/Browse",
      "state" : {
        "revision" : "a3590f11f00016e94de4fcb0aa193b14dacd9a57",
      }
    },
    {
      "identity" : "restaurantorder",
      "kind" : "remoteSourceControl",
      "location" : "https://github.com/micro-frontends-mobile/RestaurantOrder",
      "state" : {
        "revision" : "e82e3faa1ccf669f115166aa73e1954dbd228581",
      }
    }
  ],
  "version" : 2
}
```

Due to bring in coupling at build and release stage, This approach is not recommended on Web, but there is no other choices on Mobile native.

#### Run-time integration via WKWebView

Benefited from WKWebView, you can use to incorporate web content seamlessly into you app. It's a good time to use it When your app’s content changes frequently.

```swift
var webView: WKWebView = WKWebView()
let request = URLRequest(url: self.url, cachePolicy: .returnCacheDataElseLoad)
webView.load(request)
```

Embedding a WKWebView object programmatically into your view hierarchy is another way, It makes WKWebView have native experience to use your navigation delegate to modify the web view’s navigation behavior. The users can't feel differently to use it, but we will get automatically updates via web service after our app is released.

We recommend hosting a WKWebView into the MFE, whether it could serve the whole user experience or not.

Notice your app might be rejected if the core features and functionality dynamically with web technology like HTML5, due to [App Store Review Guidelines][App_Updates_for_HTML5_Apps].


### Micro Frontends Environment

After spliting monolithic app to smaller MFEs, the custodian teams can work autonomously. It's possible to reinvent the wheel when they makes all decisions without considering other teams. So it's a big win to setup a foundational MFE environment that provides common capabilities, which allows custodian team to only focus on their own user experiences.

[Diagram here]

There are some common capabilities should (not must) put into MFE environment:
- UI components
- Configuration
- Router
- Authentication and Authorization
- Network
- Cache
- Tag analytics
- Logger and monitor system integration
- Create MFE App
...

It's a good way to create a team to focus on MFE environment setup, And make MFEs team easlier.


### Cross MFE communication

If you found one MFE has many communications with other one, or communicate with many MFEs, You might rethink your MFE seperation. A good MFE seperation can reduce commnunication among them, but it can't avoid it.

Navigation is a common commnuication, for example, when we click restaurant card, it will bring us in order screen. We can solve this problem via router, a common tool in our MFE environment, as mentioned above.

Another common way is to use Notification, which broadcasts information through a notification to all registered observers. MFE can subscribe notification to exchange information.


### Backend services

Since we have seperated our frontends via user experiences, it's straightforward to seperate [micro services][Micro_Services] using user experiences as well. Comparing [BFF][BFF] pattern, User experiences API can be used by both mobile and web, and also help write decoupling code on server side.

[Diagram here]


### Decision Record

Idea exchanges, inspirations and thinking process is a valuabe possession for a team. What/how/why the team makes current decision is worthy to record. It's a good way to share contexts in years.


### The example in detail

The rest of this article will be a detailed explanation how feed me application can be implemented. We'll focus mostly on how to use Swift package to host MFEs, and how to integrate them in shell app. And the full source code can be seen on [Github][micro-frontends-mobile].

We didn't setup our own services and all assets are coming from Cam Jackson's [demo][micro-frontends-demo].


#### The MFE Environment

Let's start from [Env][Env]. Env is a singleton of the MFE environment, it hosts and manages all common capabilities for MFE. In our demo, it is implemented to an `ObservableObject` and treated as `EnvironmentObject`. 

```swift
public class Env: ObservableObject {
  public private(set) static var shared: Env!

  public static func initialize(router: Router = Router(), configuration: Configuration = Configuration()) -> Env {
    if shared != nil {
      return shared
    }

    shared = Env(router: router, configuration: configuration)
    return shared
  }

  init(router: Router, configuration: Configuration) {
    self.router = router
    self.configuration = configuration
  }

  public let router: Router
  public let configuration: Configuration
}
```

Currently env only hosts two capabilities:

##### Configuration

In our demo, we put some theme configurations like `backgroundColor` and `tabBarTintColor` which will be shared in all MFEs to make visiual consistency. And also, the color can be changed to adapt dark mode automatically. Another configurations are host, here we shamelessly use Cam Jackson's APIs.

```swift
public struct Configuration {
  public init() {}
  public var backgroundColor: Color {
    UI.color(token: .backgroundColor)
  }

  public var tabBarTintColor: Color {
    UI.color(token: .tabBarTintColor)
  }

  public var host: String {
    "https://content.demo.microfrontends.com"
  }

  public var demoHost: String {
    "https://demo.microfrontends.com"
  }
}
```

##### Router

The router is carefully designed for navigation. Our demo uses it to switch user experience from browse restaurant list to order screen.

```swift
open class Router: ObservableObject {
  public init() {}
  open func navigate<Source>(to: String, source: () -> Source) -> AnyView where Source : View {
    AnyView(NavigationLink {
      route(to)
    } label: {
      source()
    })
  }

  open func route(_ to: String) -> AnyView {
    AnyView(Text("\(to)"))
  }
}
```

Instead of encapsulating destination view, here `Router` encapsulate `NavigationLink`. It needs to be wrappered by `NavigationView` when we use it. There is a detailed example later. Here, `Router` has a default implementation, it will route to a View with Text. You can overwrite it in your app. The typical route function should be like below:

```swift
  override func route(_ to: String) -> AnyView {
    if to.match("/restaurants") {
      return BrowseView()
    } else if to.match("/restaurants/{id}") {
      return OrderView()
    } else {
      ...
    }
  }
```

In our design. ShellApp is a good place to overwrite Router. You can also design your own Router pattern, it is a principle not to create dependencies among MFEs, not to let Env know you have a component named OrderView, that is defined on RestaurantOrder MFE.

Env is a facade that handles all fundational capabilities for all your MFEs and shell APP, you can add more mentioned on Micro Frontends Environment section.

#### The micro frontends

We use [Swift packages][swift_packages] to manager our MFEs, it's easy to change to [Cocopods][cocoapods] if you like. Each MFE is a Swift package. Take Browse MFE as an example, The project folder structure is like below:

```sh
.
├── BrowseExample
│   ├── BrowseExample
│   └── BrowseExample.xcodeproj
├── Package.swift
├── README.md
├── Sources
│   └── Browse
└── Tests
    └── BrowseTests
```

Except package files, we add an example app named `BrowseExample`, which will depend on Browse MFE package locally. Adding Browse MFE as a local package here is a good idea, we can open `BrowseExample` app and edit the package. In contrast, we can't edit remote package even if we can see the source code.

An example app is important in our scenario, we need rely on it to deliver our MFEs to internal app library for test purpose. And also, it's a good place to elaborate on how to integrate.

##### Browse MFE

[Browse MFE][Browse] is a restaurant screen where users can search, filter and browse for restaurants. 

[Screenshot here]

Let's start from example app, `BrowseView` is a screen level component which is imported from Browse MFE.

```swift
import SwiftUI
import Browse
import Env

struct ContentView: View {
  var body: some View {
    NavigationView {
      BrowseView()
        .navigationBarHidden(true)
        .environmentObject(Env.initialize())
    }
  }
}
```

Since there is a navigation, when we click `RestaurantCard`, it will navigate to other screen. So we inject Env shared instance to BrowseView as a environment object, which can be read by any child by using EnvironmentObject.

```swift
import Env

struct RestaurantListView: View {
  var restaurantList: RestaurantList
  @EnvironmentObject var env: Env

  var body: some View {
    ScrollView {
      VStack(spacing: 16) {
        ForEach(restaurantList) { restaurant in
          env.router.navigate(to: restaurant.url.path) {
            RestaurantCard(restaurant: restaurant)
          }
        }
      }
    }
  }
}
```

As above, we wrapped `RestaurantCard` with `env.router.navigate(to: restaurant.url.path)`, it will navigate to `OrderView` screen finally, but In Browse MFE, we don't know there is a screen named `OrderView`, that is implemented in Order MFE. Env.router helps us to write decoupling code.

##### Order MFE

[Two Screenshots here]

[RestaurantOrder MFE][RestaurantOrder] is a order screen that users can review, choose and order foods in each restaurant. There are two scenarios to show order view in Feed me app:

1. When you click `RestaurantCard`, it will navigate to order view with specific restaurant information

    We pass a specific url and show order information as below:

    ```swift
    public struct OrderView: View {
      public init(url: URL) {
        self.url = url
      }

      public var body: some View {
        ...
      }
    }
    ```

2. When you click `Surprise Me` tab, it will show order view with a random restaurant information

    Here, we don't need a url, we will load restaurant list, and random pick up one to show.


    ```swift
    public struct RandomOrderView: View {
      public init() {}
      public var body: some View {
        Group {
          if viewModel.url != nil {
            OrderView(url: viewModel.url!)
          } else {
            ProgressView()
          }
        }
        .onAppear {
          viewModel.loadData()
        }
      }
    }
    ```

    Maybe you have noticed, we need to fetch restaurant list when we show `RestaurantList`. And here we fetch it again for Suprise tab. Although we can fix this issue using service side cache or client side cache in Env, it is a kind of cost we need to afford in MFE architecture.

All of these two scenarios, we think they are the same user experiences, so we use one MFE instead of two ones, but export two public views.

There is a question: Which view should be displayed on the example app?. The answer is both definitely.

##### About MFE

[Screenshot here]

[About MFE][About] should be an screen that shows user profile, order history and payment options. In our example, we simplify it as a static content screen. But we get some information from web service via WKWebView to show run-time integration way.

We wrapped WKWebView in `AboutWebView` and load web content as below:

```swift
struct AboutWebView: UIViewRepresentable {
  let url: URL
  var webView: WKWebView = WKWebView()
  @Binding var dynamicHeight: CGFloat

  func makeUIView(context: UIViewRepresentableContext<AboutWebView>) -> WKWebView {
    let request = URLRequest(url: self.url, cachePolicy: .returnCacheDataElseLoad)
    webView.load(request)

    return webView
  }

  ...
}
```

In order to let users have native experience, we delegate link click event to open Safari.

```swift
  func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction) async -> WKNavigationActionPolicy {
    if navigationAction.navigationType == .linkActivated, let url = navigationAction.request.url, await UIApplication.shared.canOpenURL(url) {
      DispatchQueue.main.async {
        UIApplication.shared.open(url)
      }
      return .cancel
    } else {
      return .allow
    }
  }
```

##### Shell App

[ShellApp][ShellApp] is a centrel app that integrate all MFEs to the whole one. It's an only app that wil be deployed to our end users.

Let's have a look, the content view is pretty simple, right?

```swift
struct ContentView: View {

  init() {
    _ = Env.initialize(router: AppRouter() as Router)
  }

  var body: some View {
    TabView {
      browse
        .tabItem {
          Image(systemName: "magnifyingglass")
          Text("Restaurant")
        }
      RandomOrderView()
        .tabItem {
          Image(systemName: "shuffle")
          Text("Surprise me")
        }
      AboutView()
        .tabItem {
          Image(systemName: "person")
          Text("About")
        }
    }
  }
}
```

Let's focus on how to navigate to OrderView when the user click RestaurantCard, which is sub childrend of `BrowseView`. There are two things need be done in shell APP.

1. Wrap BrowserView with NavigationView

    ```swift
      var browse: some View {
        NavigationView {
          BrowseView()
            .foregroundColor(.primary)
            .navigationBarHidden(true)
            .environmentObject(Env.shared)
        }
      }
    ```
2. Overwrite Router to inject `OrderView`

    ```swift
    class AppRouter: Router {
      override func route(_ to: String) -> AnyView {
        AnyView(
          OrderView(url: url(to))
            .navigationBarTitleDisplayMode(.inline)
        )
      }

      private func url(_ to: String) -> URL {
        URL(string: "\(Env.shared.configuration.host)\(to)")!
      }
    }
    ```

Wow, We are finished! Explore full source code on [Github][micro-frontends-mobile].

CONTENTS


Downsides
  - Payload size
  - Environment differences
  - Operational and governance complexity
Conclusion


[Micro_Frontends]: https://martinfowler.com/articles/micro-frontends.html
[Technology_Radar_Micro_Frontends]: https://www.thoughtworks.com/radar/techniques/micro-frontends
[Technology_Radar_Micro_Frontends_For_Mobile]: https://www.thoughtworks.com/radar/techniques/micro-frontends-for-mobile
[Feed_Me]: https://demo.microfrontends.com/
[TodoMVC]: https://todomvc.com/
[BFF]: https://samnewman.io/patterns/architectural/bff/
[Micro_Services]: https://martinfowler.com/articles/microservices.html
[micro-frontends-mobile]: https://github.com/micro-frontends-mobile
[micro-frontends-demo]: https://github.com/micro-frontends-demo
[Env]: https://github.com/micro-frontends-mobile/Env
[Browse]: https://github.com/micro-frontends-mobile/Browse
[RestaurantOrder]: https://github.com/micro-frontends-mobile/RestaurantOrder
[About]: https://github.com/micro-frontends-mobile/About
[ShellApp]: https://github.com/micro-frontends-mobile/ShellApp
[swift_packages]: https://developer.apple.com/documentation/swift_packages
[cocoapods]: https://cocoapods.org/
[App_Updates_for_HTML5_Apps]: https://developer.apple.com/news/?id=09062019b
