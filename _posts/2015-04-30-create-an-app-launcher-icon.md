---
layout: post
title: "为Android Studio创建图标"
category: Memo
tags: "AndroidStudio ubuntu"
---

当你在Ubuntu上满心欢喜的安装了最新版的`Android Studio`正准备大干一场的时候，却发现只能从命令行打开，是不是很郁闷呢?

其实做法很简单，Ubuntu上的应用图标配置放在`/usr/share/applications`目录下。复制其中任意一个文件到`android-studio.desktop`,修改对应的参数即可。

<!-- more -->

以下是修改后的文件:

```sh
[Desktop Entry]
Version=1.0
Type=Application
Name=Android Studio
Icon=/opt/android-studio/bin/androidstudio.svg
Exec="/opt/android-studio/bin/studio.sh" %f
Comment=Develop with pleasure!
Terminal=false
StartupWMClass=android-studio
```

什么？还有更简单的方法！原来Android Studio自带了生成launcher图标的方法:
一路点击`Tools->Create Desktop Entry->Ok`就行了。

按花键搜索以下看看吧。

你妹，为什么要我知道这些, 安装的时候为什么不自动生成，差评!
