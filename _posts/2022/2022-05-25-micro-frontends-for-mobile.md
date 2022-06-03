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



CONTENTS
- Benefits
  - Incremental upgrades
  - Simple, decoupled codebases
  - Independent deployment
  - Autonomous teams
  - In a nutshell
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

