---
layout: post
title: "Github Project Board 统计"
journal: '2021年22周'
---

践行过敏捷实践的团队都用过白板吧，使用最多的有 Trello，Jira，最近 Github 也出了个 Project，由于和 Github 的血亲关系，很多团队开始转向 Github Project。

Github Project 基本还原了 Trello 的主要功能，但是缺乏丰富的插件，目前还没有看到 Github 开放接口来支持插件。没有插件的 Board 没有统计功能，甚至一个迭代做了多少张卡都需要自己数一数。

苦于此，第一时间能想到的是 Github API，尝试了一番，能很容易拿到 `MovedColumnsInProjectEvent`。当卡片移动时，移动的信息将记录在卡中。但是看不出来是从那个 column 移动到那个 column。最终在[文档](https://docs.github.com/en/enterprise-server@3.0/graphql/reference/objects#movedcolumnsinprojectevent)发现，`projectColumnName` 信息在 preview API 中才会出现。preview API 需要在 headers 里面添加

```
Accept: application/vnd.github.starfox-preview+json
```

注：Preview API 不稳定，可能随时被删除。

GraphQL Client 中的文档并不全，可见在使用 API 时，还是要以官方文档为主。GraphQL 查询脚本如下所示：

```graphql
{
  repository(owner: "zddhub", name: "gpkit") {
    milestones(first: 20) {
      nodes {
        title
        issues(last: 100, states: CLOSED) {
          nodes {
            title
            url
            timelineItems(itemTypes: [MOVED_COLUMNS_IN_PROJECT_EVENT, ADDED_TO_PROJECT_EVENT], first: 100) {
              nodes {
                __typename
                ... on MovedColumnsInProjectEvent {
                  createdAt
                  projectColumnName
                }
                ... on AddedToProjectEvent {
                  createdAt
                  projectColumnName
                }
              }
            }
          }
        }
      }
    }
  }
}
```

注意这里使用 Milestone 来跟踪项目的进度，对没有使用 Milestone 的项目，也可以直接查 issues 获得类似结果。

搞清楚了 API，剩下的就简单了，顺便学习了一下 `deno`, 一个小的统计功能就生成了，目前支持两个指标：

- Cycle Time：卡从开始做(Doing)到完成(Done)所用的时间。
- Lead Time: 代码提交到部署上生产所用的时间。

统计的准确性需要卡片正确的拖动来配合，源码在[这里](https://github.com/zddhub/gpkit)，欢迎参考使用。
