# 通道类型和默认值

Knative使用两种类型的通道:

* 一个通用Channel对象。
* 每个通道实现都有自己的自定义资源定义(CRDs)，例如InMemoryChannel和KafkaChannel。
  KafkaChannel支持有序的消费者传递保证，这是一个每个分区的阻塞消费者，它等待来自CloudEvent订阅者的成功响应，然后再传递该分区的下一个消息。

每个自定义通道实现都有自己的事件传递机制，比如基于内存或基于代理的机制。
代理的例子包括KafkaBroker和GCP Pub/Sub Broker。

Knative默认提供InMemoryChannel通道实现。
这个默认实现对于不希望配置特定实现类型(如Apache Kafka或NATSS Channels)的开发人员非常有用。

如果希望创建通道而不指定使用哪个通道实现CRD，则可以使用通用通道对象。
如果您不关心特定通道实现提供的属性(比如排序和持久性)，并且希望使用集群管理员选择的实现，那么这将非常有用。


集群管理员可以通过编辑`knative-eventing`命名空间中的`default-ch-webhook` ConfigMap来修改默认通道实现设置。


有关修改ConfigMaps的更多信息，请参见[配置事件操作符自定义资源](../../install/operator/configuring-eventing-cr.md#setting-a-default-channel).

可以为集群、集群上的命名空间或两者配置默认通道。

!!! note
    如果为名称空间配置了默认通道实现，则这将覆盖集群的配置。

在下面的例子中，集群默认通道实现是InMemoryChannel，而 `example-namespace` 的命名空间默认通道实现是KafkaChannel。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: default-ch-webhook
  namespace: knative-eventing
data:
  default-ch-config: |
    clusterDefault:
      apiVersion: messaging.knative.dev/v1
      kind: InMemoryChannel
    namespaceDefaults:
      example-namespace:
        apiVersion: messaging.knative.dev/v1beta1
        kind: KafkaChannel
        spec:
          numPartitions: 2
          replicationFactor: 1
```


!!! note
    InMemoryChannel在生产环境中不能使用通道。


## 下一个步骤


- 创建[InMemoryChannel](create-default-channel.md)
