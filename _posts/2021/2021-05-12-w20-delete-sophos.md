---
layout: post
title: "卸载 Sophos"
journal: '2021年19周'
---

当新系统 macOS Big Sir `version 11.3.1 (20E241)` + Sophos `version 10.0.4`, 遇上  Charles (`v4.6.1`), 不出两分钟，操作系统直接关闭，连黑屏的机会都没有。
在没关机的两分钟内，iOS Simulator 模拟器 Wifi 连不上网。而且稳定复现。


如何来修这个问题呢？


一番折腾后发现，卸载 Sophos 后 Charles 和模拟器正常工作，此时再安装 Sophos 发现安装不上，直接 Failed，也没有多余的 log。再次探索一番发现，Sophos 有两个进程 `com.sophos.endpoint.networkextension` 和 `com.sophos.endpoint.scanextension` 仍然在运行，重启或者强杀都无济于事。要论流氓，杀毒软件绝对首当其冲，卸载是卸不干净的。

原来这两个程序被放在 `/Library/SystemExtensions` 下，强删失败。一番查询后发现，原来是 `SIP（System Integrity Protection ）`阻止了删除操作。

重启系统，按住 Command + R 进入恢复模式后，Utilities ->  Terminal 后，

```
csrutil disable
```

重启后，顺利删除了 `/Library/SystemExtensions` 目录下的执行文件，Sophos 的两个流氓进程也消失不见。再次安装 Sophos 成功，Charles 重新工作。

最后不要忘记了 enable `SIP`。
