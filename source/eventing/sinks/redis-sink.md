# Redis Stream Sink

Knative的Redis流事件接收器接收CloudEvents并将它们添加到Redis实例的指定[`stream`][redisstreamsink]。

Redis流接收器可以与本地版本的Redis数据库实例或基于云的实例一起工作，其[`address`][redisstreamsink]将在接收器规范中指定。
此外，指定的[`stream`][redisstreamsink]名称将由接收方创建，如果它们不存在的话。

[redisstreamsink]: ./300-redisstreamsink.yaml

## 开始

### 安装

#### 先决条件

- Knative服务安装说明在[这里](https://knative.dev/docs/install/any-kubernetes-cluster/#installing-the-serving-component) 

- 如果您使用的是本地Redis实例，则可以跳过此步骤。
  如果您正在使用Redis的云实例(例如，IBM cloud上的Redis DB)，则需要在安装事件接收器之前配置TLS证书。

      编辑[`tls-secret`](./tls-secret.yaml) Secret将您的Redis云实例中的TLS证书添加到`TLS_CERT`数据密钥:

      ```sh
      vi config/sink/tls-secret.yaml
      ```

      将您的证书添加到文件中，并保存该文件。将在下一步应用。

#### 创建`RedisStreamSink`接收器定义及其所有组件:

应用 [`config/sink`](.)

```sh
ko apply -f config/sink
```

### 例子

在本例中，您创建了一个Redis流事件接收器，它将接收CloudEvent事件，然后将这些项目添加到`mystream`流中。

1. 通过该命令安装本地Redis。使用实例:

    ```sh
    kubectl apply -f samples/redis
    ```

2. 为这个示例接收器创建一个名称空间:

    ```sh
    kubectl create ns redex
    ```

3. 通过运行这个命令安装一个Redis流接收示例:

    !!! note

        除了配置您的TLS证书，如果您正在使用Redis DB的云实例，您将需要在[`redisstreamsink`](../../samples/sink/redisstreamsink.yaml) 接收器 yaml中设置适当的地址。
        下面是一个连接字符串的例子:

    ```
    address: "rediss://$USERNAME:$PASSWORD@7f41ece8-ccb3-43df-b35b-7716e27b222e.b2b5a92ee2df47d58bad0fa448c15585.databases.appdomain.cloud:32086"
    ```

    然后，应用 [`samples/sink`](../../samples/sink) 创建一个Redis流接收器资源

    ```sh
    kubectl apply -n redex -f samples/sink
    ```

4. 验证Redis流接收器是否准备就绪:

    ```sh
    kubectl get -n redex redisstreamsink.sinks.knative.dev mystream
    NAME       URL                                                     AGE   READY   REASON
    mystream   http://redistreamsinkmystream.redex.svc.cluster.local   35s   True
    ```

5. 向接收器发送一个事件:

    ```sh
    curl $(kubectl get ksvc redistreamsinkmystream -ojsonpath='{.status.url}' -n redex) \
    -H "ce-specversion: 1.0" \
    -H "ce-type: dev.knative.sources.redisstream" \
    -H "ce-source: cli" \
    -H "ce-id: 1" \
    -H "datacontenttype: application/json" \
    -d '["fruit", "orange"]'
    ```

6. 检查redis中是否添加了新消息:

    ```sh
    kubectl exec -n redis svc/redis redis-cli xinfo stream mystream
    ...
    last-entry
    1598652372717-0
    fruit
    orange
    ```

7. 要进行清理，请删除Redis流接收器示例和redex命名空间:

    ```sh
    kubectl delete -f samples/sink -n redex
    kubectl delete ns redex
    ```

## 参考

### 先决条件

- Redis安装。部署本地Redis的说明在上面。

- 了解[Redis流的基础知识](https://redis.io/topics/streams-intro), 以及一些[特定于流的命令](https://redis.io/commands#stream) 

### 源字段

`RedisStreamSink` 资源是Kubernetes对象。
除了标准的Kubernetes `apiVersion`， `kind`和`metadata`，它们还有以下的`spec`字段:

| 字段      | 值            |
| --------- | ------------- |
| `address` | Redis TCP地址 |
| `stream`  | Redis流的名称 |

一旦在集群中创建了对象，接收器将通过对象上的`status`字段提供关于就绪或错误的输出信息。

### 调试技巧

- 你可以检查Redis流接收器资源的`status.condition`值来诊断任何问题，通过运行以下命令:

```
kubectl get redisstreamsinks -n redex
kubectl describe redisstreamsinks mystream -n redex
```

- 您还可以阅读日志以检查接收器部署的问题:

```
kubectl get pods -n redex
kubectl logs {podname} -n redex
```

- 您还可以阅读日志以检查接收器控制器的部署问题:

```
kubectl logs redis-controller-manager-0  -n knative-sinks
```

- KO安装问题?

参考: https://github.com/google/ko/issues/106

尝试重新安装KO和设置 `export GOROOT=$(go env GOROOT)`

### 配置选项

[`config-observability`](./config-observability.yaml) 和 [`config-logging`](./config-logging.yaml) configmap 可用于管理日志记录和度量配置。