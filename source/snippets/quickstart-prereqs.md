<!-- Snippet used in the following topics:
- /docs/getting-started/README.md
- /docs/install/quickstart-install.md
- /docs/getting-started/quickstart-install.md
-->
## 在你开始之前

!!! warning
    Knative`快速启动`环境仅用于实验使用。
    关于可用于生产的安装，请参阅[基于yaml的安装](/docs/install/yaml-install/)
    或[Knative Operator安装](/docs/install/operator/knative-with-operators/).

在开始使用Knative`快速入门`部署之前，您必须安装Knative:

- [kind](https://kind.sigs.k8s.io/docs/user/quick-start){target=_blank} (Kubernetes in Docker) 或 [minikube](https://minikube.sigs.k8s.io/docs/start/){target=_blank} 使您能够使用Docker容器节点运行本地Kubernetes集群。

- [Kubernetes CLI (`kubectl`)](https://kubernetes.io/docs/tasks/tools/install-kubectl){target=_blank} 在Kubernetes集群上运行命令。 可以使用`kubectl`部署应用程序、检查和管理集群资源以及查看日志。

- Knative CLI (`kn`). 有关说明，请参阅下一节。

- 要创建的集群至少需要3个cpu和3个GB的RAM。
