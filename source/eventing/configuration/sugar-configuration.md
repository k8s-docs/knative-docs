# 配置糖控制器

介绍糖控制器的配置方法。
您可以配置糖控制器，以便在使用配置的标签创建名称空间或触发器时创建代理。
参见[Knative 事件糖控制器](../sugar/README.md)的示例。

默认的`config-sugar` ConfigMap 通过将`namespace-selector`和`trigger-selector`设置为空字符串来禁用糖控制器。

启用糖控制器

- 对于命名空间，可以配置 LabelSelector `namespace-selector`。
- 对于触发器，可以配置 LabelSelector `trigger-selector`。

在选定的名称空间和触发器上启用糖控制器的示例配置

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
name: config-sugar
namespace: knative-eventing
labels:
  eventing.knative.dev/release: devel
data:
  namespace-selector: |
    matchExpressions:
    - key: "eventing.knative.dev/injection"
      operator: "In"
      values: ["enabled"]

  trigger-selector: |
    matchExpressions:
    - key: "eventing.knative.dev/injection"
      operator: "In"
      values: ["enabled"]
```

糖控制器只会在带有`eventing.knative.dev/injection: enabled`标签的命名空间或触发器上操作。
这也模拟了命名空间的遗留糖控制器行为。

你可以通过下面的命令编辑这个 ConfigMap:

```bash
kubectl edit cm config-sugar -n knative-eventing
```
