# ContainerSource 参考

![API version v1](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

本主题提供有关ContainerSource对象的可配置字段的参考信息。

## ContainerSource

ContainerSource定义支持以下字段:

| Field                                                    | Description                                                               | 必须 or 可选 |
| -------------------------------------------------------- | ------------------------------------------------------------------------- | ------------ |
| [`apiVersion`][kubernetes-overview]                      | 指定API版本，例如 `sources.knative.dev/v1`.                               | 必须         |
| [`kind`][kubernetes-overview]                            | 将此资源对象标识为ContainerSource对象。                                   | 必须         |
| [`metadata`][kubernetes-overview]                        | 指定唯一标识ContainerSource对象的元数据。例如, a `name`.                  | 必须         |
| [`spec`][kubernetes-overview]                            | 指定此ContainerSource对象的配置信息。                                     | 必须         |
| [`spec.sink`](../../sinks/README.md#sink-as-a-parameter) | 对解析为用作接收器的URI的对象的引用。                                     | 必须         |
| [`spec.template`](#template-parameter)                   | 一个形状为`Deployment.spec.template`的`template`，用于此ContainerSource。 | 必须         |
| [`spec.ceOverrides`](#cloudevent-overrides)              | 定义覆盖以控制发送到接收器的事件的输出格式和修改。                        | 可选         |


### 模板参数

This is a `template` in the shape of `Deployment.spec.template` to use for the ContainerSource.
For more information, see the [Kubernetes Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

<!-- not sure what the required and 可选 fields are for this. -->

#### 示例:模板参数

```yaml
apiVersion: sources.knative.dev/v1
kind: ContainerSource
metadata:
  name: test-heartbeats
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-nightly/knative.dev/eventing/cmd/heartbeats
          name: heartbeats
          args:
            - --period=1
          env:
            - name: POD_NAME
              value: "mypod"
            - name: POD_NAMESPACE
              value: "event-test"
  ...
```


### CloudEvent Overrides

CloudEvent Overrides定义了覆盖来控制发送到接收器的事件的输出格式和修改。

A `ceOverrides` definition supports the following fields:

| Field        | Description                                                                                                                                                             | 必须 or 可选 |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| `extensions` | Specifies which attributes are added or overridden on the outbound event. Each `extensions` key-value pair is set independently on the event as an attribute extension. | 可选         |

!!! note
    Only valid [CloudEvent attribute names][cloudevents-attribute-naming]
    are allowed as extensions. You cannot set the spec defined attributes from
    the extensions override configuration. For example, you can not modify the
    `type` attribute.

#### 举例: CloudEvent Overrides

```yaml
apiVersion: sources.knative.dev/v1
kind: ContainerSource
metadata:
  name: test-heartbeats
spec:
  ...
  ceOverrides:
    extensions:
      extra: this is an extra attribute
      additional: 42
```

!!! contract
    This results in the `K_CE_OVERRIDES` environment variable being set on the
    `subject` as follows: <!-- unsure about this -->
    ```{ .json .no-copy }
    { "extensions": { "extra": "this is an extra attribute", "additional": "42" } }
    ```

[kubernetes-overview]:
  https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#required-fields
[cloudevents-attribute-naming]:
  https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md#attribute-naming-convention
