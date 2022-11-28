# Knative 事件糖控

Knative 事件糖控将对配置的标签做出反应，以生成或控制集群或命名空间中的事件资源。
这允许集群操作人员和开发人员专注于创建更少的资源，并按需创建底层事件基础设施，并在不再需要时进行清理。

## 安装

糖控默认是`disabled`的，可以通过配置`config-sugar` ConfigMap 启用。
参见下面的简单示例，并配置糖控以获得更多细节。

## 自动创建代理

创建代理的一种方法是使用默认设置手动将资源应用到集群:

1.  将以下 YAML 复制到一个文件中:

    ```yaml
    apiVersion: eventing.knative.dev/v1
    kind: Broker
    metadata:
      name: default
      namespace: default
    ```

1.  运行以下命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

    其中`<filename>``是您在上一步中创建的文件的名称。

在某些情况下，可能需要自动创建代理，例如在名称空间创建时，或者在触发器创建时。
糖控使这些用例成为可能。
下面是 `sugar-config` ConfigMap 的示例配置，为选定的名称空间和所有触发器启用糖控。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
name: config-sugar
namespace: knative-eventing
labels:
  eventing.knative.dev/release: devel
data:
  # Specify a label selector to selectively apply sugaring to certain namespaces
  namespace-selector: |
    matchExpressions:
    - key: "my.custom.injection.key"
      operator: "In"
      values: ["enabled"]
  # Use an empty object to enable for all triggers
  trigger-selector: |
    {}
```

- 当使用标签`my.custom.injection.key: enabled` 创建命名空间时，糖控将在该命名空间中创建一个名为`default`的代理。
- 当创建一个触发器时，糖控将在触发器的名称空间中创建一个名为`default`的代理。

当一个代理被删除，但引用的标签选择器正在使用时，糖控将自动重新创建一个默认代理。

### 名称空间的例子

在创建命名空间时创建一个"default"代理:

1.  将以下 YAML 复制到一个文件中:

    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: example
      labels:
        my.custom.injection.key: enabled
    ```

1.  运行以下命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

    其中 `<filename>` 是您在上一步中创建的文件的名称。

要在命名空间存在后自动创建代理，请将命名空间标记为:

```bash
kubectl label namespace default my.custom.injection.key=enabled
```

如果命名为“default”的代理已经存在于命名空间中，糖控将不做任何事情。

### 触发的例子

当创建一个触发器时，在触发器的命名空间中创建一个"default"代理:

```bash
kubectl apply -f - << EOF
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: hello-sugar
  namespace: hello
spec:
  broker: default
  subscriber:
    ref:
      apiVersion: v1
      kind: Service
      name: event-display
EOF
```

这将在命名空间“hello”中创建一个名为“default”的代理，并尝试向“event-display”服务发送事件。

如果命名为“default”的代理已经存在于命名空间中，糖控将不做任何事情，触发器也不会拥有现有的代理。
