# 配置并发

并发性决定了应用程序的每个副本在任何给定时间内可以处理的并发请求的数量。
<!-- this is where including files would be useful. We could create a concurrency global config module and insert it here, in the docs for metrics, and in the docs for targets. Showing the correct information each time instead of having it in one place with the per revision config jumbled in with it makes it easier to understand IMHO, and would mean users don't need to visit different pages or hunt for the same information for similar user stories @abrennan89.-->
对于每个版本并发，您必须同时配置`autoscaling.knative.dev/metric`和`autoscaling.knative.dev/target`作为[软限制](#soft-limit)，或`containerConcurrency`作为[硬限制](#hard-limit)。

对于全局并发性，您可以设置`container-concurrency-target-default`值。

## 软与硬并发限制

可以设置 _软_/_硬_ 并发限制。

!!! note
    如果同时指定了软限制和硬限制，则将使用两个值中较小的值。
    这可以防止Autoscaler拥有硬限制值不允许的目标值。

软限额是一个有针对性的限额，而不是一个严格执行的限额。
在某些情况下，特别是在请求突然爆发的情况下，可以超过这个值。

硬性限制是强制的上限。
如果并发性达到硬限制，多余的请求将被缓冲，必须等待，直到有足够的空闲容量来执行请求。

!!! warning
    只有在应用程序中有明确的用例时，才建议使用硬限制配置。
    指定较低的硬限制可能会对应用程序的吞吐量和延迟产生负面影响，并可能导致额外的冷启动。

### 软限制

* **全局键:** `container-concurrency-target-default`
* **每修订注释键:** `autoscaling.knative.dev/target`
* **可用值:** An integer.
* **默认值:** `"100"`

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
            autoscaling.knative.dev/target: "200"
    ```

=== "全局 (ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-autoscaler
     namespace: knative-serving
    data:
     container-concurrency-target-default: "200"
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
          container-concurrency-target-default: "200"
    ```




### 硬限制

硬限制是在每个修订中使用修订规范上的`containerConcurrency`字段指定的。
此设置不是注释。
在自动伸缩`ConfigMap`中没有硬限制的全局设置，因为`containerConcurrency`在自动伸缩之外也有影响，比如对请求的缓冲和排队。
但是，可以在 `config-defaults.yaml` 中为修订版的 `containerConcurrency` 字段设置默认值。


默认值是 `0` ，这意味着不限制允许流入修订版的请求的数量。
大于 `0` 的值指定在任何时候允许流向副本的确切请求数。

* **全局键:** `container-concurrency` (in `config-defaults.yaml`)
* **每修订规范键:** `containerConcurrency`
* **可用值:** integer
* **默认值:** `0`, 意思是没有限制


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
        spec:
          containerConcurrency: 50
    ```

=== "全局 (默认 ConfigMap)"
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: config-defaults
     namespace: knative-serving
    data:
     container-concurrency: "50"
    ```

=== "全局 (Operator)"
    ```yaml
    apiVersion: operator.knative.dev/v1alpha1
    kind: KnativeServing
    metadata:
      name: knative-serving
    spec:
      config:
        defaults:
          container-concurrency: "50"
    ```




## 目标利用率

除了前面解释的文字设置之外，还可以通过使用 _目标利用率值_ 进一步调整并发值。

该值指定Autoscaler实际针对前面指定的目标的百分比。
这也被称为指定副本运行时的 _hotness_，这将导致Autoscaler在达到定义的硬限制之前扩大。

例如，如果`containerConcurrency`设置为10，目标利用率设置为70%(百分之七十)，当所有现有副本的平均并发请求数达到7时，Autoscaler将创建一个新的副本。
编号为7到10的请求仍然会被发送到现有的副本，但这允许在达到`containerConcurrency`限制时启动额外的副本。

* **全局键:** `container-concurrency-target-percentage`
* **每修订注释键:** `autoscaling.knative.dev/target-utilization-percentage`
* **可用值:** float
* **默认值:** `70`

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
            autoscaling.knative.dev/target-utilization-percentage: "80"
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
     container-concurrency-target-percentage: "80"
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
          container-concurrency-target-percentage: "80"
    ```
