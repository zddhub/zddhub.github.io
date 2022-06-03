---
layout: post
title: "Micro Frontends for Mobile"
category: Article
tags: "Application architecture; Micro Frontends; Mobile"
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



CONTENTS
- The example
- Integration approaches
  - Build-time integration
  - Run-time integration via WKWebView
- Styling
- Shared component libraries
- Cross-application communication
- Backend communication
- Testing
- The example in detail
  - The container
  - The micro frontends
  - Cross-application communication via routing
  - Common content
  - Infrastructure
Downsides
  - Payload size
  - Environment differences
  - Operational and governance complexity
Conclusion


[Micro_Frontends]: https://martinfowler.com/articles/micro-frontends.html
[Technology_Radar_Micro_Frontends]: https://www.thoughtworks.com/radar/techniques/micro-frontends
[Technology_Radar_Micro_Frontends_For_Mobile]: https://www.thoughtworks.com/radar/techniques/micro-frontends-for-mobile

