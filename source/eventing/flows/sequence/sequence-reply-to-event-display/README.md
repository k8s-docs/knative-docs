# 有应答序列 序列连接到事件显示

我们将创建以下逻辑配置。
我们创建一个PingSource，向[`Sequence`](../README.md)提供事件，然后获取该`Sequence`的输出并显示结果输出。

![合理的配置](sequence-reply-to-event-display.png)

这些示例中使用的函数位于[https://github.com/knative/eventing/blob/main/cmd/appender/main.go](https://github.com/knative/eventing/blob/main/cmd/appender/main.go).

## 先决条件

对于本例，我们假设您已经设置了一个`InMemoryChannel` 以及Knative服务(用于我们的函数)。
示例使用`default`名称空间，同样，如果您希望部署到另一个名称空间，您将需要修改示例以反映这一点。

如果你想使用不同类型的`Channel`，你必须修改`Sequence.Spec.ChannelTemplate`来创建适当的通道资源。

## 设置

### 创建Knative服务

在下面的命令中修改`default`，在你想要创建资源的命名空间中创建步骤:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: first
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-releases/knative.dev/eventing/cmd/appender
          env:
            - name: MESSAGE
              value: " - Handled by 0"

---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: second
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-releases/knative.dev/eventing/cmd/appender
          env:
            - name: MESSAGE
              value: " - Handled by 1"
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: third
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-releases/knative.dev/eventing/cmd/appender
          env:
            - name: MESSAGE
              value: " - Handled by 2"
---

```

```bash
kubectl -n default create -f ./steps.yaml
```

### 创建序列

`sequence.yaml`文件包含了创建序列的规范。
如果使用不同类型的通道，则需要更改`spec.channelTemplate`以指向所需的通道。

```yaml
apiVersion: flows.knative.dev/v1
kind: Sequence
metadata:
  name: sequence
spec:
  channelTemplate:
    apiVersion: messaging.knative.dev/v1
    kind: InMemoryChannel
  steps:
    - ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: first
    - ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: second
    - ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: third
  reply:
    ref:
      kind: Service
      apiVersion: serving.knative.dev/v1
      name: event-display
```

在下面的命令中修改`default`，在你想要创建资源的命名空间中创建`Sequence`:

```bash
kubectl -n default create -f ./sequence.yaml
```

### 创建按顺序显示事件的服务

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: event-display
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-releases/knative.dev/eventing/cmd/event_display
```

在下面的命令中修改 `default` ，在你想要创建资源的命名空间中创建 `Sequence`:

```bash
kubectl -n default create -f ./event-display.yaml
```

### 创建以序列为目标的PingSource

这将创建一个PingSource，它将每2分钟发送一个带有`{“message":"Hello world!"}`作为数据负载的CloudEvent。

```yaml
apiVersion: sources.knative.dev/v1
kind: PingSource
metadata:
  name: ping-source
spec:
  schedule: "*/2 * * * *"
  contentType: "application/json"
  data: '{"message": "Hello world!"}'
  sink:
    ref:
      apiVersion: flows.knative.dev/v1
      kind: Sequence
      name: sequence
```

```bash
kubectl -n default create -f ./ping-source.yaml
```

### 检查结果

现在可以通过检查事件显示Pod的日志看到最终的输出。

```bash
kubectl -n default get pods
```

稍等片刻，然后查看事件显示Pod的日志:

```bash
kubectl -n default logs -l serving.knative.dev/service=event-display -c user-container --tail=-1
☁️  cloudevents.Event
Validation: valid
Context Attributes,
  specversion: 1.0
  type: samples.http.mode3
  source: /apis/v1/namespaces/default/pingsources/ping-source
  id: e8fa7906-ab62-4e61-9c13-a9406e2130a9
  time: 2020-03-02T20:52:00.0004957Z
  datacontenttype: application/json
Extensions,
  knativehistory: sequence-kn-sequence-0-kn-channel.default.svc.cluster.local; sequence-kn-sequence-1-kn-channel.default.svc.cluster.local; sequence-kn-sequence-2-kn-channel.default.svc.cluster.local
  traceparent: 00-6e2947379387f35ddc933b9190af16ad-de3db0bc4e442394-00
Data,
  {
    "id": 0,
    "message": "Hello world! - Handled by 0 - Handled by 1 - Handled by 2"
  }
```

你可以看到初始的PingSource消息`("Hello World!")` 已经被序列中的每一个步骤附加到它上面。
