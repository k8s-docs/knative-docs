# Knative 服务安装文件

本指南提供了关于核心 Knative 服务 YAML 文件的参考信息，包括:

- 安装 Knative 服务所需的自定义资源定义(CRDs)和核心组件。
- 可用于自定义安装的可选组件。

有关安装这些文件的信息，请参见[使用 YAML 文件安装 Knative 服务](install-serving-with-yaml.md).

下表描述了 Knative 服务中包含的安装文件:

| 文件名                                 | 描述                                                                                                                                         | 依赖              |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| serving-core.yaml                      | 必须:Knative 服务核心组件。                                                                                                                  | serving-crds.yaml |
| serving-crds.yaml                      | 必须:Knative 服务核心 CRDs。                                                                                                                 | none              |
| serving-default-domain.yaml            | 配置 Knative 服务使用[http://sslip.io](http://sslip.io)作为默认的 DNS 后缀。                                                                 | serving-core.yaml |
| serving-hpa.yaml                       | 通过 Kubernetes 水平 Pod 自动缩放器自动缩放 Knative 修订的组件。                                                                             | serving-core.yaml |
| serving-post-install-jobs.yaml         | 安装`serving-core.yaml`后的附加任务。目前它与`serving-storage-version-migration.yaml`相同。                                                  | serving-core.yaml |
| serving-storage-version-migration.yaml | 将 Knative 资源(包括 Service、Route、Revision 和 Configuration)的存储版本从 `v1alpha1` 和`v1beta1`迁移到`v1`。从 0.18 版本升级到 0.19 需要。 | serving-core.yaml |
