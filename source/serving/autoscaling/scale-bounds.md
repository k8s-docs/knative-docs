# 配置缩放边界

您可以配置上界和下界来控制自动缩放行为。

您还可以指定在创建之后立即将修订扩展到的初始规模。
这可以是所有修订的默认配置，也可以是使用注释的特定修订的默认配置。

## 下界

这个值控制每个修订版本应该拥有的副本的最小数量。
Knative将尝试在任何时间点上都不少于这个数量的副本。

* **全局键:** `min-scale`
* **每修订注释键:** `autoscaling.knative.dev/min-scale`
* **可能值:** integer
* **默认值:** `0` 如果启用了scale-to-zero，并且使用了KPA类，则为1

!!! note
    有关缩零配置的更多信息，请参见[缩零配置](scale-to-zero.md)文档。

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

=== "全局 (ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: config-autoscaler
      namespace: knative-serving
    data:
      min-scale: "3"
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
          min-scale: "3"
    ```

## 上届

这个值控制每个修订版本应该拥有的副本的最大数量。
Knative在任何时间点上运行的副本或正在创建的副本的数量都不会超过这个数目。


如果设置了 `max-scale-limit` 全局键，Knative将确保全局最大刻度和每个新修订版本的最大刻度都不会超过这个值。
当 `max-scale-limit` 设置为正值时，不允许最大刻度高于该值(包括0，这意味着无限)的修订。

* **全局键:** `max-scale`
* **每修订注释键:** `autoscaling.knative.dev/max-scale`
* **可用值:** integer
* **默认值:** `0` 这意味着无限的

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
            autoscaling.knative.dev/max-scale: "3"
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
      max-scale: "3"
      max-scale-limit: "100"
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
          max-scale: "3"
          max-scale-limit: "100"
    ```

## 初始缩放

这个值控制了一个修订在被标记为`Ready`之前必须立即达到的初始目标规模。
在修订一次达到此规模后，该值将被忽略。
这意味着，如果实际接收的流量只需要较小的规模，则在达到初始目标规模后，修订将逐步缩小。

在创建修订时，自动选择初始标度和下限中较大的作为初始目标标度。

* **全局键:** `initial-scale` 结合 `allow-zero-initial-scale`
* **每修订注释键:** `autoscaling.knative.dev/initial-scale`
* **可用值:** integer
* **默认值:** `1`

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
            autoscaling.knative.dev/initial-scale: "0"
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
      initial-scale: "0"
      allow-zero-initial-scale: "true"
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
          initial-scale: "0"
          allow-zero-initial-scale: "true"
    ```

## 扩大最低

此值控制当修订从零扩展时将创建的副本的最小数量。

* **全局键:** n/a
* **每修订注释键:** `autoscaling.knative.dev/activation-scale`
* **可用值:** integer
* **默认值:** `1`


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
            autoscaling.knative.dev/activation-scale: "5"
        spec:
          containers:
            - image: gcr.io/knative-samples/helloworld-go
    ```

## 缩延迟

缩延迟指定在应用缩小决策之前，在并发减少时必须经过的时间窗口。
这可能很有用，例如，在可配置的持续时间内保留容器，以避免在新请求进入时出现冷启动惩罚。
与设置下限不同的是，如果在延迟期间维持较低的并发性，则修订最终将被缩小。

!!! note 
    只支持默认的KPA自动缩放器类。

* **全局键:** `scale-down-delay`
* **每修订注释键:** `autoscaling.knative.dev/scale-down-delay`
* **可用值:** Duration, `0s` <= value <= `1h`
* **默认值:** `0s` (no delay)

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
            autoscaling.knative.dev/scale-down-delay: "15m"
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
      scale-down-delay: "15m"
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
          scale-down-delay: "15m"
    ```
## 稳定窗口

稳定窗口定义滑动时间窗口，当自动缩放器不处于[Panic mode](kpa-specific.md)时，在该时间窗口上对指标进行平均，以提供缩放决策的输入。

* **全局键:** `stable-window`
* **每修订注释键:** `autoscaling.knative.dev/window`
* **可用值:** Duration, `6s` <= value <= `1h`
* **默认值:** `60s`

!!! note
    在缩小期间，在大多数情况下，在稳定窗口的整个持续时间内没有访问修订版的流量后，最后一个副本将被删除。

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

---
