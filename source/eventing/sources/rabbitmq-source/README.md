# 创建一个RabbitMQSource

![stage](https://img.shields.io/badge/Stage-stable-green?style=flat-square)
![version](https://img.shields.io/badge/API_Version-v1alpha1-red?style=flat-square)

本节介绍如何创建RabbitMQSource。

## 先决条件

1. 您已经安装了[Knative事件](../../../install/yaml-install/eventing/install-eventing-with-yaml.md)
2. 您已经安装了[CertManager v1.5.4](https://github.com/jetstack/cert-manager/releases/tag/v1.5.4)—与RabbitMQ消息拓扑操作器最简单的集成
3. 您已经安装了[RabbitMQ消息拓扑操作器](https://github.com/rabbitmq/messaging-topology-operator)-我们的建议是[最新版本](https://github.com/rabbitmq/messaging-topology-operator/releases/latest)与CertManager
4. 一个正在工作的RabbitMQ实例，我们建议使用[RabbitMQ集群操作符](https://github.com/rabbitmq/cluster-operator)创建一个.有关配置`RabbitmqCluster` CRD的更多信息，请参见[RabbitMQ网站](https://www.rabbitmq.com/kubernetes/operator/using-operator.html)

## 安装RabbitMQ控制器

1. 运行以下命令安装RabbitMQSource控制器:

    ```bash
    kubectl apply -f {{ artifact(org="knative-sandbox", repo="eventing-rabbitmq", file="rabbitmq-source.yaml") }}
    ```

1. 验证`rabbitmq-controller-manager` 和 `rabbitmq-webhook`正在运行:

    ```bash
    kubectl get deployments.apps -n knative-sources
    ```

    Example output:

    ```{ .bash .no-copy }
    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    rabbitmq-controller-manager     1/1     1            1           3s
    rabbitmq-webhook                1/1     1            1           4s
    ```

{% include "event-display.md" %}

## 创建一个RabbitMQSource对象

1. 使用以下模板创建一个YAML文件:

    ```yaml
    apiVersion: sources.knative.dev/v1alpha1
    kind: RabbitmqSource
    metadata:
      name: <source-name>
    spec:
      rabbitmqClusterReference:
        # Configure name if a RabbitMQ Cluster Operator is being used.
        name: <cluster-name>
        # Configure connectionSecret if an external RabbitMQ cluster is being used.
        connectionSecret:
          name: rabbitmq-secret-credentials
      rabbitmqResourcesConfig:
        parallelism: 10
        exchangeName: "eventing-rabbitmq-source"
        queueName: "eventing-rabbitmq-source"
      delivery:
        retry: 5
        backoffPolicy: "linear"
        backoffDelay: "PT1S"
      sink:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: event-display
    ```
    Where:

    - `<source-name>` 是你想要的RabbitMQSource对象的名称。
    - `<cluster-name>` 是你之前创建的RabbitMQ集群的名称。

    !!! note
        您不能同时设置`name` 和 `connectionSecret`，因为`name`是针对与Source运行在同一集群中的RabbitMQ集群操作实例，而`connectionSecret`是针对外部RabbitMQ服务器。

1. 运行以下命令应用YAML文件:

    ```bash
    kubectl apply -f <filename>
    ```
    其中`<filename>` 是您在上一步中创建的文件的名称。

### 验证

检查事件显示服务，看它是否正在接收事件。
Source开始向Sink发送事件可能需要一段时间。

```sh
  kubectl -l='serving.knative.dev/service=event-display' logs -c user-container
  ☁️  cloudevents.Event
  Context Attributes,
    specversion: 1.0
    type: dev.knative.rabbitmq.event
    source: /apis/v1/namespaces/default/rabbitmqsources/<source-name>
    subject: f147099d-c64d-41f7-b8eb-a2e53b228349
    id: f147099d-c64d-41f7-b8eb-a2e53b228349
    time: 2021-12-16T20:11:39.052276498Z
    datacontenttype: application/json
  Data,
    {
      ...
      Random Data
      ...
    }
```

### 清理

1. 删除RabbitMQSource:

    ```sh
    kubectl delete -f <source-yaml-filename>
    ```

1. 删除RabbitMQ的证书秘密:

    ```sh
    kubectl delete -f <secret-yaml-filename>
    ```

1. 删除事件显示服务:

    ```sh
    kubectl delete -f <service-yaml-filename>
    ```

## 额外信息

- 更多示例请访问[`eventing-rabbitmq` Github库示例目录](https://github.com/knative-sandbox/eventing-rabbitmq/tree/main/samples)
- 要报告一个bug或请求一个特性，在[`eventing-rabbitmq` Github库中打开一个问题](https://github.com/knative-sandbox/eventing-rabbitmq).
