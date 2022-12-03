# 开发人员配置选项

## 代理配置示例

下面是一个基于多租户(MT)通道的代理对象的完整示例，其中显示了您可以修改的可能配置选项:

```yaml
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: default
  namespace: default
  annotations:
    eventing.knative.dev/broker.class: MTChannelBasedBroker
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: config-br-default-channel
    namespace: knative-eventing
  delivery:
    deadLetterSink:
      ref:
        kind: Service
        namespace: example-namespace
        name: example-service
        apiVersion: v1
      uri: example-uri
    retry: 5
    backoffPolicy: exponential
    backoffDelay: "2007-03-01T13:00:00Z/P1Y2M10DT2H30M"
```

- 您可以为代理指定任何有效的`name`。使用`default`将创建一个名为`default`的代理。
- `namespace`必须是集群中已存在的命名空间。使用`default`将在`default`命名空间中创建代理。
- 您可以设置`event.knative.dev/broker.class`注释来更改代理的类。
  默认的代理类是`MTChannelBasedBroker`，但 Knative 也支持使用`Kafka`和`RabbitMQBroker`代理类。
  更多信息请参见[Apache Kafka Broker](../brokers/broker-types/kafka-broker/README.md)或[RabbitMQ Broker](../brokers/broker-types/rabbitmq-broker/README.md)文档。
- `spec.config`用于为基于 MT 通道的代理实现指定默认的后备通道配置。
  有关配置默认通道类型的更多信息，请参阅有关[配置代理默认值](../brokers/broker-admin-config-options.md)的文档.
- `spec.delivery`用于配置事件传递选项。
  事件传递选项指定未能传递到事件接收器的事件会发生什么。
  有关更多信息，请参阅关于[事件传递](../event-delivery.md)的文档。
