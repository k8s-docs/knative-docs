# 创建一个自定义事件源

如果希望为特定事件生成器类型创建自定义事件源，则必须创建支持将事件从该生成器类型转发到Knative接收器的组件。

这种类型的集成比使用一些更简单的集成类型需要更多的工作，例如[SinkBinding](../sinkbinding/README.md)或[ContainerSource](../containersource/README.md);但是，这提供了最完善的结果，并且是用户最容易使用的集成类型。
通过为源提供自定义资源定义(CRD)而不是通用容器定义，更容易向用户公开有意义的配置选项和文档，并隐藏实现细节。

!!! note
    如果你已经创建了一个新的事件源类型，而不是核心Knative项目的一部分，你可以打开一个拉请求将它添加到[第三方源](../../sources/#third-party-sources)的列表中，并在[Knative Slack](https://slack.knative.dev/)的一个通道中宣布新的源。

    您还可以将事件源添加到[`knative-sandbox`](https://github.com/knative-sandbox)组织中，方法是遵循[创建沙箱存储库](https://github.com/knative/community/blob/main/mechanics/CREATING-A-SANDBOX-REPO.md)的说明。

## 所需的组件

要创建自定义事件源，必须创建以下组件:

| 组件                 | 描述                                                                                    |
| -------------------- | --------------------------------------------------------------------------------------- |
| 收到适配器           | 包含指定如何从生产者获取事件、接收器URI是什么以及如何将事件转换为CloudEvent格式的逻辑。 |
| Kubernetes控制器     | 管理事件源并协调底层接收适配器部署。                                                    |
| 自定义资源定义 (CRD) | 提供控制器用于管理接收适配器的配置。                                                    |

<!--TODO: Add a diagram for this-->

## 使用示例源代码

Knative项目提供了一个示例存储库，其中包含用于基本事件源控制器和接收适配器的模板。

有关使用示例源代码的更多信息，请参见[文档](./sample-repo.md).

## 额外的资源

* 实现CloudEvent绑定接口，[CloudEvent的go sdk](https://github.com/cloudevents/sdk-go) 提供标准访问库，根据需要配置接口。

* 控制器运行时(这是我们通过注入共享的)将特定于协议的配置合并到“通用控制器”CRD中。
