# 投递失败处理

您可以为Knative事件组件配置事件传递参数，当事件传递失败时应用这些参数

## 配置订阅事件传递

您可以通过向`Subscription`对象添加`delivery`规范来配置事件如何为每个Subscription传递，如下例所示:

```yaml
apiVersion: messaging.knative.dev/v1
kind: Subscription
metadata:
  name: example-subscription
  namespace: example-namespace
spec:
  delivery:
    deadLetterSink:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: example-sink
    backoffDelay: <duration>
    backoffPolicy: <policy-type>
    retry: <integer>
```

Where

- The `deadLetterSink` spec contains configuration settings to enable using a dead letter sink. This tells the Subscription what happens to events that cannot be delivered to the subscriber. When this is configured, events that fail to be delivered are sent to the dead letter sink destination. The destination can be a Knative Service or a URI. In the example, the destination is a `Service` object, or Knative Service, named `example-sink`.
- The `backoffDelay` delivery parameter specifies the time delay before an event delivery retry is attempted after a failure. The duration of the `backoffDelay` parameter is specified using the ISO 8601 format. For example, `PT1S` specifies a 1 second delay.
- The `backoffPolicy` delivery parameter can be used to specify the retry back off policy. The policy can be specified as either `linear` or `exponential`. When using the `linear` back off policy, the back off delay is the time interval specified between retries. When using the `exponential` back off policy, the back off delay is equal to `backoffDelay*2^<numberOfRetries>`.
- `retry` specifies the number of times that event delivery is retried before the event is sent to the dead letter sink.

## 配置代理事件传递

您可以通过添加一个`delivery`规范来配置事件如何为每个代理交付，如下例所示:

```yaml
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: with-dead-letter-sink
spec:
  delivery:
    deadLetterSink:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: example-sink
    backoffDelay: <duration>
    backoffPolicy: <policy-type>
    retry: <integer>
```

Where

- The `deadLetterSink` spec contains configuration settings to enable using a dead letter sink. This tells the Subscription what happens to events that cannot be delivered to the subscriber. When this is configured, events that fail to be delivered are sent to the dead letter sink destination. The destination can be any Addressable object that conforms to the Knative Eventing sink contract, such as a Knative Service, a Kubernetes Service, or a URI. In the example, the destination is a `Service` object, or Knative Service, named `example-sink`.
- The `backoffDelay` delivery parameter specifies the time delay before an event delivery retry is attempted after a failure. The duration of the `backoffDelay` parameter is specified using the ISO 8601 format. For example, `PT1S` specifies a 1 second delay.
- The `backoffPolicy` delivery parameter can be used to specify the retry back off policy. The policy can be specified as either `linear` or `exponential`. When using the `linear` back off policy, the back off delay is the time interval specified between retries. This is a linearly increasing delay, which means that the back off delay increases by the given interval for each retry. When using the `exponential` back off policy, the back off delay increases by a multiplier of the given interval for each retry.
- `retry` specifies the number of times that event delivery is retried before the event is sent to the dead letter sink. The initial delivery attempt is not included in the retry count, so the total number of delivery attempts is equal to the `retry` value +1.

### 代理支持

下表总结了每种代理实现类型支持的交付参数:

| Broker Class         | Supported Delivery Parameters                              |
| -------------------- | ---------------------------------------------------------- |
| googlecloud          | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |
| Kafka                | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |
| MTChannelBasedBroker | depends on the underlying Channel                          |
| RabbitMQBroker       | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |

!!! note
    `deadLetterSink` must be a GCP Pub/Sub topic URI.
    `googlecloud` Broker only supports the `exponential` back off policy.

## 配置通道事件传递

失败的事件可以在转发到`deadLetterSink`之前通过扩展属性进行增强，具体取决于所使用的特定通道实现。
这些扩展属性如下:

- **knativeerrordest**
    - **Type:** String
    - **Description:** The original destination URL to which the failed event
      was sent.  This could be either a `delivery` or `reply` URL based on
      which operation encountered the failed event.
    - **Constraints:** Always present because every HTTP Request has a
      destination URL.
    - **Examples:**
        - "http://myservice.mynamespace.svc.cluster.local:3000/mypath"
        - ...any `deadLetterSink` URL...

- **knativeerrorcode**
    - **Type:** Int
    - **Description:** The HTTP Response **StatusCode** from the final event
      dispatch attempt.
    - **Constraints:** Always present because every HTTP Response contains
      a **StatusCode**.
    - **Examples:**
        - "500"
        - ...any HTTP StatusCode...

- **knativeerrordata**
    - **Type:** String
    - **Description:** The HTTP Response **Body** from the final event dispatch
      attempt.
    - **Constraints:** Empty if the HTTP Response **Body** is empty,
      and may be truncated if the length is excessive.
    - **Examples:**
        - 'Internal Server Error: Failed to process event.'
        - '{"key": "value"}'
        - ...any HTTP Response Body...

### 通道支持

下表总结了每个通道实现支持的交付参数。

| 通道类型   | 支持的交付参数                                             |
| ---------- | ---------------------------------------------------------- |
| GCP PubSub | none                                                       |
| In-Memory  | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |
| Kafka      | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |
| Natss      | none                                                       |
