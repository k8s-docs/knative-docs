# 关于 RedisStreamSource

![stage](https://img.shields.io/badge/Stage-alpha-green?style=flat-square)
![version](https://img.shields.io/badge/API_Version-v1alpha1-red?style=flat-square)

RedisStreamSource 从[Redis 流](https://redis.io/docs/data-types/streams/)读取消息，并将它们作为 CloudEvents 发送到引用的接收器，该接收器可以是Kubernetes服务或Knative服务等。
它被配置为重试发送CloudEvents，以便事件不会丢失。

Redis流源可以与本地版本的Redis数据库实例或基于云的实例一起工作，其[`address`][redisstreamsource]将在源规范中指定。
此外，指定的[`stream`][redisstreamsource]名称和消费者[`group`][redisstreamsource]名称将由接收适配器创建，如果它们不存在的话。

消费者组中的消费者数量也可以通过[`config-redis`][config-redis]中的数据进行配置。
这使得每个使用者可以使用到达流中的不同消息。
每个消费者都有一个唯一的消费者名，这是由接收适配器创建的字符串。

当一个Redis流源资源被删除时，组中的所有消费者都被优雅地关闭/删除，然后消费组本身被销毁。
在一个消费者被关闭之前，它的所有挂起的消息都作为CloudEvents发送并被确认。

[redisstreamsource]: ./300-redisstreamsource.yaml
[config-redis]: ./config-redis.yaml

## 开始

### 安装

#### 使用基于云的Redis实例的前提条件:

如果您使用的是本地Redis实例，则可以跳过此步骤。
如果您正在使用Redis的云实例(例如，IBM云上的Redis DB)，则需要在安装事件源之前配置TLS证书。

编辑 [`tls-secret`](./tls-secret.yaml) Secret将您的Redis云实例中的TLS证书添加到`TLS_CERT`数据密钥:

```
vi config/source/tls-secret.yaml
```

将您的证书添加到文件中，并保存该文件。将在下一步应用。

#### 创建 `RedisStreamSource` 源码定义及其所有组件:

在安装事件源之前，还可以使用组中的消费者数量配置接收适配器。

编辑[`config-redis`][config-redis] ConfigMap来编辑`numConsumers`数据键:

```
vi config/source/config-redis.yaml
```

然后,应用 [`config/source`](.)

```sh
ko apply -f config/source
```

### 例子

在这个例子中，你创建了一个Redis Stream事件源，监听添加到“mystream”流中的项目。
然后将这些项作为CloudEvent事件发送到事件显示服务。

1. 安装本地Redis命令如下:

    ```sh
    kubectl apply -f samples/redis
    ```

2. 为这个示例源创建一个名称空间:

    ```sh
    kubectl create ns redex
    ```

3. 通过运行以下命令安装Redis流源示例资源:

    !!! note 
        除了配置您的TLS证书，如果您正在使用Redis DB的云实例，您将需要在[`redisstreamsource`](../../samples/source/redisstreamsource.yaml)源yaml中设置适当的地址。
        对于redis v5及更老版本，不需要指定用户名:

    ```
    address: "rediss://:password@7f41ece8-ccb3-43df-b35b-7716e27b222e.b2b5a92ee2df47d58bad0fa448c15585.databases.appdomain.cloud:32086"
    ```

    对于redis v6，需要一个用户名:
    ```
    address: "rediss://username:password@7f41ece8-ccb3-43df-b35b-7716e27b222e.b2b5a92ee2df47d58bad0fa448c15585.databases.appdomain.cloud:32086"
    ```

    然后，应用[`samples/source`](../../samples/source)创建一个事件显示服务和一个Redis流源资源

    ```sh
    kubectl apply -n redex -f samples/source
    ```

4. 验证Redis流源已经准备好:

    ```sh
    kubectl get  -n redex redisstreamsources.sources.knative.dev mystream
    NAME       SINK                                            AGE   READY   REASON
    mystream   http://event-display.redex.svc.cluster.local/   38s   True
    ```

5. 添加一个项目到`mystream`:

    ```sh
    kubectl exec -n redis svc/redis redis-cli xadd mystream '*' fruit banana color yellow
    ```

6. 检查事件显示接收器，看看是否收到了事件:

    ```sh
    kubectl logs -n redex svc/event-display
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

    数据包含添加到流中的字段值对列表。

7. 要清理，删除Redis流源示例和redex命名空间:

    ```sh
    kubectl delete -f samples/source -n redex
    kubectl delete ns redex
    ```

## 参考

### 先决条件

- Redis安装。(部署本地Redis的说明在上面)

- 了解[Redis Stream的基础知识](https://redis.io/topics/streams-intro), 以及一些[特定于Streams的命令](https://redis.io/commands#stream).

### 源字段

`RedisStreamSource`源是Kubernetes对象。
除了标准的Kubernetes `apiVersion`, `kind`, and `metadata`，它们还有以下的`spec`字段:

| 字段      | 值                                                                                     | 必选或可选 |
| --------- | -------------------------------------------------------------------------------------- | ---------- |
| `address` | Redis TCP地址                                                                          | 必选       |
| `stream`  | Redis流的名称                                                                          | 必选       |
| `group`   | 与此源关联的使用者组的名称。当为空时，将自动为该源创建一个组，并在删除该源时删除该组。 | 可选       |
| `sink`    | 对`可寻址` Kubernetes对象的引用，该对象将解析为用作接收器的uri                         | 必选       |

一旦在集群中创建了对象，源将通过对象上的`status`字段提供关于就绪或错误的输出信息。

### 调试技巧

- 您可以查看Redis流源资源的状态。条件的值，通过运行以下命令之一来诊断任何问题:

```
kubectl get redisstreamsource -n redex
kubectl describe redisstreamsource mystream -n redex
```

- 你也可以阅读日志来检查接收适配器的部署问题:

```
kubectl logs redissource-mystream-1234-0 -n redex
```

- 你也可以阅读日志来检查源控制器的部署问题:

```
kubectl logs redis-controller-manager-0 -n knative-sources
```

- KO安装问题?

参考: https://github.com/google/ko/issues/106

尝试重新安装KO和设置 `export GOROOT=$(go env GOROOT)`

### 配置选项

[`config-observability`](./config-observability.yaml) 和 [`config-logging`](./config-logging.yaml) configmap可用于管理日志记录和度量配置。
