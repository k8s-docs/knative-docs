# 配置缩零

!!! warning
    只有在使用KnativePodAutoscaler (KPA)时，才能启用缩放到零，并且只能全局配置。有关使用KPA或全局配置的更多信息，请参阅关于[支持的Autoscaler类型](autoscaler-types.md)的文档。

## 缩放到零

缩放到零的值控制Knative是否允许副本缩小到零(如果设置为 `true`)，或如果设置为`false`，在1个副本时停止。

!!! note
    有关每修订缩放边界配置的更多信息，请参阅关于[配置缩放边界](scale-bounds.md)的文档。

* **全局键:** `enable-scale-to-zero`
* **每修订注释键:** No per-revision setting.
* **可能值:** boolean
* **默认的:** `true`

**举例:**

=== "全局 (ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-autoscaler
     namespace: knative-serving
    data:
     enable-scale-to-zero: "false"
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
          enable-scale-to-zero: "false"
    ```




## 缩零宽限期

此设置指定了一个上限时间限制，在删除最后一个副本之前，系统将在内部等待从零开始扩展机制到位。

!!! warning
    这是一个控制允许内部网络编程的时间的值，只有当您遇到在修订缩到零副本时请求被丢弃的问题时，才应该调整该值。

此设置不会调整流量结束后最后一个副本将保留多长时间，也不保证在整个持续时间内实际保留该副本。

* **全局键:** `scale-to-zero-grace-period`
* **每修订注释键:** n/a
* **可能值:** Duration
* **默认值:** `30s`

**举例:**

=== "全局 (ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-autoscaler
     namespace: knative-serving
    data:
     scale-to-zero-grace-period: "40s"
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
          scale-to-zero-grace-period: "40s"
    ```





## 缩零保留期

`scale-to-zero-pod-retention-period`标志决定了在自动缩放器决定将Pod缩到零后，最后一个Pod将保持活动的最小时间。

这与`scale-to-zero-grace-period`标志形成对比，该标志决定了在自动缩放器决定将Pod缩到零后，最后一个Pod保持活动的最长时间。

* **Global key:** `scale-to-zero-pod-retention-period`
* **Per-revision annotation key:** `autoscaling.knative.dev/scale-to-zero-pod-retention-period`
* **Possible values:** 非负时间字符串
* **Default:** `0s`

**Example:**

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
            autoscaling.knative.dev/scale-to-zero-pod-retention-period: "1m5s"
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
     scale-to-zero-pod-retention-period: "42s"
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
          scale-to-zero-pod-retention-period: "42s"
    ```
