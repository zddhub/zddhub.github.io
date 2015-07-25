---
layout: post
title: 常用视频转接口
category: Tips
tags: Thunderbolt DP DVI VGA
---

> 求知若饥，虚心若愚 (Stay Hungry, Stay Foolish)

作为一个资深的月光族＋剁手党，终于在双十一（光棍节）的最后一分钟，花光所有存款，入手了一台U2412M，光荣地为571亿尽了绵薄之力。今天快递哥哥就以惊人的速度将其送到了我手上，令人抓狂的是，自带的线没有一根是能用的。于是百科了一下，记录在此，献给和我一样写了好多年程序，也装过很多台电脑，却叫不上接口名称的人（有图有真相哦）。

<!-- more -->

### VGA

VGA（Video Graphics Array）还有一个名称叫D-Sub。VGA接口共有15针，分成3排，每排5个孔，是显卡上应用最为广泛的接口类型，绝大多数显卡都带有此种接口。它传输红、绿、蓝模拟信号以及同步信号（水平和垂直信号）。使用VGA连接设备，线缆长度最好不要超过10米，而且要注意接头是否安装牢固，否则可能引起图像中出现虚影。

![VGA]({{ post.img | replace: '..', site.url }}/asserts/images/2014-11-12/VGA.png)


### DVI

DVI（Digital Visual Interface）是由DDWG（Digital Display Working Group，数字显示工作组）推出的接口标准接口。与VGA都是电脑中最常用的接口，与VGA不同的是，DVI可以传输数字信号，不用再经过数模转换，所以画面质量非常高。目前，很多高清电视上也提供了DVI接口。需要注意的是，DVI接口有多种规范，常见的是DVI-D（Digital）和DVI- I（Integrated）。DVI-D只能传输数字信号，大家可以用它来连接显卡和平板电视。DVI-I则在DVI-D可以和VGA相互转换。

![DVI]({{ post.img | replace: '..', site.url }}/asserts/images/2014-11-12/DVI.png)

### HDMI

HDMI（High Definition Multimedia Interface） 同DVI一样是传输全数字信号的。不同的是HDMI接口不仅能传输高清数字视频信号，还可以同时传输高质量的音频信号。同时功能跟射频接口相同，不过由于采用了全数字化的信号传输，不会像射频接口那样出现画质不佳的情况。

![HDMI]({{ post.img | replace: '..', site.url }}/asserts/images/2014-11-12/HDMI.png)

### DisplayPort
DisplayPort是由视频电子标准协会(VESA)发布的显示接口。作为DVI接口的继任者，DisplayPort将在传输视频信号的同时加入对高清音频信号传输的支持，同时支持更高的分辨率和刷新率。

有意要取代旧有的VGA、DVI和FPD-Link(LVDS)视频传输接口技术。

根据设计，DisplayPort既支持外置显示连接，也支持内置显示连接。VESA希望笔记本厂商不仅使用DisplayPort连接独立显示器，也能使用它来直接连接液晶显示屏和主板，方便笔记本的升级。为此，DisplayPort接口也设计得非常小巧，既方便笔记本的使用，也允许显卡配置多个接口。

![DisplayPort]({{ post.img | replace: '..', site.url }}/asserts/images/2014-11-12/DisplayPort.png)

### Thunderbolt
Thunderbolt（计划代号Light Peak）是由英特尔发表的连接器标准，早期使用光纤，后期与苹果公司共同研发，并改用铜线和苹果的Mini DisplayPort接口外形，因此它既能以双向10Gb/s传输数据（10Gb/s + 10Gb/s），也能兼容Mini DisplayPort设备直接连接Thunderbolt接口传输视频与声音信号，还可连接Apple Thunderbolt Display直接同时输出视频，声音与数据。
2013年推出了新一代Thunderbolt 2，速度翻倍到20Gb/s，以对抗最新版USB 3.1速度翻倍到10Gb/s。

![Thunderbolt]({{ post.img | replace: '..', site.url }}/asserts/images/2014-11-12/Thunderbolt.png)

我的Mini DP->DP 转接线，你快回来，我已经承受不来。。。

