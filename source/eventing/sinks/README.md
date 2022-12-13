# 关于接收器

在创建事件源时，可以指定一个 _接收器_，将事件从源发送到该 _接收器_。
接收器是一个 _可寻址_ 或 _可调用的_ 资源，可以从其他资源接收传入的事件。
Knative服务、通道和代理都是接收器的例子。

可寻址对象接收并确认通过HTTP传递到`status.address.url`字段中定义的地址的事件。
作为一个特例，核心[Kubernetes服务对象](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#service-v1-core)也实现了可寻址接口。

可调用对象能够接收通过HTTP传递的事件并转换该事件，在HTTP响应中返回0或1个新事件。
这些返回的事件可以进一步处理，处理方式与处理来自外部事件源的事件相同。

## Sink作为参数

Sink用作对解析为用作接收器的URI的对象的引用。

`接收器`定义支持以下字段:

| 字段                                     | 描述                                                                                              | 必选 or 可选              |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------- |
| `ref`                                    | 这指向一个可寻址的。                                                                              | 必选 如果 _不_ 使用 `uri` |
| `ref.apiVersion`                         | 引用的API版本。                                                                                   | 必选 如果使用`ref`        |
| [`ref.kind`][kubernetes-kinds]           | 引用对象Kind.                                                                                     | 必选 如果使用`ref`        |
| [`ref.namespace`][kubernetes-namespaces] | 引用对象的命名空间。如果省略该参数，默认为保存它的对象。                                          | 可选                      |
| [`ref.name`][kubernetes-names]           | 引用对象名字                                                                                      | 必选 如果使用`ref`        |
| `uri`                                    | 这可以是一个带有非空方案和指向目标或相对URI的非空主机的绝对URL。使用从Ref检索的基URI解析相对URI。 | 必选 如果 _不_ 使用 `ref` |

!!! note
    至少需要`ref` or `uri`中的一个。
    如果两者都指定了，`uri`将从可寻址`ref`结果解析为URL。

### 参数示例

给定下面的YAML，如果`ref`解析为`"http://mysink.default.svc.cluster.local"`，则`uri`被添加到其中，结果为`"http://mysink.default.svc.cluster.local/extra/path"`。

<!-- TODO we should have a page to point to describing the ref+uri destinations and the rules we use to resolve those and reuse the page. -->

```yaml
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: bind-heartbeat
spec:
  ...
  sink:
    ref:
      apiVersion: v1
      kind: Service
      namespace: default
      name: mysink
    uri: /extra/path
```

!!! contract
    这将导致在`subject`上将`K_SINK`环境变量设置为`"http://mysink.default.svc.cluster.local/extra/path"`。

## 使用自定义资源作为接收器

要使用Kubernetes自定义资源(CR)作为事件接收器，您必须:

1. 使CR可寻址。
   您必须确保CR包含`status.address.url`。
   有关更多信息，请参见[可寻址资源](https://github.com/knative/specs/blob/main/specs/eventing/overview.md#addressable)规范。

1. 创建一个可寻址解析器ClusterRole，以获取接收事件所需的RBAC规则。

    例如，你可以创建一个`kafkasinks-addressable-resolver` ClusterRole来允许`get`, `list`, and `watch`访问KafkaSink对象和状态:

    ```yaml
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: kafkasinks-addressable-resolver
      labels:
        kafka.eventing.knative.dev/release: devel
        duck.knative.dev/addressable: "true"
    # Do not use this role directly. These rules will be added to the "addressable-resolver" role.
    rules:
      - apiGroups:
          - eventing.knative.dev
        resources:
          - kafkasinks
          - kafkasinks/status
        verbs:
          - get
          - list
          - watch
    ```

## 通过使用触发器过滤发送到接收器的事件

您可以将触发器连接到接收器，以便在将事件发送到接收器之前对其进行过滤。
连接到触发器的接收器在触发器资源规范中被配置为`subscriber`。

例如:

```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: <trigger-name>
spec:
...
  subscriber:
    ref:
      apiVersion: eventing.knative.dev/v1alpha1
      kind: KafkaSink
      name: <kafka-sink-name>
```

Where;

- `<trigger-name>` 它是连接到接收器的触发器的名称。
- `<kafka-sink-name>` 它是一个KafkaSink对象的名称。

## 使用 kn CLI --sink 标志指定接收器

当您使用Knative (`kn`) CLI创建一个事件生成CR时，您可以使用`--sink`标志指定一个从该资源发送事件的接收器。

下面的例子创建了一个使用服务`http://event-display.svc.cluster.local`作为接收器的SinkBinding:

```bash
kn source binding create bind-heartbeat \
  --namespace sinkbinding-example \
  --subject "Job:batch/v1:app=heartbeat-cron" \
  --sink http://event-display.svc.cluster.local \
  --ce-override "sink=bound"
```

`http://event-display.svc.cluster.local`中的`svc`决定接收器是Knative服务。
其他默认接收器前缀包括通道和代理。

!!! tip
    你可以通过[自定义 `kn`](../../client/configure-kn.md#example-configuration-file)为`kn` CLI命令使用`--sink`标志来配置哪些资源可以被使用.

## 支持的第三方接收器类型

| Name                                                                                       | Maintainer | Description         |
| ------------------------------------------------------------------------------------------ | ---------- | ------------------- |
| [KafkaSink](kafka-sink.md)                                                                 | Knative    | 发送事件到Kafka主题 |
| [RedisStreamSink](https://github.com/knative-sandbox/eventing-redis/tree/main/config/sink) | Knative    | 发送事件到Redis流   |


[kubernetes-kinds]:
  https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
[kubernetes-names]:
  https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names
[kubernetes-namespaces]:
  https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
