---
layout: post
title: "联合索引与列顺序相关"
journal: '2019年19周'
---

联合索引与列顺序相关，将区分度高的列放在第一位，能帮助提高索引的效率。

之前对数据库完全是懵逼状态，最近由于项目的需要，看了几集 PostgreSQL 的文档，感觉有点意思。不再是懵逼的状态，机缘巧合下得一个改进，提高了查询效率 8 ~ 20 倍（本地 8 倍，线上 20 倍），并得到了客户的表扬，😄。

这里有一篇文章值得一读 [Simple tips for postgresql query optimization](https://dzone.com/articles/simple-tips-for-postgresql-query-optimization)
