# PingSource 源

![API version v1](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

介绍PingSource对象可配置字段的参考信息。

## PingSource

PingSource定义支持以下字段:

| Field                                                    | Description                                                                                                                                                                                                                            | 必须或可选                           |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| [`apiVersion`][kubernetes-overview]                      | 指定API版本，例如 `sources.knative.dev/v1`.                                                                                                                                                                                            | 必须                                 |
| [`kind`][kubernetes-overview]                            | 将此资源对象标识为PingSource对象。                                                                                                                                                                                                     | 必须                                 |
| [`metadata`][kubernetes-overview]                        | 指定唯一标识PingSource对象的元数据。例如，一个`name`。                                                                                                                                                                                 | 必须                                 |
| [`spec`][kubernetes-overview]                            | 指定此PingSource对象的配置信息。                                                                                                                                                                                                       | 必须                                 |
| `spec.contentType`                                       | 媒体类型为`data`或`dataBase64`。默认为空。                                                                                                                                                                                             | 可选                                 |
| `spec.data`                                              | 用作发布到接收器的事件主体的数据。默认为空。与`dataBase64`互斥。                                                                                                                                                                       | 如果没有发送base64编码的数据，则需要 |
| `spec.dataBase64`                                        | 发送到接收器的实际事件主体的base64编码的字符串。默认为空。与`data`相互排斥。                                                                                                                                                           | 如果发送base64编码的数据，则需要     |
| `spec.schedule`                                          | 指定cron计划。默认为 `* * * * *`.                                                                                                                                                                                                      | 可选                                 |
| [`spec.sink`](../../sinks/README.md#sink-as-a-parameter) | 对解析为用作接收器的URI的对象的引用。                                                                                                                                                                                                  | 必须                                 |
| `spec.timezone`                                          | 修改相对于指定时区的实际时间。默认为系统时区。 <br><br> 参见Wikipedia上的[有效tz数据库时区列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。有关时区的一般信息，请参见[IANA](https://www.iana.org/time-zones)网站。 | 可选                                 |
| [`spec.ceOverrides`](#cloudevent-overrides)              | 定义覆盖以控制发送到接收器的事件的输出格式和修改。                                                                                                                                                                                     | 可选                                 |
| `status`                                                 | 定义PingSource的观察状态。                                                                                                                                                                                                             | 可选                                 |
| `status.observedGeneration`                              | 最后由控制器处理的服务的`Generation`。                                                                                                                                                                                                 | 可选                                 |
| `status.conditions`                                      | 资源当前状态的最新可用观察。                                                                                                                                                                                                           | 可选                                 |
| `status.sinkUri`                                         | 为Source配置的当前活动接收器URI。                                                                                                                                                                                                      | 可选                                 |

### CloudEvent Overrides（覆盖）

CloudEvent Overrides定义了覆盖来控制发送到接收器的事件的输出格式和修改。

`ceOverrides` 定义支持以下字段:

| 字段         | 描述                                                                                            | 必须或可选 |
| ------------ | ----------------------------------------------------------------------------------------------- | ---------- |
| `extensions` | 指定在出站事件上添加或覆盖哪些属性。每个`extensions` key-value 对在事件上作为属性扩展独立设置。 | 可选       |

!!! note
    只允许有效的[CloudEvent属性名][cloudevents-attribute-naming]作为扩展。
    您不能从扩展覆盖配置中设置规范定义的属性。
    例如，你不能修改 `type` 属性。

#### 示例: CloudEvent Overrides

```yaml
apiVersion: sources.knative.dev/v1
kind: PingSource
metadata:
  name: test-heartbeats
spec:
  ...
  ceOverrides:
    extensions:
      extra: this is an extra attribute
      additional: 42
```

!!! 合同
    这导致在`subject`上设置`K_CE_OVERRIDES`环境变量，如下所示: <!-- unsure about this -->
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
