# 使用集群或命名空间默认值创建通道

开发人员可以通过创建通道对象的实例来创建任何支持的实现类型的通道。

创建一个通道:

1. 使用以下模板为[通道对象](https://knative.dev/docs/reference/api/eventing-api/#messaging.knative.dev/v1.Channel)创建一个YAML文件:

    ```yaml
    apiVersion: messaging.knative.dev/v1
    kind: Channel
    metadata:
      name: <example-channel>
      namespace: <namespace>
    ```
    Where:

    * `<example-channel>` 它是您想要创建的通道的名称。
    * `<namespace>` 它是目标名称空间的名称。

1. 运行以下命令应用YAML文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```
    `<filename>` 是您在上一步中创建的文件的名称。

如果您在`default`命名空间中创建该对象，根据[通道类型和默认值](channel-types-defaults.md)中的默认ConfigMap示例，它是一个InMemoryChannel通道实现。

<!-- TODO: Add tabs for kn etc-->

通道对象创建后，一个突变的准入webhook根据默认通道实现设置`spec.channelTemplate`:

```yaml
apiVersion: messaging.knative.dev/v1
kind: Channel
metadata:
  name: <example-channel>
  namespace: <namespace>
spec:
  channelTemplate:
    apiVersion: messaging.knative.dev/v1
    kind: <channel-template-kind>
```
Where:

* `<example-channel>` 它是您想要创建的通道的名称。
* `<namespace>` 它是目标名称空间的名称。
* `<channel-template-kind>` 它是一种基于默认ConfigMap的通道，例如InMemoryChannel或KafkaChannel。参见[通道类型和默认值](channel-types-defaults.md)中的示例.

!!! note
    `spec.channelTemplate`属性在创建后不能更改，因为它是由默认通道机制而不是用户设置的。


通道控制器基于`spec.channelTemplate`创建一个后备通道实例。

当使用这个机制时，会创建两个对象;一个通用通道对象和一个InMemoryChannel对象。
通过将InMemoryChannel对象的订阅复制到InMemoryChannel对象，并将其状态设置为InMemoryChannel对象的订阅，泛型对象充当InMemoryChannel对象的代理。

!!! note
    默认值只在最初创建通道或序列时由webhook应用。
    如果更改了默认设置，则新的默认设置将只应用于新创建的通道、代理或序列。
    现有资源不会自动更新以使用新配置。
