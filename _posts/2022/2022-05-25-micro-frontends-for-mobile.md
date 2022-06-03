---
layout: post
title: "Micro Frontends for Mobile"
category: Article
tags: "Application architecture; Micro Frontends; Mobile"
---

On web, We have seen significant benefits from including [Micro Frontends][Micro_Frontends], which breaks up frontend monoliths into many smaller, more manageable pieces, and makes many teams to work simultaneously on a large and complex product. But Micro Frontends for mobile seems stucks these days. In this article we will describe a comprehensive way to bring these benefits to native mobile apps, as well as cover some of the implementation options that are available for mobile. And finally we will dive deep into a full example application to show the technique.

<!-- more -->

[Micro Frontends][Micro_Frontends] technique 

Mobile has been becoming the first-class citizen for business requirement shipping.


> "An architectural style where independently deliverable frontend applications are composed into a greater whole"

An architectural style where independently deliverable frontend components with a single cohesive user experience are composed into a greater whole

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

