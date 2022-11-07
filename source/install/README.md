# 安装 Knative

您可以使用以下部署选项之一在集群上安装服务组件、事件组件或两者都安装:

- 使用[Knative Quickstart plugin](quickstart-install.md)安装预配置的Knative本地发行版，用于开发目的。

- 使用基于yaml的安装来安装生产就绪部署:
    - [使用YAML安装Knative服务](yaml-install/serving/install-serving-with-yaml.md)
    - [使用YAML安装Knative事件](yaml-install/eventing/install-eventing-with-yaml.md)

- 使用[Knative Operator](operator/knative-with-operators.md)安装和配置一个生产就绪部署。

- 遵循供应商管理[Knative产品](knative-offerings.md)的文档.

您还可以[升级现有的Knative安装](upgrade/README.md).

!!! note
    Knative安装说明假设您正在运行带有Bash shell的Mac或Linux。
<!-- TODO: Link to provisioning guide for advanced installation -->
