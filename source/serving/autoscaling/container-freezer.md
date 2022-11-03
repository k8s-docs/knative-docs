# 配置容器冰箱

当启用容器冷冻时，队列代理在其流量降至零或从零扩展时调用端点API。

在社区维护的端点API实现容器-冷冻中，当容器的流量降为零时，运行的进程被冻结，当容器的流量从零上升时，运行的进程被恢复。
但是，用户也可以运行自己的实现(例如，作为计费组件，在处理请求时记录日志)。

## 配置min-scale

要使用容器冷冻，每修订注释键`autoscaling.knative.dev/min-scale`的值必须大于零。

**举例:**
=== "每修订"
    ```yaml
    apiVersion: serving.knative.dev/v1
    kind: Service
    metadata:
      name: helloworld-go
      namespace: default
    spec:
      template:
        metadata:
          annotations:
            autoscaling.knative.dev/min-scale: "3"
        spec:
          containers:
            - image: gcr.io/knative-samples/helloworld-go
    ```


## 配置端点API地址

当启用容器冷冻时，队列代理调用端点API地址，因此需要配置API地址。

1. 运行以下命令打开`config-deployment`ConfigMap:
    ```bash
    kubectl edit configmap config-deployment -n knative-serving
    ```
1. 例如，编辑该文件以配置端点API地址:
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: config-deployment
      namespace: knative-serving
    data:
      concurrency-state-endpoint: "http://$HOST_IP:9696"
    ```

    !!! note
        如果使用社区维护的实现, 使用 `http://$HOST_IP:9696` 作为`concurrency-state-endpoint`的值, 由于社区维护的实现是一个后台进程，相应的值将在运行时由队列代理插入. 
        如果特定于用户的端点API实现部署在集群中的服务中，则使用特定的服务地址，例如 `http://billing.default.svc:9696`.

## 下一步
* 实现您自己的特定于用户的端点API，并将其部署到集群中。
* 使用社区维护的[container-freeze](https://github.com/knative-sandbox/container-freezer)实现。
