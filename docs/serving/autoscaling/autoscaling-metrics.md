# 指标

指标配置定义由Autoscaler监视的度量类型。

## 设置每个修订的指标

对于[每修订](autoscaler-types.md#global-versus-per-revision-settings)配置，这是使用`autoscaling.knative.dev/metric`注释确定的。
可以在每个版本中配置的可能的度量类型取决于您正在使用的Autoscaler实现的类型:

* 默认的KPA Autoscaler支持`concurrency`和`rps`指标。
* HPA Autoscaler 支持 `cpu` 指标。

<!-- TODO: Add details about different metrics types, how concurrency and rps differ. Explain cpu. -->

有关KPA和HPA的更多信息，请参阅关于[支持的自动缩放器类型](autoscaler-types.md)的文档。

* **每修订注释键:** `autoscaling.knative.dev/metric`
* **可能值:** `"concurrency"`, `"rps"`, `"cpu"`, `"memory"` 或任何自定义指标名称，这取决于您的Autoscaler类型. `"cpu"`, `"memory"` 和 `"custom"`指标仅在使用HPA类的版本中支持。
* **默认:** `"concurrency"`


=== "每修订并发配置"

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
            autoscaling.knative.dev/metric: "concurrency"
    ```

=== "每修订rps配置"

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
            autoscaling.knative.dev/metric: "rps"
    ```

=== "每修订cpu配置"

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
            autoscaling.knative.dev/class: "hpa.autoscaling.knative.dev"
            autoscaling.knative.dev/metric: "cpu"
    ```

=== "每修订内存配置"

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
            autoscaling.knative.dev/class: "hpa.autoscaling.knative.dev"
            autoscaling.knative.dev/metric: "memory"
    ```

=== "每修订自定义指标配置"

    您可以创建一个HPA，以根据您指定的指标来扩展修订。
    HPA将被配置为在修订的所有Pods中使用度量的**平均值**。

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
            autoscaling.knative.dev/class: "hpa.autoscaling.knative.dev"
            autoscaling.knative.dev/metric: "<metric-name>"
    ```

    Where `<metric-name>` is your custom metric.



## 下一个步骤

* 为应用程序配置[并发目标](concurrency.md)
* 为应用程序的副本配置[每秒请求目标](rps-target.md)
