# 序列

序列CRD提供了一种方法来定义将被调用的函数的按顺序列表。
每个步骤都可以修改、筛选或创建一种新的事件类型。
序列在底层创建“频道”和“订阅”。

!!! info
    序列需要"hairpin"流量。
    请验证您的 Pod 可以通过服务IP到达自己。
    如果"hairpin"流量不可用，您可以联系您的集群管理员，因为这是一个集群级别(通常是CNI)设置。


## 使用

### 序列规范

序列有三个部分的规范:

1. `Steps`，它定义了`Subscriber`的按顺序列表，也就是按列出的顺序执行的函数。
   这些是使用`messaging.v1.SubscriberSpec`指定的，就像创建`Subscription`时一样。
   每个步骤都应该是`Addressable`。
2. `ChannelTemplate`定义了用于在步骤之间创建`Channel`的模板。
3. `Reply`(可选)引用序列中最后一步的结果被发送到的位置。

### 序列状态

序列有四个部分的状态:

1. 详细描述序列对象的整体状态的条件
2. `ChannelStatuses`，它传递作为该序列一部分创建的底层“通道”资源的状态
   它是一个数组，每个Status对应于Step号，
   因此数组中的第一个条目是第一个Step之前的 `Channel` 的Status。
3. `SubscriptionStatuses`，它传递作为该序列的一部分创建的底层“订阅”资源的状态。
   它是一个数组，每个Status对应于Step号，因此数组中的第一个条目是“Subscription”，
   创建它是为了将第一个通道连接到“Steps”数组中的第一个步骤。
4. `AddressStatus` 是公开的，以便Sequence可以在可以使用Addressable的地方使用。
   发送到此地址将针对序列中第一步前面的`Channel`。

## 例子

对于下面的每个示例，都使用[`PingSource`](../../../eventing/sources/ping-source/README.md)作为事件源。

我们还使用一个非常简单的[转换器](https://github.com/knative/eventing/blob/main/cmd/appender/main.go)，它对传入的事件执行非常简单的转换，以演示它们已经通过了每个阶段。

### 无应答序列

对于第一个例子，我们将使用直接连接到`PingSource`的3步`Sequence`。
每个步骤都简单地附加在`- Handled by <STEP NUMBER>`上，例如`Sequence`中的第一个步骤将接受传入消息并将"- Handled by 0"附加到传入消息。

参见[无应答的序列(终端最后一步)](../sequence/sequence-terminal/README.md).

### 有应答序列

对于下一个例子，我们将使用直接连接到`PingSource`的相同的3步`Sequence`。
每个步骤都简单地附加在`- Handled by <STEP NUMBER>`上，例如`Sequence`中的第一个步骤将接受传入消息并将"- Handled by 0"附加到传入消息。

唯一的区别是我们将使用`Subscriber.Spec.Reply`字段将最后一步的输出连接到事件显示Pod。

参见[有应答序列(最后一步产生输出)](../sequence/sequence-reply-to-event-display/README.md).

### 将序列链接在一起

对于下一个例子，我们将使用直接连接到`PingSource`的相同的3步`Sequence` 。
每个步骤都简单地附加在`- Handled by <STEP NUMBER>`上，例如`Sequence`中的第一个步骤将接受传入消息并将"- Handled by 0"附加到传入消息。

唯一的区别是，我们将使用`Subscriber.Spec.Reply`字段将最后一步的输出连接到另一个`Sequence`，该`Sequence`执行与第一个管道相同的消息修改(只是步骤不同)。

参见[将序列连接在一起](../sequence/sequence-reply-to-sequence/README.md).

### 使用带有代理/触发器模型的序列

你也可以创建一个目标为`Sequence`的触发器。
这次我们将连接`PingSource`将事件发送到`Broker`，然后我们将让`Sequence`将产生的事件发送回Broker，以便`Sequence`的结果可以被其他触发器观察到。

参见[使用带有代理/触发器模型的序列](../sequence/sequence-with-broker-trigger/README.md).
