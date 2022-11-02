# 自动缩放器支持类型

Knative服务支持Knative Pod Autoscaler (KPA)和Kubernetes的 Horizontal Pod Autoscaler (HPA)的实现。
本主题列出每种自动缩放器的特性和限制，以及如何配置它们。

!!! 重要的
    如果您想使用Kubernetes水平Pod自动缩放器(HPA)，您必须在安装Knative服务之后安装它。
关于如何安装HPA，请参见[安装可选服务扩展](../../install/yaml-install/serving/install-serving-with-yaml.md#install-optional-serving-extensions).

## Knative Pod Autoscaler (KPA)

* Knative服务核心的一部分，安装Knative服务后默认启用。
* 支持伸缩到零的功能。
* 不支持基于cpu的自动伸缩。

## Horizontal Pod Autoscaler (HPA)

* 不是Knative Serving核心的一部分，你必须先安装Knative Serving。
* 不支持伸缩到零的功能。
* 支持CPU-based自动定量。

## 配置自动缩放器实现

自动缩放器实现的类型(KPA或HPA)可以通过使用`class`注释来配置。

* **全局设置键:** `pod-autoscaler-class`
* **每个修订注释键:** `autoscaling.knative.dev/class`
* **可能的值:** `"kpa.autoscaling.knative.dev"` 或 `"hpa.autoscaling.knative.dev"`
* **默认:** `"kpa.autoscaling.knative.dev"`

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
            autoscaling.knative.dev/class: "kpa.autoscaling.knative.dev"
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
     pod-autoscaler-class: "kpa.autoscaling.knative.dev"
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
          pod-autoscaler-class: "kpa.autoscaling.knative.dev"
    ```

### 全局和每个修订设置

Knative中的自动伸缩配置可以使用全局设置或每修订设置进行设置。

1. 如果没有指定每修订的自动缩放设置，则将使用全局设置。
1. 如果指定了每修订的设置，当这两种类型的设置都存在时，这些设置将覆盖全局设置。

#### 全局设置

使用 `config-autoscaler` ConfigMap配置自动缩放的全局设置。
如果您使用运营商安装了Knative服务，您可以在 `spec.config.autoscaler` ConfigMap 中设置全局配置设置，位于`KnativeServing`自定义资源(CR)中。

##### 默认自动伸缩 ConfigMap 示例

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
 name: config-autoscaler
 namespace: knative-serving
data:
 container-concurrency-target-default: "100"
 container-concurrency-target-percentage: "0.7"
 enable-scale-to-zero: "true"
 max-scale-up-rate: "1000"
 max-scale-down-rate: "2"
 panic-window-percentage: "10"
 panic-threshold-percentage: "200"
 scale-to-zero-grace-period: "30s"
 scale-to-zero-pod-retention-period: "0s"
 stable-window: "60s"
 target-burst-capacity: "200"
 requests-per-second-target-default: "200"
```

#### 每修订设置

自动伸缩的每修订设置是通过向版本添加 _annotations_ 来配置的。

**Example:**

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
        autoscaling.knative.dev/target: "70"
```

!!! important
    如果您正在使用服务或配置创建修订，则必须在 _revision template_ 中设置注释，以便在创建每个修订时将任何修改应用于它们。
    在单个修订的顶层元数据中设置注释不会将更改传播到其他修订，也不会将更改应用到应用程序的自动伸缩配置中。
