# 创建 RabbitMQ Broker

本节介绍如何创建 RabbitMQ Broker。

## 先决条件

1. 您已经安装了 knative 事件
2. 已安装[CertManager v1.5.4](https://github.com/jetstack/cert-manager/releases/tag/v1.5.4) - 与 RabbitMQ 消息拓扑操作符最简单的集成
3. 您已经安装[RabbitMQ 消息传递拓扑操作符](https://github.com/rabbitmq/messaging-topology-operator) - 我们的建议是使用 CertManager 的[最新版本](https://github.com/rabbitmq/messaging-topology-operator/releases/latest)
4. 你可以访问一个正在工作的 RabbitMQ 实例。你可以使用[RabbitMQ 集群 Kubernetes 操作符](https://github.com/rabbitmq/cluster-operator)来创建一个 RabbitMQ 实例。更多信息请参见[RabbitMQ 网站](https://www.rabbitmq.com/kubernetes/operator/using-operator.html).

## 安装 RabbitMQ 控制器

1. 运行命令安装 RabbitMQ 控制器:

   ```bash
   kubectl apply -f {{ artifact(org="knative-sandbox", repo="eventing-rabbitmq", file="rabbitmq-broker.yaml") }}
   ```

1. 验证`rabbitmq-broker-controller` 和 `rabbitmq-broker-webhook`正在运行:

   ```bash
   kubectl get deployments.apps -n knative-eventing
   ```

   示例输出:

   ```{ .bash .no-copy }
   NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
   eventing-controller            1/1     1            1           10s
   eventing-webhook               1/1     1            1           9s
   rabbitmq-broker-controller     1/1     1            1           3s
   rabbitmq-broker-webhook        1/1     1            1           4s
   ```

## 创建一个 RabbitMQBrokerConfig 对象

1.  使用以下模板创建一个 YAML 文件:

    ```yaml
    apiVersion: eventing.knative.dev/v1alpha1
    kind: RabbitmqBrokerConfig
    metadata:
      name: <rabbitmq-broker-config-name>
    spec:
      rabbitmqClusterReference:
        # Configure name if a RabbitMQ Cluster Operator is being used.
        name: <cluster-name>
        # Configure connectionSecret if an external RabbitMQ cluster is being used.
        connectionSecret:
          name: rabbitmq-secret-credentials
      queueType: quorum
    ```

    在哪里:

    - `<rabbitmq-broker-config-name>` 是你想要的 RabbitMQBrokerConfig 对象的名称。
    - `<cluster-name>` 是之前创建的 RabbitMQ 集群的名称。

    !!! note

         你不能同时设置`name`和`connectionSecret`，
         因为`name`是针对与Broker运行在同一集群中的RabbitMQ集群操作实例，
         而`connectionSecret`是针对外部RabbitMQ服务器。

2.  运行以下命令应用 YAML 文件:

    ```bash
    kubectl create -f <filename>
    ```

    其中`<filename>`是您在上一步中创建的文件的名称。

## 创建一个 RabbitMQBroker 对象

1.  使用以下模板创建一个 YAML 文件:

    ```yaml
    apiVersion: eventing.knative.dev/v1
    kind: Broker
    metadata:
      annotations:
        eventing.knative.dev/broker.class: RabbitMQBroker
      name: <broker-name>
    spec:
      config:
        apiVersion: rabbitmq.com/v1beta1
        kind: RabbitmqBrokerConfig
        name: <rabbitmq-broker-config-name>
    ```

    其中`<rabbitmq-broker-config-name>`是你在上面步骤中给你的 RabbitMQBrokerConfig 的名称。

2.  运行以下命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>
    ```

    其中`<filename>`是您在上一步中创建的文件的名称。

## 配置消息排序

默认情况下，触发器每次使用一条消息以保持顺序。
如果事件的顺序并不重要，并且希望获得更高的性能，那么可以通过使用`parallelism`注释来配置。
将`parallelism`设置为`n`为触发器创建`n`个 worker，这些 worker 都将并行使用消息。

下面的 YAML 显示了一个并行度设置为`10`的触发器示例:

```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: high-throughput-trigger
  annotations:
    rabbitmq.eventing.knative.dev/parallelism: "10"
```

## 额外的信息

- 更多示例请访问[`eventing-rabbitmq` Github 库示例目录](https://github.com/knative-sandbox/eventing-rabbitmq/tree/main/samples)
- 要报告一个 bug 或请求一个特性，在 [`eventing-rabbitmq` Github 存储库](https://github.com/knative-sandbox/eventing-rabbitmq)中打开一个问题.
