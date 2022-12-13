# 创建一个SinkBinding

![API version v1](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

本主题描述如何创建SinkBinding对象。
SinkBinding将接收器解析为URI，在环境变量`K_SINK`中设置URI，并使用`K_SINK`将URI添加到主题中。
如果URI发生变化，SinkBinding会更新`K_SINK`的值。

在下面的示例中，接收器是Knative服务，而主题是CronJob。
如果您有一个现有的主题和接收器，您可以用您自己的值替换示例。

## 在开始之前

在创建SinkBinding对象之前:

- 您必须在集群上安装Knative事件处理。
- 可选:如果你想在SinkBinding中使用`kn`命令，请安装`kn`命令行。

## 可选:选择SinkBinding命名空间选择行为

SinkBinding对象以两种模式之一操作:`exclusion` or `inclusion`。

默认模式为`exclusion`。
在排除模式下，默认情况下为命名空间启用了SinkBinding行为。
为了禁止一个命名空间被评估为突变，你必须使用`bindings.knative.dev/exclude: true`标签来排除它。

在包含模式中，没有为命名空间启用SinkBinding行为。
在对命名空间进行变异评估之前，必须使用标签`bindings.knative.dev/include: true`显式包含它。

设置SinkBinding对象为包含模式。

1. 将`SINK_BINDING_SELECTION_MODE`的值从`exclusion`改为`inclusion`，执行以下命令:

    ```bash
    kubectl -n knative-eventing set env deployments eventing-webhook --containers="eventing-webhook" SINK_BINDING_SELECTION_MODE=inclusion
    ```

2. 要验证`SINK_BINDING_SELECTION_MODE`按需要设置，执行:

    ```bash
    kubectl -n knative-eventing set env deployments eventing-webhook --containers="eventing-webhook" --list | grep SINK_BINDING
    ```

## 创建命名空间

如果没有命名空间，请为SinkBinding对象创建一个命名空间:

```bash
kubectl create namespace <namespace>
```

其中`<namespace>`是您希望SinkBinding使用的名称空间。
例如, `sinkbinding-example`.

!!! note
    如果选择了包含模式，则必须将`bindings.knative.dev/include: true`标签添加到命名空间，以启用SinkBinding行为。

## 创建一个接收器

接收器可以是任何可以接收事件的可寻址Kubernetes对象。

如果没有想要连接到SinkBinding对象的现有接收器，请创建Knative服务。

!!! note
    要创建Knative服务，必须在集群上安装Knative服务。

=== "kn"

    通过运行以下命令创建Knative服务:

    ```bash
    kn service create <app-name> --image <image-url>
    ```
    Where:

    - `<app-name>` 它是应用程序的名称。
    - `<image-url>` 它是图像容器的URL。

    例如:

    ```bash
    $ kn service create event-display --image gcr.io/knative-releases/knative.dev/eventing/cmd/event_display
    ```

=== "YAML"
    1. 使用以下模板为Knative服务创建一个YAML文件:

        ```yaml
        apiVersion: serving.knative.dev/v1
        kind: Service
        metadata:
          name: <app-name>
        spec:
          template:
            spec:
              containers:
                - image: <image-url>
        ```
        Where:

        - `<app-name>` 它是应用程序的名称。例如, `event-display`.
        - `<image-url>` 是图像容器的URL。例如, `gcr.io/knative-releases/knative.dev/eventing/cmd/event_display`.

    1. 运行以下命令应用YAML文件:

        ```bash
        kubectl apply -f <filename>.yaml
        ```
        其中 `<filename>` 是您在上一步中创建的文件的名称。

## 创建一个主题

主题必须是PodSpecable资源。
你可以在你的集群中使用任何PodSpecable资源，例如:

- `Deployment`
- `Job`
- `DaemonSet`
- `StatefulSet`
- `Service.serving.knative.dev`

如果没有想要使用的现有PodSpecable主题，可以使用以下示例创建CronJob对象作为主题。
下面的CronJob创建了一个针对`K_SINK`的云事件，并添加了`CE_OVERRIDES`给出的任何额外覆盖。

1. 为CronJob创建一个YAML文件，示例如下:

    ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: heartbeat-cron
    spec:
      # Run every minute
      schedule: "*/1 * * * *"
      jobTemplate:
        metadata:
          labels:
            app: heartbeat-cron
        spec:
          template:
            spec:
              restartPolicy: Never
              containers:
                - name: single-heartbeat
                  image: gcr.io/knative-nightly/knative.dev/eventing/cmd/heartbeats
                  args:
                  - --period=1
                  env:
                    - name: ONE_SHOT
                      value: "true"
                    - name: POD_NAME
                      valueFrom:
                        fieldRef:
                          fieldPath: metadata.name
                    - name: POD_NAMESPACE
                      valueFrom:
                        fieldRef:
                          fieldPath: metadata.namespace
    ```

1. 运行以下命令应用YAML文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```
    其中' <filename> '是您在上一步中创建的文件的名称。

## 创建一个SinkBinding对象

创建一个`SinkBinding`对象，将事件从主题引导到接收器。

=== "kn"

    通过运行以下命令创建一个`SinkBinding`对象:

    ```bash
    kn source binding create <name> \
      --namespace <namespace> \
      --subject "<subject>" \
      --sink <sink> \
      --ce-override "<cloudevent-overrides>"
    ```
    Where:

    - `<name>` 它是您想要创建的SinkBinding对象的名称。
    - `<namespace>` 它是您为要使用的SinkBinding创建的名称空间。
    - `<subject>` 它是连接的主体。例子:
        - `Job:batch/v1:app=heartbeat-cron` 它匹配命名空间中所有标签为`app=heartbeat-cron`的作业。
        - `Deployment:apps/v1:myapp` 它匹配命名空间中名为`myapp`的部署。
        - `Service:serving.knative.dev/v1:hello` 它匹配名为`hello`的服务。
    - `<sink>` 它是连接的接收器。例如 `http://event-display.svc.cluster.local`.
    - Optional: `<cloudevent-overrides>` 形式是 `key=value`.
      Cloud Event覆盖控制发送到接收器的事件的输出格式和修改，并在发送事件之前应用。
      您可以多次提供此标志。
    

    有关可用选项的列表，请参阅[Knative客户端文档](https://github.com/knative/client/blob/main/docs/cmd/kn_source_binding_create.md#kn-source-binding-create).

    例如:
    ```bash
    $ kn source binding create bind-heartbeat \
      --namespace sinkbinding-example \
      --subject "Job:batch/v1:app=heartbeat-cron" \
      --sink http://event-display.svc.cluster.local \
      --ce-override "sink=bound"
    ```

=== "YAML"
    1. 使用以下模板为`SinkBinding`对象创建一个YAML文件:

        ```yaml
        apiVersion: sources.knative.dev/v1
        kind: SinkBinding
        metadata:
          name: <name>
        spec:
          subject:
            apiVersion: <api-version>
            kind: <kind>
            selector:
              matchLabels:
                <label-key>: <label-value>
          sink:
            ref:
              apiVersion: serving.knative.dev/v1
              kind: Service
              name: <sink>
        ```
        Where:

        - `<name>` 它是您想要创建的SinkBinding对象的名称。例如, `bind-heartbeat`.
        - `<api-version>` 它是该主题的API版本。例如 `batch/v1`.
        - `<kind>` 这是你的主题。例如 `Job`.
        - `<label-key>: <label-value>` 它是键值对的映射，用于选择具有匹配标签的主题。例如, `app: heartbeat-cron` 它会选择任何带有`app: heartbeat-cron`标签的主题.
        - `<sink>` 它是连接的水槽。例如 `event-display`.

        有关可以为SinkBinding对象配置的字段的更多信息，请参见[Sink Binding Reference](reference.md).。

    2. 运行以下命令应用YAML文件:

        ```bash
        kubectl apply -f <filename>.yaml
        ```
        其中`<filename>`是您在上一步中创建的文件的名称。

## 验证SinkBinding对象

1. 通过查看您的接收器的服务日志，验证消息被发送到Knative事件系统:

    ```bash
    kubectl logs -l <sink> -c <container> --since=10m
    ```
    Where:

    - `<sink>` is the name of your sink.
    - `<container>` is the name of the container your sink is running in.

    For example:
    ```bash
    $ kubectl logs -l serving.knative.dev/service=event-display -c user-container --since=10m
    ```

2. 从输出中，观察显示由源发送到显示函数的事件消息的请求头和消息体的行。例如:

    ```{ .bash .no-copy }
      ☁️  cloudevents.Event
      Validation: valid
      Context Attributes,
        specversion: 1.0
        type: dev.knative.eventing.samples.heartbeat
        source: https://knative.dev/eventing-contrib/cmd/heartbeats/#default/heartbeat-cron-1582120020-75qrz
        id: 5f4122be-ac6f-4349-a94f-4bfc6eb3f687
        time: 2020-02-19T13:47:10.41428688Z
        datacontenttype: application/json
      Extensions,
        beats: true
        heart: yes
        the: 42
      Data,
        {
          "id": 1,
          "label": ""
        }
    ```

## 删除一个SinkBinding

要删除命名空间中的SinkBinding对象和所有相关资源，执行以下命令删除命名空间:

```bash
kubectl delete namespace <namespace>
```

其中 `<namespace>` 是包含SinkBinding对象的名称空间的名称。