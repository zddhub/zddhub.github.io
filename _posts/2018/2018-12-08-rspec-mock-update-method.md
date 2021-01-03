---
layout: post
title: "Rspec 如何 mock update 方法更新自己？"
category: Mics
tags: "rspec"
---

Rspec 用来 mock 对象方法的语法如下：

```ruby
allow(model).to receive(:message).and_return(result)
```

允许 `model` 接收 `message` 方法返回 `result` 结果。那么，如何来 mock update 方法更新对象自己呢？

<!-- more -->

### mock update 方法更新自己

假设我们需要 mock 测试如下代码：

```ruby
model = Model.find('id')
model.update!(updated: true)
# model.updated = true
```

其实 `allow ... receive` 除了 `and_return` 外，还可以接收 [block](https://relishapp.com/rspec/rspec-mocks/docs/configuring-responses/block-implementation), 我们可以在 block 里更新 model。

mock 代码片段如下：

```ruby
let(:model) { double('Model', {updated: false}) }

allow(Model).to receive(:find).with('id').and_return(model)
allow(model).to receive(:update!).with(updated: true) {
  allow(model).to receive(:updated).and_return(true)
}

...

expect(model.updated).to eq(true)
```

### 使用 double 而不是 OpenStruct

在上述场景中，创建 model 时使用 double 而不是 OpenStruct。double 和 OpenStruct 都可以将 Hash 的键值转化为属性值，区别在于，double 可以 mock 任何方法，而我们不能 mock OpenStruct 没有实现的方法。
