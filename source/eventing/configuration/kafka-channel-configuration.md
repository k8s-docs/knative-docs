# 配置Kafka通道

!!! note
    本指南假设Knative事件处理安装在`knative-eventing`名称空间中。
    如果您在不同的名称空间中安装了Knative事件，请用该名称空间的名称替换`knative-eventing`。

要使用Kafka通道，你必须:

1. 安装KafkaChannel定制资源定义(CRD)。
1. 创建一个ConfigMap，指定如何创建KafkaChannel实例的默认配置。

## 创建一个`kafka-channel`ConfigMap

1. 使用以下模板为`kafka-channel` ConfigMap创建一个YAML文件:

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: kafka-channel
      namespace: knative-eventing
    data:
      channel-template-spec: |
        apiVersion: messaging.knative.dev/v1beta1
        kind: KafkaChannel
        spec:
          numPartitions: 3
          replicationFactor: 1
    ```

    !!! note
        这个例子指定了两个额外的特定于Kafka通道的参数;`numPartitions` and `replicationFactor`。

1. 运行以下命令应用YAML文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```
    其中 `<filename>` 是您在上一步中创建的文件的名称。


1. 可选的。要创建使用Kafka通道的代理，请在代理规范中指定`kafka-channel` ConfigMap。你可以用下面的模板创建一个YAML文件:

    ```yaml
    apiVersion: eventing.knative.dev/v1
    kind: Broker
    metadata:
      annotations:
        eventing.knative.dev/broker.class: MTChannelBasedBroker
      name: kafka-backed-broker
      namespace: default
    spec:
      config:
        apiVersion: v1
        kind: ConfigMap
        name: kafka-channel
        namespace: knative-eventing
    ```

1. 运行以下命令应用YAML文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```
    其中 `<filename>` 是您在上一步中创建的文件的名称。
