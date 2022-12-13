# 事件实验特征

为了保持Knative的创新性，这个项目的维护者开发了一个[实验特性过程](https://github.com/knative/eventing/blob/main/docs/experimental-features.md)，允许用户交付和测试新的实验特性，而不影响核心项目的稳定性。

<!--TODO: Add note about HOW / where users can provide feedback, otherwise there's not much point mentioning that-->

!!! warning
    实验特性是不稳定的，可能会在您的Knative设置甚至集群设置中导致问题。
    这些特性应该谨慎使用，永远不应该在生产环境中进行测试。
    有关不同开发阶段特性的质量保证的更多信息，请参见[特性阶段定义](https://github.com/knative/eventing/blob/main/docs/experimental-features.md#stage-definition)文档。

本文档解释了如何启用实验特性以及哪些特性现在可用。

## 在开始之前

You must have a Knative cluster running with Knative Eventing installed.

## 实验特性配置

When you install Knative Eventing, the `config-features` ConfigMap is added to
your cluster in the `knative-eventing` namespace.

To enable a feature, you must add it to the `config-features` ConfigMap
under the `data` spec, and set the value for the feature to `enabled`. For
example, to enable a feature called `new-cool-feature`, you would add the
following ConfigMap entry:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-features
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
    knative.dev/config-propagation: original
    knative.dev/config-category: eventing
data:
  new-cool-feature: enabled
```

To disable it, you can either remove the flag or set it to `disabled`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-features
  namespace: knative-eventing
  labels:
    eventing.knative.dev/release: devel
    knative.dev/config-propagation: original
    knative.dev/config-category: eventing
data:
  new-cool-feature: disabled
```

## 可用的实验特性

下表概述了Knative事件处理中可用的实验特性:

| Feature                                                    | Flag                  | Description                                                                                                                                                 | Maturity                   |
| ---------------------------------------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| [DeliverySpec.RetryAfterMax field](delivery-retryafter.md) | `delivery-retryafter` | 在计算重试**429**和**503**响应的回退时间时，指定覆盖HTTP [Retry-After](https://datatracker.ietf.org/doc/html/rfc7231#section-7.1.3)报头的最大重试持续时间。 | Alpha, disabled by default |
| [DeliverySpec.Timeout field](delivery-timeout.md)          | `delivery-timeout`    | 当使用' delivery '规范配置事件传递参数时，可以使用' timeout '字段指定每个发送的HTTP请求的超时时间。                                                         | Alpha, disabled by default |
| [KReference.Group field](kreference-group.md)              | `kreference-group`    | 指定不带API版本的`KReference`资源的API`group`。                                                                                                             | Alpha, disabled by default |
| [Knative参考映射](kreference-mapping.md)                   | `kreference-mapping`  | 提供从[Knative引用](https://github.com/knative/specs/blob/main/specs/eventing/overview.md#destination)到模板化URI的映射。                                   | Alpha, disabled by default |
| [新的触发过滤器](new-trigger-filters.md)                   | `new-trigger-filters` | 启用一个新的触发器`filters`字段，该字段支持一组功能强大的过滤器表达式。                                                                                     | Alpha, disabled by default |
| [严格的订阅用户](strict-subscriber.md)                     | `strict-subscriber`   | 如果未定义字段`spec.subscriber`，则使订阅无效。                                                                                                             | Alpha, disabled by default |
