<!-- Snippet used in the following topics:
- /docs/functions/README.md
-->
Knative函数为在Knative上使用函数提供了一个简单的编程模型，不需要深入了解Knative、Kubernetes、容器或dockerfile。

Knative函数使您能够通过使用`func`  CLI轻松创建、构建和部署作为Knative服务的无状态事件驱动函数。

当您构建或运行一个函数时，一个[开放容器初始化(OCI)格式](https://opencontainers.org/about/overview/){target=_blank} 容器映像将自动为您生成，并存储在容器注册表中。
每次更新代码并运行或部署它时，容器映像也会更新。

您可以通过使用 `func`  CLI或使用Knative CLI的 `kn func` 插件来创建函数和管理函数工作流。