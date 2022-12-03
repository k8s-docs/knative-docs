# 并行

并行CRD提供了一种容易定义分支列表的方法，每个分支接收发送到并行入口通道的相同的CloudEvent。
通常，每个分支都由一个过滤器函数组成，以保护分支的执行。

并行创建`Channel` 和 `Subscription` 的底层。

## 使用

### 并行的规范

并行有三个部分的规格:

1. `branches` defines the list of `filter` and `subscriber` pairs, one per branch,
   and optionally a `reply` object. For each branch:
   1. (optional) the `filter` is evaluated and when it returns an event the `subscriber` is
      executed. Both `filter` and `subscriber` must be `Addressable`.
   1. the event returned by the `subscriber` is sent to the branch `reply`
      object. When the `reply` is empty, the event is sent to the `spec.reply`
      object.
1. (optional) `channelTemplate` defines the Template which will be used to
   create `Channel`s.
1. (optional) `reply` defines where the result of each branch is sent to when
   the branch does not have its own `reply` object.

### 并行状态

Parallel has three parts for the Status:

1. `conditions` which details the overall status of the Parallel object
1. `ingressChannelStatus` and `branchesStatuses` which convey the status of
   underlying `Channel` and `Subscription` resource that are created as part of
   this Parallel.
1. `address` which is exposed so that Parallel can be used where Addressable can
   be used. Sending to this address will target the `Channel` which is fronting
   this Parallel (same as `ingressChannelStatus`).

## 例子

按照[代码示例](https://github.com/knative/docs/tree/main/code-samples/eventing/parallel)学习如何使用并行.
