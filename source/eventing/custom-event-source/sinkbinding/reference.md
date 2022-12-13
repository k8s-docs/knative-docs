# SinkBinding 参考

![API version v1](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

本主题提供关于SinkBinding对象可配置参数的参考信息。

## 支持参数

`SinkBinding`资源支持以下参数:

| 字段                                                     | 描述                                                      | 必须 或 可选 |
| -------------------------------------------------------- | --------------------------------------------------------- | ------------ |
| [`apiVersion`][kubernetes-overview]                      | 指定API版本，例如 `sources.knative.dev/v1`.               | 必须         |
| [`kind`][kubernetes-overview]                            | 将该资源对象标识为`SinkBinding` 对象。                    | 必须         |
| [`metadata`][kubernetes-overview]                        | 指定唯一标识`SinkBinding`对象的元数据。例如，一个`name`。 | 必须         |
| [`spec`][kubernetes-overview]                            | 指定此`SinkBinding`对象的配置信息。                       | 必须         |
| [`spec.sink`](../../sinks/README.md#sink-as-a-parameter) | 对解析为用作接收器的URI的对象的引用。                     | 必须         |
| [`spec.subject`](#subject-parameter)                     | 对“运行时契约”通过绑定实现增强的资源的引用。              | 必须         |
| [`spec.ceOverrides`](#cloudevent-overrides)              | 定义覆盖以控制发送到接收器的事件的输出格式和修改。        | 可选         |

### 主题参数

Subject参数引用“运行时契约”通过绑定实现增强的资源。

`subject`定义支持以下字段:

| 字段                                 | 描述                                                                                                                                                                                            | 必须 or 可选                                    |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| `apiVersion`                         | 引用的API版本。                                                                                                                                                                                 | 必须                                            |
| [`kind`][kubernetes-kinds]           | Kind of the referent.                                                                                                                                                                           | 必须                                            |
| [`namespace`][kubernetes-namespaces] | 引用对象的命名空间。如果省略，默认为保存它的对象。                                                                                                                                              | 可选                                            |
| [`name`][kubernetes-names]           | 推荐人的名字。                                                                                                                                                                                  | 如果你配置了`selector`，请不要使用。            |
| `selector`                           | 引用对象的选择器。                                                                                                                                                                              | 如果配置了`name`，请不要使用。                  |
| `selector.matchExpressions`          | 标签选择器要求的列表。要求是 ANDed。                                                                                                                                                            | 使用`matchExpressions` or `matchLabels`中的一个 |
| `selector.matchExpressions.key`      | 选择器应用的标签键。                                                                                                                                                                            | 如果使用 `matchExpressions` 必须                |
| `selector.matchExpressions.operator` | 表示键与一组值的关系。有效的操作符是`In`, `NotIn`, `Exists` and `DoesNotExist`。                                                                                                                | 如果使用 `matchExpressions` 必须                |
| `selector.matchExpressions.values`   | 字符串值的数组。如果`operator` 为`In` or `NotIn`，值数组必须非空。如果`operator` 是`Exists` or `DoesNotExist`，值数组必须为空。在策略合并补丁期间替换此数组。                                   | 如果使用 `matchExpressions`     必须            |
| `selector.matchLabels`               | 键值对的映射。`matchLabels`映射中的每个键-值对相当于`matchExpressions`的一个元素，其中键字段是`matchLabels.<key>`， `operator`是`In`， `values` 数组只包含"matchLabels.<value>"。要求是 ANDed。 | 使用`matchExpressions` or `matchLabels`中的一个 |

#### 主题参数示例

给定下面的YAML，在`default`命名空间中命名为`mysubject`的`Deployment`被选中:

```yaml
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: bind-heartbeat
spec:
  subject:
    apiVersion: apps/v1
    kind: Deployment
    namespace: default
    name: mysubject
  ...
```

给定下面的YAML，在`default`命名空间中选择任何带有`working=example`标签的`Job`:

```yaml
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: bind-heartbeat
spec:
  subject:
    apiVersion: batch/v1
    kind: Job
    namespace: default
    selector:
      matchLabels:
        working: example
  ...
```

给定以下YAML，在`default`命名空间中选择任何带有`working=example` or `working=sample`标签的`Pod`:

```yaml
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: bind-heartbeat
spec:
  subject:
    apiVersion: v1
    kind: Pod
    namespace: default
    selector:
      - matchExpression:
        key: working
        operator: In
        values:
          - example
          - sample
  ...
```

### CloudEvent Overrides (覆盖)

CloudEvent Overrides定义了覆盖来控制发送到接收器的事件的输出格式和修改。

`ceOverrides`定义支持以下字段:

| Field        | Description                                                                                                                                                             | 必须 or 可选 |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| `extensions` | Specifies which attributes are added or overridden on the outbound event. Each `extensions` key-value pair is set independently on the event as an attribute extension. | 可选         |

!!! note
    Only valid [CloudEvent attribute names][cloudevents-attribute-naming] are
    allowed as extensions. You cannot set the spec defined attributes from
    the extensions override configuration. For example, you can not modify the
    `type` attribute.

#### CloudEvent Overrides 举例

```yaml
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: bind-heartbeat
spec:
  ...
  ceOverrides:
    extensions:
      extra: this is an extra attribute
      additional: 42
```

!!! contract
    This results in the `K_CE_OVERRIDES` environment variable being set on the
    `subject` as follows:
    ```{ .json .no-copy }
    { "extensions": { "extra": "this is an extra attribute", "additional": "42" } }
    ```

[kubernetes-overview]:
  https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/#required-fields
[kubernetes-kinds]:
  https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
[kubernetes-names]:
  https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names
[kubernetes-namespaces]:
  https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[cloudevents-attribute-naming]:
  https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md#attribute-naming-convention
