# 目标

配置目标为Autoscaler提供一个值，它试图为修订的配置指标维护该值。
有关可配置度量类型的更多信息，请参阅[指标](autoscaling-metrics.md)文档。

用于配置每个版本目标的`target`注释是 _metric agnostic_ 。
这意味着目标只是一个整数值，可以应用于任何度量类型。

## 配置目标

* **全局设置键:** `container-concurrency-target-default`. 有关更多信息，请参阅关于[指标](autoscaling-metrics.md)的文档。.
* **每修订注释键:** `autoscaling.knative.dev/target`
* **可能值:** An integer (metric agnostic).
* **默认值:** `"100"` for `container-concurrency-target-default`. 没有为`target`注释设置默认值。


=== "目标注释-每修订"
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
            autoscaling.knative.dev/target: "50"
    ```

=== "并发目标-全局(ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-autoscaler
     namespace: knative-serving
    data:
     container-concurrency-target-default: "200"
    ```

=== "并发目标-全局容器(Operator)"
    ```yaml
    apiVersion: operator.knative.dev/v1alpha1
    kind: KnativeServing
    metadata:
      name: knative-serving
    spec:
      config:
        autoscaler:
          container-concurrency-target-default: "200"
    ```
