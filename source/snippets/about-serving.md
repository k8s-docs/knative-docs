<!-- Snippet used in the following topics:
- /docs/serving/README.md
- /docs/concepts/README.md
-->
Knative Serving将一组对象定义为Kubernetes自定义资源定义(CRDs)。
这些资源用于定义和控制无服务器工作负载在集群上的行为。

![显示服务资源如何相互协调的关系图。](https://github.com/knative/serving/raw/main/docs/spec/images/object_model.png)

主要的Knative服务资源是服务、路由、配置和修订:

- [服务](https://github.com/knative/specs/blob/main/specs/serving/knative-api-specification-1.0.md#service){target=_blank}:
  `service.serving.knative.dev` 资源自动管理您的工作负载的整个生命周期。
  它控制其他对象的创建，以确保你的应用程序在每次服务更新时都有路由、配置和新的修订。
  服务可以定义为始终将通信路由到最新修订或固定修订。

- [路由](https://github.com/knative/specs/blob/main/specs/serving/knative-api-specification-1.0.md#route){target=_blank}:
  `route.serving.knative.dev` 资源将一个网络端点映射到一个或多个修订。
  您可以通过多种方式管理流量，包括部分流量和命名路由。

- [配置](https://github.com/knative/specs/blob/main/specs/serving/knative-api-specification-1.0.md#configuration){target=_blank}:
  `configuration.serving.knative.dev` 资源为您的部署维护所需的状态。
  它在代码和配置之间提供了清晰的分离，并遵循了十二因素应用程序方法。
  修改配置会创建一个新的修订。

- [修订](https://github.com/knative/specs/blob/main/specs/serving/knative-api-specification-1.0.md#revision){target=_blank}:
  `revision.serving.knative.dev`资源是对工作负载进行的每次修订的代码和配置的时间点快照。
  修订是不可变的对象，只要有用就可以保留。
  Knative服务修订可以根据传入的流量自动缩放。

有关资源及其交互的更多信息，请参见`serving`Github存储库中的[资源类型概述](https://github.com/knative/specs/blob/main/specs/serving/overview.md){target=_blank}。