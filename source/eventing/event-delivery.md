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

- `deadLetterSink`规范包含启用使用死信接收的配置设置。
  这将告诉订阅无法传递给订阅服务器的事件发生了什么。
  配置此参数后，发送失败的事件将被发送到死信接收目的地。
  目的地可以是Knative Service或URI。
  在本例中，目的地是一个名为`example-sink`的`Service` 对象或Knative Service。
- `backoffPolicy`传递参数指定失败后重试尝试事件传递之前的时间延迟。
  `backoffDelay`参数的持续时间使用ISO 8601格式指定。
  例如，`PT1S` 指定1秒延迟。
- `backoffPolicy`传递参数可用于指定重试回退策略。
  该策略可以指定为“线性”或“指数”。
  当使用“线性”后退策略时，后退延迟是重试之间指定的时间间隔。
  当使用'指数'回退策略时，回退延迟等于`backoffDelay*2^<numberOfRetries>`。
- `retry`指定在事件被发送到死信接收之前重试事件传递的次数。

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

- `deadLetterSink`规范包含启用使用死信接收的配置设置。
  这将告诉订阅无法传递给订阅服务器的事件发生了什么。
  配置此参数后，发送失败的事件将被发送到死信接收目的地。
  目的地可以是符合Knative事件接收器契约的任何可寻址对象，例如Knative服务、Kubernetes服务或URI。
  在本例中，目的地是一个名为`example-sink`的`Service`对象或Knative Service。
- `backoffDelay`传递参数指定失败后重试尝试事件传递之前的时间延迟。
  `backoffDelay`参数的持续时间使用ISO 8601格式指定。
  例如，`PT1S`指定1秒延迟。
- `backoffPolicy`传递参数可用于指定重试回退策略。
  该策略可以指定为“线性”或“指数”。
  当使用“线性”后退策略时，后退延迟是重试之间指定的时间间隔。
  这是一个线性增加的延迟，这意味着后退延迟按每次重试的给定间隔增加。
  当使用“指数”后退策略时，后退延迟为每次重试增加给定间隔的乘数。
- `retry`指定在事件被发送到死信接收之前重试事件传递的次数。
  初始的交付尝试不包括在重试计数中，因此交付尝试的总数等于“重试”值+1。

### 代理支持

下表总结了每种代理实现类型支持的交付参数:

| 代理类               | 支持的交付参数                                             |
| -------------------- | ---------------------------------------------------------- |
| googlecloud          | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |
| Kafka                | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |
| MTChannelBasedBroker | 取决于底层的通道                                           |
| RabbitMQBroker       | `deadLetterSink`, `retry`, `backoffPolicy`, `backoffDelay` |

!!! note
    `deadLetterSink` 必须是一个GCP Pub/Sub主题URI。
    `googlecloud` 代理只支持“指数”回退策略。

## 配置通道事件传递

失败的事件可以在转发到`deadLetterSink`之前通过扩展属性进行增强，具体取决于所使用的特定通道实现。
这些扩展属性如下:

- **knativeerrordest**
    - **Type:** String
    - **Description:** 将失败事件发送到的原始目标URL。
        根据遇到失败事件的操作，这可以是一个“交付”或“回复”URL。
    - **Constraints:** 总是存在，因为每个HTTP请求都有一个目标URL。
    - **Examples:**
        - "http://myservice.mynamespace.svc.cluster.local:3000/mypath"
        - ...any `deadLetterSink` URL...

- **knativeerrorcode**
    - **Type:** Int
    - **Description:** 来自最后事件分派尝试的HTTP响应**StatusCode**。
    - **Constraints:** 总是存在，因为每个HTTP响应都包含一个**StatusCode**。
    - **Examples:**
        - "500"
        - ...any HTTP StatusCode...

- **knativeerrordata**
    - **Type:** String
    - **Description:** 来自最后事件分派尝试的HTTP响应**Body**。
    - **Constraints:** 如果HTTP Response **Body**为空，则为空;如果长度过大，则可能被截断。
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
