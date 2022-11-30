# 创建 RedisStreamSource

![version](https://img.shields.io/badge/API_Version-v1alpha1-red?style=flat-square)

本主题描述如何创建一个`RedisStreamSource`对象。

## 安装 RedisStreamSource 插件

`RedisStreamSource` 是一个 Knative 事件附加组件。

1.  通过运行命令安装 RedisStreamSource:

    ```bash
    kubectl apply -f {{ artifact(org="knative-sandbox", repo="eventing-redis", file="redis-source.yaml") }}
    ```

1.  验证`redis-controller-manager`正在运行:

    ```bash
    kubectl get deployments.apps -n knative-sources
    ```

    示例输出:

    ```{ .bash .no-copy }
    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    redis-controller-manager        1/1     1            1           3s
    ```

{% include "event-display.md" %}

## 创建 RedisStreamSource 对象

1.  使用下面的 YAML 模板创建 `RedisStreamSource` 对象:

    ```yaml
    apiVersion: sources.knative.dev/v1alpha1
    kind: RedisStreamSource
    metadata:
      name: <redis-stream-source>
    spec:
      address: <redis-uri>
      stream: <redis-stream-name>
      group: <consumer-group-name>
      sink: <sink>
    ```

    Where:

    - `<redis-stream-source>` 是你的源名字。(必需)
    - `<redis-uri>` 是 Redis URI. 更多信息请参见[Redis 文档](https://redis.io/docs/manual/cli/#host-port-password-and-database)。(必需)
    - `<redis-stream-name>` 是 Redis 流的名称. (必需)
    - `<consumer-group-name>` 是 Redis 消费群体的名称。当为空时，将自动为该源创建一个组，并在删除该源时删除该组。(可选)
    - `<sink>` 它是发送事件。(必需)

1.  运行以下命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>
    ```

    其中 `<filename>` 是您在上一步中创建的文件的名称。

## 验证 RedisStreamSource 对象

1.  查看 `event-display` 事件消费者的日志:

    ```bash
    kubectl logs -l app=event-display --tail=100
    ```

    样例输出:

    ```{ .bash .no-copy }
    ☁️  cloudevents.Event
    Validation: valid
    Context Attributes,
      specversion: 1.0
      type: dev.knative.sources.redisstream
      source: /mystream
      id: 1597775814718-0
      time: 2020-08-18T18:36:54.719802342Z
      datacontenttype: application/json
    Data,
      [
        "fruit",
        "banana"
        "color",
        "yellow"
      ]
    ```

## 删除 RedisStreamSource 对象

- 删除 `RedisStreamSource` 对象:

  ```bash
  kubectl delete -f <filename>
  ```

## 额外的信息

- 有关 Redis 流源的更多信息，请参见[`eventing-redis` Github 库](https://github.com/knative-sandbox/eventing-redis/tree/main/config/source)
