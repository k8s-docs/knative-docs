# 可用的代理类型

以下代理类型可与Knative事件一起使用。

## 基于多租户通道的代理

Knative事件提供了一个基于多租户(MT)通道的代理实现，该实现使用通道进行事件路由。

在使用MT基于通道的代理之前，必须安装通道实现。

## 可选的代理实现

在Knative事件生态系统中，只要遵守[代理规范](https://github.com/knative/specs/blob/main/specs/eventing/control-plane.md#broker-lifecycle)，其他代理实现都是受欢迎的。

以下是社区或供应商提供的代理列表:

### Apache Kafka代理

更多信息，参见[Apache Kafka Broker](kafka-broker/README.md).

### RabbitMQ 代理

RabbitMQ代理使用[RabbitMQ](https://www.rabbitmq.com/)作为底层实现。
要了解更多信息，请参见[RabbitMQ代理](rabbitmq-broker/README.md)或[GitHub上可用的文档](https://github.com/knative-sandbox/eventing-rabbitmq).
