# 使用触发器

触发器表示从特定代理订阅事件的愿望。

`subscriber` 值必须是[type Destination](https://pkg.go.dev/knative.dev/pkg/apis/duck/v1#Destination).

## 触发器举例

下面的触发器从 `default` 代理接收所有事件，并将它们传递给 Knative 服务 `my-service`:

1.  使用以下示例创建一个 YAML 文件:

    ```yaml
    apiVersion: eventing.knative.dev/v1
    kind: Trigger
    metadata:
      name: my-service-trigger
    spec:
      broker: default
      subscriber:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: my-service
    ```

1.  通过运行该命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

    其中 `<filename>` 是您在上一步中创建的文件的名称。

以下触发器从`default`代理接收所有事件，并将它们交付到 Kubernetes 服务`my-service`的自定义路径`/my-custom-path`:

1.  使用以下示例创建一个 YAML 文件:

    ```yaml
    apiVersion: eventing.knative.dev/v1
    kind: Trigger
    metadata:
      name: my-service-trigger
    spec:
      broker: default
      subscriber:
        ref:
          apiVersion: v1
          kind: Service
          name: my-service
        uri: /my-custom-path
    ```

1.  通过运行该命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

    其中' <filename> '是您在上一步中创建的文件的名称。

## 触发过滤

支持对任意数量的 CloudEvents 属性和扩展进行精确匹配筛选。
如果过滤器设置了多个属性，那么一个事件必须具有触发器筛选它所需的所有属性。
注意，我们只支持字符串值的精确匹配。

### 举例

此示例过滤来自`default`代理的事件，这些事件类型为`dev.knative.foo.bar`，扩展名为`myextension`，值为`my-extension-value`。

1.  使用以下示例创建一个 YAML 文件:

    ```yaml
    apiVersion: eventing.knative.dev/v1
    kind: Trigger
    metadata:
      name: my-service-trigger
    spec:
      broker: default
      filter:
        attributes:
          type: dev.knative.foo.bar
          myextension: my-extension-value
      subscriber:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: my-service
    ```

1.  通过运行该命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

    其中`<filename>`是您在上一步中创建的文件的名称。

## 触发器注释

你可以通过设置以下两个注释来修改触发器的行为:

- `eventing.knative.dev/injection`: 如果设置为`enabled`，事件将自动为触发器创建一个不存在的代理。
  代理在创建触发器的名称空间中创建。
  此注释仅在启用[糖控制器](../sugar/README.md)时有效，这是可选的，默认情况下不启用。
- `knative.dev/dependency`: 此注释用于标记触发器所依赖的源。
  如果其中一个依赖项没有准备好，那么触发器也不会准备好。

下面的 YAML 是一个带有依赖的触发器的例子:

```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: my-service-trigger
  annotations:
    knative.dev/dependency: '{"kind":"PingSource","name":"test-ping-source","apiVersion":"sources.knative.dev/v1"}'
spec:
  broker: default
  filter:
    attributes:
      type: dev.knative.foo.bar
      myextension: my-extension-value
    subscriber:
      ref:
        apiVersion: serving.knative.dev/v1
        kind: Service
        name: my-service
```
