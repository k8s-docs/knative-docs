<!--
This is a generated file and should not be changed manually. All changes should follow the
procedure:

1. Update the information in [`channels.yaml`](channels.yaml).

2. Run the generator tool:
    ```bash
    go run eventing/channels/generator/main.go
    ```
-->

这是Knative事件可用通道的一个不详尽的列表。

!!! note
    列入这个名单并不代表背书，也不意味着任何程度的支持。

| 名字                                                                                                                | 状态   | 维护人员 | 描述                                                                                  |
| ------------------------------------------------------------------------------------------------------------------- | ------ | -------- | ------------------------------------------------------------------------------------- |
| [InMemoryChannel](https://github.com/knative/eventing/tree/{{version}}/config/channels/in-memory-channel/README.md) | Stable | Knative  | 内存通道是一种最好的通道。它们不应该在生产中使用。它们对开发是有用的。                |
| [KafkaChannel](https://github.com/knative-sandbox/eventing-kafka-broker/tree/{{version}}/README.md)                 | Beta   | Knative  | 通道由[Apache Kafka](http://kafka.apache.org/)主题支持。                              |
| [NatssChannel](https://github.com/knative-sandbox/eventing-natss/tree/{{version}}/config/README.md)                 | Alpha  | Knative  | 频道由[NATS流媒体](https://github.com/nats-io/nats-streaming-server#configuring)支持. |


