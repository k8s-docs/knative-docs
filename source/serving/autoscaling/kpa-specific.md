# Knative Pod Autoscaler的额外自动缩放配置

以下设置是针对Knative Pod Autoscaler (KPA)的。

## 模式

KPA对基于时间的窗口中聚合的[指标](`concurrency` or `rps`)起作用。

这些窗口定义Autoscaler考虑的历史数据量，并用于在指定的时间内平滑数据。
这些窗口越短，Autoscaler的反应就越快。

KPA的实现有两种模式:**stable** and **panic**。
每个模式都有单独的聚合窗口:`stable-window` and `panic-window`。

稳定模式用于一般操作，而恐慌模式在默认情况下有一个更短的窗口，如果流量激增，将被用于快速扩展修订。

!!! 注释
    当使用恐慌模式时，版本将不会缩小以避免流失。
    如果在稳定窗口时间内没有快速反应的理由，自动缩放器将离开恐慌模式。

### 稳定窗口

* **全局键:** `stable-window`
* **每修订注释键:** `autoscaling.knative.dev/window`
* **可用值:** Duration, `6s` <= value <= `1h`
* **默认值:** `60s`

!!! note
    当副本归零时，只有在稳定窗口的整个持续时间内没有任何访问修订版本的流量时，才会删除最后一个副本。

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
            autoscaling.knative.dev/window: "40s"
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
     stable-window: "40s"
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
          stable-window: "40s"
    ```




### 恐慌窗口

恐慌窗口被定义为稳定窗口的百分比，确保两者在工作方式上彼此相对。
此值指示在进入恐慌模式时，对历史数据进行评估的窗口将如何收缩。
例如，值 `10.0` 意味着在紧急模式下，窗口将是稳定窗口大小的10%。

* **全局键:** `panic-window-percentage`
* **每修订注释键:** `autoscaling.knative.dev/panic-window-percentage`
* **可用值:** float, `1.0` <= value <= `100.0`
* **默认值:** `10.0`

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
            autoscaling.knative.dev/panic-window-percentage: "20.0"
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
     panic-window-percentage: "20.0"
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
          panic-window-percentage: "20.0"
    ```




### 恐慌的阈值

这个阈值定义Autoscaler何时从稳定模式转入恐慌模式。

该值是当前副本数量所能处理的流量的百分比。

!!! note
    值`100.0`(100%)意味着Autoscaler总是处于紧急模式，因此最小值应该高于`100.0`。

默认设置为`200.0`意味着如果流量是当前副本总体可以处理的两倍，将启动恐慌模式。

* **全局键:** `panic-threshold-percentage`
* **每修订注释键:** `autoscaling.knative.dev/panic-threshold-percentage`
* **可用值:** float, `110.0` <= value <= `1000.0`
* **默认值:** `200.0`

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
            autoscaling.knative.dev/panic-threshold-percentage: "150.0"
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
     panic-threshold-percentage: "150.0"
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
          panic-threshold-percentage: "150.0"
    ```




## 伸缩比率

这些设置通过在单个评估周期中复制集群可以扩大或缩小多少来控制。

在每个方向上总是允许一个副本的最小变化，因此Autoscaler可以随时扩展到+/- 1副本，而不管设置的缩放率如何。

### 扩大率

此设置确定所需与现有Pod的最大比例。
例如，如果值为`2.0`，则修订在一个评估周期中只能从`N`扩展到`2*N`个Pods。

* **全局键:** `max-scale-up-rate`
* **每修订注释键:** n/a
* **可用值:** float
* **默认值:** `1000.0`

**举例:**

=== "全局 (ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-autoscaler
     namespace: knative-serving
    data:
     max-scale-up-rate: "500.0"
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
          max-scale-up-rate: "500.0"
    ```




### 降低率

此设置确定现有Pod与所需Pod的最大比例。
例如，当值为`2.0`时，修订在一个评估周期内只能从`N`扩展到`N/2`个pod。

* **全局键:** `max-scale-down-rate`
* **每修订注释键:** n/a
* **可用值:** float
* **默认值:** `2.0`

**举例:**

=== "全局 (ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-autoscaler
     namespace: knative-serving
    data:
     max-scale-down-rate: "4.0"
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
          max-scale-down-rate: "4.0"
    ```
