# 并行

并行CRD提供了一种容易定义分支列表的方法，每个分支接收发送到并行入口通道的相同的CloudEvent。
通常，每个分支都由一个过滤器函数组成，以保护分支的执行。

并行创建`Channel` 和 `Subscription` 的底层。

## 使用

### 并行的规范

并行有三个部分的规格:

1. `branches`定义了`filter`和`subscriber`对的列表，每个分支一个，还有一个可选的`reply`对象。对于每个分支:
   1. (可选)`filter`被求值，当它返回一个事件时`subscriber`被执行。`filter` and `subscriber`都必须是`可寻址`。
   1. `subscriber`返回的事件被发送到分支`reply`对象。
      当`reply`为空时，事件被发送到`spec.reply`对象。
2. (可选)`channelTemplate`定义了用于创建`Channel`s的模板。
3. (可选)`reply`定义了当分支没有自己的`reply`对象时，每个分支的结果被发送到哪里。

### 并行状态

并行状态分为三个部分:

1. `conditions`，详细描述并行对象的整体状态
1. `ingressChannelStatus` and `branchesStatuses`传递作为并行的一部分创建的底层`Channel` and `Subscription`资源的状态。
1. `address`被公开，以便parallel可以在可寻址的地方使用。
   发送到此地址将针对这个并行的前面的“通道”(与`ingressChannelStatus`相同)。

## 例子

按照[代码示例](https://github.com/knative/docs/tree/main/code-samples/eventing/parallel)学习如何使用并行.
