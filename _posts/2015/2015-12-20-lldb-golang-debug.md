---
layout: post
title: "LLDB debug Golang"
category: Memo
tags: "lldb debug Golang"
---

> 除了 GDB，还可以使用 LLDB debug go code

熟练调试工具是程序员的必备技能，lldb 的发展势头，大有取缔 gdb 的趋势，因此需要把 lldb 加入到自己的工具箱里。

# LLDB 命令

[LLDB](http://lldb.llvm.org/) 的所有命令都遵从以下格式：

    <noun> <verb> [-options [option-value]] [argument [argument...]]

对于常用命令，可设置别名，降低输入负担。(可通过别名设置和 GDB 相同的语法，降低记忆负担)。

<!-- more -->

常用命令集

    go build -gcflags "-N -l" -o test test.go 关闭编译优化
    b test.go:10    设置断点，如果项目中存在同名文件，会根据行号对应的内容选择一个，或者两个都选
    run             运行到断点处
    n               运行到下一行
    ni              回到当前运行的代码片段
    p varname       打印变量值
    frame variable  打印当然 frame 中的所有变量
    r               重新运行程序


# LLDB 练习

以下通过一个例子程序的调试，了解 LLDB 的基本用法。待调试程序如下：

```go
package main

import (
  "fmt"
  "time"
)

func counting(c chan<- int) {
  for i := 0; i < 10; i++ {
    time.Sleep(1 * time.Second)
    c <- i
  }
  close(c)
}

func main() {
  msg := "Starting main"
  fmt.Println(msg)
  bus := make(chan int)
  msg = "starting a gorountie"
  go counting(bus)
  for count := range bus {
    fmt.Println("count:", count)
  }
}
```

编译源码，使用 `-gcflags "-N -l"` 参数关闭编译器代码优化。

    $ go build -gcflags "-N -l" -o test test.go

通过 lldb 命令启动调试:

    $ lldb test

可通过 `run (r)` 运行整个程序，如下所示，和我们在终端运行命令结果一致。

```sh
(lldb) run
Process 3242 launched: '/Users/zdd/Projects/Go/src/github.com/zddhub/test/test' (x86_64)
Starting main
count: 0
count: 1
count: 2
count: 3
count: 4
count: 5
count: 6
count: 7
count: 8
count: 9
Process 3242 exited with status = 0 (0x00000000)
(lldb)
```

断点命令

    (lldb) breakpoint set --file test.go --line 17
    (lldb) b test.go:17 #简写

为程序设置断点并查看断点，

```sh
(lldb) b test.go:10
Breakpoint 1: where = test`main.counting + 43 at test.go:10, address = 0x000000000000206b
(lldb) b test.go:23
Breakpoint 2: where = test`main.main + 514 at test.go:23, address = 0x00000000000022f2
(lldb) br list
Current breakpoints:
1: file = 'test.go', line = 10, locations = 1
  1.1: where = test`main.counting + 43 at test.go:10, address = test[0x000000000000206b], unresolved, hit count = 0

2: file = 'test.go', line = 23, locations = 1
  2.1: where = test`main.main + 514 at test.go:23, address = test[0x00000000000022f2], unresolved, hit count = 0
```

运行一次到第一个断点

```sh
(lldb) run
Process 3302 launched: '/Users/zdd/Projects/Go/src/github.com/zddhub/test/test' (x86_64)
Starting main
Process 3302 stopped
* thread #1: tid = 0x11ca78, 0x000000000000206b test`main.counting(c=0x000000c820070000) + 43 at test.go:10, stop reason = breakpoint 1.1
    frame #0: 0x000000000000206b test`main.counting(c=0x000000c820070000) + 43 at test.go:10
   7
   8    func counting(c chan<- int) {
   9      for i := 0; i < 10; i++ {
-> 10       time.Sleep(1 * time.Second)
   11       c <- i
   12     }
   13     close(c)
(lldb)
```

之后，通过 `c` 或 `n` 继续运行程序。

查看变量的值:

    p varname

```sh
* thread #10, stop reason = breakpoint 3.1
    frame #0: 0x0000000001093939 xx`main.counting(c=0x000000c420076060) at xx.go:10
   7
   8    func counting(c chan<- int) {
   9      for i := 0; i < 10; i++ {
-> 10       time.Sleep(1 * time.Second)
   11       c <- i
   12     }
   13     close(c)
(lldb) p i
(int) i = 0
(lldb) p c
(chan<- int) c = 0x000000c420076060
```


查看当前 frame 中的所有变量

    frame variable

```sh
(lldb) frame variable
(chan<- int) c = 0x000000c420076060
(int) i = 0
([]*runtime.g) runtime.allgs = (len 5, cap 8) {
  [0] = 0x000000c420000180
  [1] = 0x000000c420000600
  [2] = 0x000000c420000a80
  [3] = 0x000000c420000f00
  [4] = 0x000000c420001080
}
```

命令 `ni` 回到当前断点所在的代码片段

```sh
(lldb) ni
Process 35396 stopped
* thread #10, stop reason = instruction step over
    frame #0: 0x0000000001093941 xx`main.counting(c=0x000000c420076060) at xx.go:10
   7
   8    func counting(c chan<- int) {
   9      for i := 0; i < 10; i++ {
-> 10       time.Sleep(1 * time.Second)
   11       c <- i
   12     }
   13     close(c)
```

watch 变量的值：

```sh
(lldb) watch set var i
Watchpoint created: Watchpoint 1: addr = 0xc820026798 size = 8 state = enabled type = w
    watchpoint spec = 'i'
    new value: 0
(lldb) c
Process 3302 resuming

Watchpoint 1 hit:
old value: 0
new value: 1
Process 3302 stopped
* thread #1: tid = 0x11ca78, 0x00000000000020be test`main.counting(c=0x000000c820070000) + 126 at test.go:9, stop reason = watchpoint 1
    frame #0: 0x00000000000020be test`main.counting(c=0x000000c820070000) + 126 at test.go:9
   6    )
   7
   8    func counting(c chan<- int) {
-> 9      for i := 0; i < 10; i++ {
   10       time.Sleep(1 * time.Second)
   11       c <- i
   12     }
(lldb)
```

列举所有观测变量：

```sh
(lldb) watch list -v
Number of supported hardware watchpoints: 4
Current watchpoints:
Watchpoint 1: addr = 0xc820026798 size = 8 state = enabled type = w
    watchpoint spec = 'i'
    old value: 0
    new value: 1
    hw_index = 0  hit_count = 1     ignore_count = 0
Watchpoint 2: addr = 0xc8200267b8 size = 8 state = enabled type = w
    watchpoint spec = 'c'
    new value: 0x000000c820070000
    hw_index = -1  hit_count = 0     ignore_count = 0
(lldb)
(lldb) fr v
(long) i = 2
(chan<- int) c = 0x000000c820070000
```
