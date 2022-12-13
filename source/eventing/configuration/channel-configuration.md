# 配置通道默认值

Knative事件提供了一个`default-ch-webhook `ConfigMap，其中包含管理默认通道创建的配置设置。

默认的`default-ch-webhook `ConfigMap如下:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: default-ch-webhook
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
    app.kubernetes.io/version: devel
    app.kubernetes.io/part-of: knative-eventing
data:
  default-ch-config: |
    clusterDefault:
      apiVersion: messaging.knative.dev/v1
      kind: InMemoryChannel
    namespaceDefaults:
      some-namespace:
        apiVersion: messaging.knative.dev/v1
        kind: InMemoryChannel
```

通过更改`data.default-ch-config`属性，我们可以定义clusterDefaults和每个命名空间的默认值。

通道自定义资源定义(CRD)使用此配置创建特定于平台的实现。

!!! note
    `clusterDefault`设置决定全局的、集群范围的默认Channel类型。
    您可以使用`namespaceDefaults`设置为各个名称空间配置通道默认值。
