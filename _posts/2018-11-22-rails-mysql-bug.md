---
layout: post
title: "Rails、mysql 异常"
category: Mics
tags: "Rails mysql2"
---

这两周做了一个 `rails` 项目，有重新认识 `rails` 之感，接着重新认识了一下自己。把其中一个有意思的 bug 拿出来分析一下。

<!-- more -->

在使用了 `gem mysql2` 时，本地 Rails 环境直接抛异常，前半部分 log 如下 (详细的 log 见文章末尾)：

```
/Users/zddhub/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/mysql2-0.5.2/lib/mysql2/client.rb:90: [BUG] Segmentation fault at 0x0000000000000000
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-darwin18]

-- Crash Report log information --------------------------------------------
   See Crash Report log file under the one of following:
     * ~/Library/Logs/DiagnosticReports
     * /Library/Logs/DiagnosticReports
   for more details.
Don't forget to include the above Crash Report log file in bug reports.

-- Control frame information -----------------------------------------------
c:0042 p:---- s:0223 e:000222 CFUNC  :connect
c:0041 p:0612 s:0211 e:000210 METHOD /Users/zddhub/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/mysql2-0.5.2/lib/mysql2/client.rb:90 [FINISH]
c:0040 p:---- s:0197 e:000196 CFUNC  :new
c:0039 p:0104 s:0192 e:000191 METHOD /Users/zddhub/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/activerecord-5.2.1/lib/active_record/connection_adapters/mysql2_a
```

在 `docker` 环境和小伙伴机器上测试了一下，bug 不复现，定位为本地环境问题。自作聪明的投机了一会，重新卸载安全了所有 ruby、 rails 环境。发现并没有解决问题。

读 mysql2 源代码，找到问题代码[lib/mysql2/client.rb:90](https://github.com/brianmario/mysql2/blob/c578718cb9adc49117358162256420d977381060/lib/mysql2/client.rb#L90)：

```
88:  conn_attrs = parse_connect_attrs(opts[:connect_attrs])
89:
90:  connect user, pass, host, port, database, socket, flags, conn_attrs
91:end
```

发现是数据库连接问题。什么会影响数据库连接呢？细读 [mysql2 README](https://github.com/brianmario/mysql2), 发现这个库比较奇葩，会安装所依赖的二进制和库文件。找到安装目录，发现安装了 mysql 在 `/usr/local/mysql/bin/mysql` 目录下。而我手动安装过一个 mysql 客户端，在 `/usr/bin` 下，rails 运行时调用了手动安装过的 mysql 命令，导致段错误，连接不上数据库。

删除手动安装的 mysql 相关的二进制，bug 不复现。

![mysql bin](/assets/images/2018-11-22/mysql_command.jpg)

详细的 log 在[这里](/assets/images/2018-11-22/mysql2_bug.log)。

解决这个问题并不难，难得是解决这个问题的思路。认真看 log， 主要是真正思考。
