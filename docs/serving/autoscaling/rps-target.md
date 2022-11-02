# 配置每秒请求数(RPS)目标

此设置为应用程序的每个副本指定每秒请求数的目标。
您的修订还必须配置为使用 `rps` [指标注释](autoscaling-metrics.md)。

* **全局值:** `requests-per-second-target-default`
* **每修订注释值:** `autoscaling.knative.dev/target`
* **可用值:** An integer.
* **默认值:** `"200"`

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
            autoscaling.knative.dev/target: "150"
            autoscaling.knative.dev/metric: "rps"
        spec:
          containers:
            - image: gcr.io/knative-samples/helloworld-go
    ```

=== "全局 (ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-autoscaler
     namespace: knative-serving
    data:
     requests-per-second-target-default: "150"
    ```

=== "全局 (Operator)"
    ```yaml
    apiVersion: operator.knative.dev/v1alpha1
    kind: KnativeServing
    metadata:
      name: knative-serving
    spec:
      config:
        autoscaler:
          requests-per-second-target-default: "150"
    ```
