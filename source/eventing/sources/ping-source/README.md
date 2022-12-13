# 创建一个PingSource对象

![stage](https://img.shields.io/badge/Stage-stable-green?style=flat-square)
![version](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

介绍如何创建PingSource对象。

PingSource是一种事件源，它在指定的[cron](https://en.wikipedia.org/wiki/Cron)调度上使用固定的有效负载生成事件。

下面的示例展示了如何将PingSource配置为事件源，该事件源每分钟向名为`event-display`的Knative服务发送事件，该服务用作接收器。
如果您有一个现有的接收器，您可以用您自己的值替换示例。

## 在开始之前

创建PingSource:

- 你必须安装[Knative事件](../../../install/yaml-install/eventing/install-eventing-with-yaml.md)。
  在安装Knative事件时，默认启用PingSource事件源类型。
- 您可以使用`kubectl`或[`kn`](../../../client/install-kn.md)命令来创建接收器和PingSource等组件。
- 在此过程的验证步骤中，您可以使用`kubectl` 或[`kail`](https://github.com/boz/kail)进行日志记录。

## 创建PingSource对象

1. 可选: 为你的PingSource创建一个命名空间。

    ```bash
    kubectl create namespace <namespace>
    ```

    其中`<namespace>`是您希望PingSource使用的名称空间。
    例如, `pingsource-example`.

    !!! note
        为PingSource和相关组件创建名称空间可以让您更容易地查看此工作流的更改和事件，因为这些与“默认”名称空间中可能存在的其他组件隔离开来。

        它还使删除源变得更容易，因为您可以删除名称空间来删除所有资源。

2. 创建一个接收器。如果您没有自己的接收器，您可以使用以下示例服务将传入的消息转储到日志中:

    1. 将下面的YAML复制到一个文件中:

        ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: event-display
          namespace: <namespace>
        spec:
          replicas: 1
          selector:
            matchLabels: &labels
              app: event-display
          template:
            metadata:
              labels: *labels
            spec:
              containers:
                - name: event-display
                  image: gcr.io/knative-releases/knative.dev/eventing/cmd/event_display

        ---

        kind: Service
        apiVersion: v1
        metadata:
          name: event-display
          namespace: <namespace>
        spec:
          selector:
            app: event-display
          ports:
          - protocol: TCP
            port: 80
            targetPort: 8080
        ```

        其中`<namespace>`是您在上面第1步中创建的命名空间的名称。

    2. 运行以下命令应用YAML文件:

        ```bash
        kubectl apply -f <filename>.yaml
        ```

        其中`<filename>`是您在上一步中创建的文件的名称。

3. 创建PingSource对象。

    !!! note
        您想要发送的数据必须在PingSource YAML文件中表示为文本。
        发送二进制数据的事件不能在YAML中直接序列化。
        然而，你可以发送base64编码的二进制数据，在PingSource规范中使用 `dataBase64` 来代替 `data`。

    使用以下选项之一:

    === "kn"

        - 创建一个PingSource，发送可以表示为纯文本的数据，如文本、JSON或XML，运行命令:

            ```bash
            kn source ping create <pingsource-name> \
              --namespace <namespace> \
              --schedule "<cron-schedule>" \
              --data '<data>' \
              --sink <sink-name>
            ```
            Where:

            - `<pingsource-name>` 它是您想要创建的PingSource的名称，例如，`test-ping-source`。
            - `<namespace>` 它是您在上面的第1步中创建的命名空间的名称。
            - `<cron-schedule>` 它是PingSource发送事件的时间表的cron表达式，例如，`*/1 * * * *`每分钟发送一个事件。
            - `<data>` 它是您想要发送的数据。此数据必须表示为文本，而不是二进制。例如，一个JSON对象，如`{"message": "Hello world!"}`。
            - `<sink-name>` 它是你的接收器的名字，例如，`http://event-display.pingsource-example.svc.cluster.local`。

            有关可用选项的列表，请参阅[Knative客户端文档](https://github.com/knative/client/blob/main/docs/cmd/kn_source_ping_create.md).

    === "kn: binary data"

        - 创建一个发送二进制数据的PingSource，使用命令:

            ```bash
            kn source ping create <pingsource-name> \
              --namespace <namespace> \
              --schedule "<cron-schedule>" \
              --data '<base64-data>' \
              --encoding 'base64' \
              --sink <sink-name>
            ```
            Where:

            - `<pingsource-name>` 它是您想要创建的PingSource的名称，例如，`test-ping-source`。
            - `<namespace>` 它是您在上面的第1步中创建的命名空间的名称。
            - `<cron-schedule>` 它是PingSource发送事件的时间表的cron表达式，例如，`*/1 * * * *`每分钟发送一个事件。
            -  `<base64-data>` 它是您想要发送的base64编码二进制数据，例如，`ZGF0YQ==`。
            - `<sink-name>` 它是你的接收器的名字，例如，`http://event-display.pingsource-example.svc.cluster.local`。

            有关可用选项的列表，请参阅[Knative客户端文档](https://github.com/knative/client/blob/main/docs/cmd/kn_source_ping_create.md).

    === "YAML"

        - 创建一个PingSource，发送可以表示为纯文本的数据，如文本、JSON或XML:

            1. 使用下面的模板创建一个YAML文件:

                ```yaml
                apiVersion: sources.knative.dev/v1
                kind: PingSource
                metadata:
                  name: <pingsource-name>
                  namespace: <namespace>
                spec:
                  schedule: "<cron-schedule>"
                  contentType: "<content-type>"
                  data: '<data>'
                  sink:
                    ref:
                      apiVersion: v1
                      kind: <sink-kind>
                      name: <sink-name>
                ```
                Where:

                - `<pingsource-name>` 它是您想要创建的PingSource的名称，例如，`test-ping-source`。
                - `<namespace>` 它是您在上面的第1步中创建的命名空间的名称。
                - `<cron-schedule>` 它是PingSource发送事件的时间表的cron表达式，例如，`*/1 * * * *`每分钟发送一个事件。
                - `<content-type>` 它是你想要发送的数据的媒体类型，例如，`application/json`。
                - `<data>` 它是您想要发送的数据。此数据必须表示为文本，而不是二进制。例如，一个JSON对象，如`{"message": "Hello world!"}`。
                - `<sink-kind>` 它是你想用作接收器的任何受支持的可寻址对象，例如`服务`或`部署`。
                - `<sink-name>` 它是你的接收器的名称，例如，`event-display`。

                有关可以为PingSource对象配置的字段的更多信息，请参见[PingSource reference](reference.md)。

            2. 运行以下命令应用YAML文件:

                ```bash
                kubectl apply -f <filename>.yaml
                ```
                其中`<filename>`是您在上一步中创建的文件的名称。

    === "YAML: binary data"

        - 创建一个发送二进制数据的PingSource:

            3. 使用下面的模板创建一个YAML文件:

                ```yaml
                apiVersion: sources.knative.dev/v1
                kind: PingSource
                metadata:
                  name: <pingsource-name>
                  namespace: <namespace>
                spec:
                  schedule: "<cron-schedule>"
                  contentType: "<content-type>"
                  dataBase64: "<base64-data>"
                  sink:
                    ref:
                      apiVersion: v1
                      kind: <sink-kind>
                      name: <sink-name>
                ```
                Where:

                - `<pingsource-name>` 它是您想要创建的PingSource的名称，例如，`test-ping-source-binary`。
                - `<namespace>` 它是您在上面的第1步中创建的命名空间的名称。
                - `<cron-schedule>` 它是PingSource发送事件的时间表的cron表达式，例如，`*/1 * * * *`每分钟发送一个事件。
                - `<content-type>` 它是你想要发送的数据的媒体类型，例如，`application/json`。
                - `<base64-data>` 它是您想要发送的base64编码二进制数据，例如，`ZGF0YQ==`。
                - `<sink-kind>` 它是您希望用作接收器的任何受支持的可寻址对象，例如，Kubernetes服务。
                - `<sink-name>` 它是你的接收器的名称，例如，`event-display`。

                有关可以为PingSource对象配置的字段的更多信息，请参见[PingSource reference](reference.md)。

            4. 运行以下命令应用YAML文件:

                ```bash
                kubectl apply -f <filename>.yaml
                ```
                其中`<filename>`是您在上一步中创建的文件的名称。


## 验证PingSource对象

1. 查看 `event-display` 事件消费者的日志:

    === "kubectl"

        ```bash
        kubectl -n pingsource-example logs -l app=event-display --tail=100
        ```


    === "kail"

        ```bash
        kail -l serving.knative.dev/service=event-display -c user-container --since=10m
        ```

1. 验证输出是否返回PingSource发送到接收器的事件的属性。
   在下例中，该命令返回PingSource发送给event-display服务的事件的“Attributes”和“Data”属性:


    ```{ .bash .no-copy }
    ☁️  cloudevents.Event
    Validation: valid
    Context Attributes,
      specversion: 1.0
      type: dev.knative.sources.ping
      source: /apis/v1/namespaces/pingsource-example/pingsources/test-ping-source
      id: 49f04fe2-7708-453d-ae0a-5fbaca9586a8
      time: 2021-03-25T19:41:00.444508332Z
      datacontenttype: application/json
    Data,
      {
        "message": "Hello world!"
      }
    ```


## 删除PingSource对象

您可以删除PingSource和所有相关资源，也可以单独删除资源:

- 要删除PingSource对象和所有相关资源，可以通过以下命令删除命名空间:

    ```bash
    kubectl delete namespace <namespace>
    ```
    其中`<namespace>`是包含PingSource对象的名称空间。


- 只删除PingSource实例，使用命令:

    === "kn"

        ```bash
        kn source ping delete <pingsource-name>
        ```
        其中`<pingsource-name>`是要删除的PingSource的名称，例如`test-ping-source`。

    === "kubectl"

        ```bash
        kubectl delete pingsources.sources.knative.dev <pingsource-name>
        ```
        其中`<pingsource-name>`是要删除的PingSource的名称，例如`test-ping-source`。

- 只删除sink，使用命令:

    === "kn"

        ```bash
        kn service delete <sink-name>
        ```
        其中`<sink-name>`是你的接收器的名称，例如`event-display`。

    === "kubectl"

        ```bash
        kubectl delete service.serving.knative.dev <sink-name>
        ```
        其中`<sink-name>`是你的接收器的名称，例如`event-display`。
