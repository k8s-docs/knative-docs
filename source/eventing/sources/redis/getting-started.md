# 创建 RedisStreamSource

![version](https://img.shields.io/badge/API_Version-v1alpha1-red?style=flat-square)

This topic describes how to create a `RedisStreamSource` object.

## 安装RedisStreamSource插件

`RedisStreamSource` 是一个Knative事件附加组件。

1. 通过运行命令安装RedisStreamSource:

    ```bash
    kubectl apply -f {{ artifact(org="knative-sandbox", repo="eventing-redis", file="redis-source.yaml") }}
    ```

1. 验证`redis-controller-manager`正在运行:

    ```bash
    kubectl get deployments.apps -n knative-sources
    ```

    Example output:

    ```{ .bash .no-copy }
    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    redis-controller-manager        1/1     1            1           3s
    ```

{% include "event-display.md" %}

## 创建 RedisStreamSource 对象

1. Create the `RedisStreamSource` object using the YAML template below:

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

* `<redis-stream-source>` is the name of your source. (required)
* `<redis-uri>` is the Redis URI. See the [Redis documentation](https://redis.io/docs/manual/cli/#host-port-password-and-database) for more information. (required)
* `<redis-stream-name>` is the name of the Redis stream. (required)
* `<consumer-group-name>` is the name of the Redis consumer group. When left empty a group
   is automatically created for this source, and deleted when this source is deleted. (optional)
* `<sink>` is where to send events. (required)

1. Apply the YAML file by running the command:

    ```bash
    kubectl apply -f <filename>
    ```

    Where `<filename>` is the name of the file you created in the previous step.

## 验证RedisStreamSource对象

1. View the logs for the `event-display` event consumer by running the command:

    ```bash
    kubectl logs -l app=event-display --tail=100
    ```

    Sample output:

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

## 删除RedisStreamSource对象

* Delete the `RedisStreamSource` object:

    ```bash
    kubectl delete -f <filename>
    ```

## 额外的信息

* 有关Redis流源的更多信息，请参见[`eventing-redis` Github库](https://github.com/knative-sandbox/eventing-redis/tree/main/config/source)
