# 订阅

在创建通道和接收器之后，可以创建订阅以启用事件传递。

订阅由一个订阅对象组成，该对象指定要向其传递事件的通道和接收器(也称为订阅服务器)。
您还可以指定一些特定于接收器的选项，例如如何处理失败。

有关订阅对象的详细信息，请参见[订阅](https://knative.dev/docs/reference/api/eventing-api/#messaging.knative.dev/v1.Subscription).

## 创建订阅


=== "kn"
    通过运行以下命令在通道和接收器之间创建订阅:

    ```bash
    kn subscription create <subscription-name> \
      --channel <Group:Version:Kind>:<channel-name> \
      --sink <sink-prefix>:<sink-name> \
      --sink-reply <sink-prefix>:<sink-name> \
      --sink-dead-letter <sink-prefix>:<sink-name>
    ```

    - `--channel` 指定应该处理的云事件的源。
      您必须提供通道名称。
      如果您不使用由通道资源支持的默认通道，则必须为指定的通道类型在通道名称前面加上`<Group:Version:Kind>`。
      例如，对于kafka支持的通道，这是`messaging.knative.dev:v1beta1:KafkaChannel`。
    

    - `--sink` 指定应将事件传递到的目标目的地。
      默认情况下，`<sink-name>`被解释为该名称的Knative服务，位于与订阅相同的名称空间中。
      您可以通过使用以下前缀之一指定接收器的类型:

        - `ksvc`: Knative服务。
        - `svc`: Kubernetes服务。
        - `channel`: 应该用作目的地的通道。此处只能引用默认的通道类型。
        - `broker`: 事件代理。
        - `--sink-reply` 它是一个可选参数，可用于指定发送接收器应答的位置。
          它使用与`--sink`标志相同的命名约定来指定接收器。
        - `--sink-dead-letter` 它是一个可选参数，您可以使用它指定向何处发送
          CloudEvent，以防出现故障。
          它使用相同的命名约定将接收器指定为`--sink`标志。

            - `ksvc`: Knative服务。
            - `svc`: Kubernetes服务。
            - `channel`: 应该用作目的地的通道。这里只能引用默认通道类型。
            - `broker`: 事件代理
        - `--sink-reply` 和 `--sink-dead-letter` 是可选参数. 
          它们可以分别用于指定发送Sink回复的位置，以及在发生故障时发送CloudEvent的位置。
          两者都使用相同的命名约定将Sink指定为`--sink`标志。


    这个示例命令创建了一个名为`mysubscription`的订阅，它将事件从名为`mychannel`的通道路由到名为`myservice`的Knative服务。


    !!! note
        Sink前缀是可选的。你也可以将`--sink`的服务指定为`--sink <service-name>`，并省略`ksvc`前缀。


=== "YAML"
    1. 使用以下示例为订阅对象创建一个YAML文件:

        ```yaml
        apiVersion: messaging.knative.dev/v1
        kind: Subscription
        metadata:
          name: <subscription-name>
          # Name of the Subscription.
          namespace: default
        spec:
          channel:
            apiVersion: messaging.knative.dev/v1
            kind: Channel
            name: <channel-name>
            # Name of the Channel that the Subscription connects to.
          delivery:
            # Optional delivery configuration settings for events.
            deadLetterSink:
            # When this is configured, events that failed to be consumed are sent to the deadLetterSink.
            # The event is dropped, no re-delivery of the event is attempted, and an error is logged in the system.
            # The deadLetterSink value must be a Destination.
              ref:
                apiVersion: serving.knative.dev/v1
                kind: Service
                name: <service-name>
          reply:
            # Optional configuration settings for the reply event.
            # This is the event Sink that events replied from the subscriber are delivered to.
            ref:
              apiVersion: messaging.knative.dev/v1
              kind: InMemoryChannel
              name: <service-name>
          subscriber:
            # Required configuration settings for the Subscriber. This is the event Sink that events are delivered to from the Channel.
            ref:
              apiVersion: serving.knative.dev/v1
              kind: Service
              name: <service-name>
        ```

    2. 运行以下命令应用YAML文件:

        ```bash
        kubectl apply -f <filename>.yaml
        ```
        其中 `<filename>` 是您在上一步中创建的文件的名称。


## 列出订阅

您可以使用`kn` CLI工具列出所有现有的订阅。

- 列出所有订阅:

    ```bash
    kn subscription list
    ```

- YAML格式的订阅列表:

    ```bash
    kn subscription list -o yaml
    ```

## 描述订阅

您可以使用`kn`CLI工具打印订阅的详细信息:

```bash
kn subscription describe <subscription-name>
```
<!--TODO: Add an example command and output-->
<!--TODO: Add details for kn Subscription update - existing generated docs weren't clear enough, need better explained examples-->

## 删除订阅

您可以使用`kn` or `kubectl`CLI工具删除订阅。

=== "kn"
    ```bash
    kn subscription delete <subscription-name>
    ```


=== "kubectl"
    ```bash
    kubectl subscription delete <subscription-name>
    ```

## 下一部

- [使用集群或命名空间默认值创建通道](create-default-channel.md)
