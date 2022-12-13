# 日志

您可以使用日志处理器和转发器[Fluent Bit](https://docs.fluentbit.io/)在中心目录中收集Kubernetes日志。
运行Knative时不需要这样做，但是使用[Knative服务](/docs/serving/)会很有帮助，它会在不再需要Pod和相关日志时自动删除它们。

Fluent Bit支持导出到许多其他日志提供程序。
如果您已经有一个现有的日志提供程序，例如Splunk、datdog、ElasticSearch或Stackdriver，您可以按照[FluentBit文档](https://docs.fluentbit.io/manual/pipeline/outputs)配置日志转发器。

## 设置日志记录组件

设置日志收集需要两个步骤:

1. 在各节点上运行日志转发DaemonSet。
2. 在集群的某个地方运行收集器。

!!! tip
    在下面的示例中，使用的是 StatefulSet，它将日志存储在Kubernetes PersistentVolumeClaim上，但您也可以使用HostPath。

### 设置收集器

`fluent-bit-collector.yaml` 文件定义了一个StatefulSet，以及一个Kubernetes服务，该服务允许从集群内部访问和读取日志。
提供的配置将在名为`logging`的命名空间中创建监视配置。

!!! important
    在转发之前设立收货人。在配置转发器时，您将需要收集器的地址，并且转发器可能会将日志排队，直到收集器准备好为止。

![系统框图:转发器和共定位收集器和nginx](system.svg)

<!-- yuml.me UML rendering of:
[Forwarder1]logs->[Collector]
[Forwarder2]logs->[Collector]

// Add notes
[Collector]->[shared volume]
[nginx]-[shared volume]
-->

#### 过程

1. 输入命令应用配置:

    ```bash
    kubectl apply -f https://github.com/knative/docs/raw/main/docs/serving/observability/logging/fluent-bit-collector.yaml
    ```
    默认配置将日志分为:

    - Knative服务，或带有`app=Knative`标签的Pod。
    - Non-Knative apps.

    !!! note
        日志默认使用Pod名称进行日志记录;这可以通过在安装之前或之后更新`log-collector-config`  ConfigMap来更改。

    !!! warning
        ConfigMap更新后，必须重新启动Fluent Bit。您可以通过删除Pod并让StatefulSet重新创建它来实现这一点。

1. 要通过web浏览器访问日志，输入以下命令:

    ```bash
    kubectl port-forward --namespace logging service/log-collector 8080:80
    ```

3. 导航到 `http://localhost:8080/`.

4. 可选:您可以在`nginx` Pod中打开一个shell，并使用Unix工具搜索日志，输入以下命令:

    ```bash
    kubectl exec --namespace logging --stdin --tty --container nginx log-collector-0
    ```

### 建立转发

请参阅[Fluent Bit](https://docs.fluentbit.io/manual/installation/kubernetes)文档，设置一个默认情况下将日志转发到ElasticSearch的Fluent Bit DaemonSet。

当您在安装步骤中创建ConfigMap时，必须:

- 将ElasticSearch配置替换为[`fluent-bit-configmap.yaml`](fluent-bit-configmap.yaml), or
- 将下面的块添加到ConfigMap中，并将`@INCLUDE output-elasticsearch.conf`更新为`@INCLUDE output-forward.conf`:

    ```yaml
    output-forward.conf: |
      [OUTPUT]
          Name            forward
          Host            log-collector.logging
          Port            24224
          Require_ack_response  True
    ```

### 设置本地收集器

!!! warning
    此过程描述了一个开发环境设置，不适合生产使用。

如果使用本地Kubernetes集群进行开发，可以创建一个`hostPath` PersistentVolume 在桌面操作系统上存储日志。
这允许您在文件上使用常用的桌面工具，而不需要特定于kubernetes的工具。

`PersistentVolumeClaim`看起来类似如下:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: shared-logs
  labels:
    app: logs-collector
spec:
  accessModes:
    - "ReadWriteOnce"
  storageClassName: manual
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: logs-log-collector-0
    namespace: logging
  capacity:
    storage: 5Gi
  hostPath:
    path: <see below>
```

!!! note
    `hostPath`将根据您的Kubernetes软件和主机操作系统而有所不同。

您必须更新StatefulSet `volumeClaimTemplates`以引用`shared-logs`卷，示例如下:

```yaml
volumeClaimTemplates:
  metadata:
    name: logs
  spec:
    accessModes: ["ReadWriteOnce"]
    volumeName: shared-logs
```

### Kind

当创建集群时，你必须使用`kind-config.yaml`并为每个节点指定`extraMounts`，如下例所示:

```yaml
apiversion: kind.x-k8s.io/v1alpha4
kind: Cluster
nodes:
  - role: control-plane
    extraMounts:
      - hostPath: ./logs
        containerPath: /shared/logs
  - role: worker
    extraMounts:
      - hostPath: ./logs
        containerPath: /shared/logs
```

然后你可以使用`/shared/logs`作为PersistentVolume中的`spec.hostPath.path`。
注意目录路径`./logs`是相对于创建Kind集群的目录的。

### Docker 桌面

Docker桌面自动在主机和客户操作系统之间创建一些共享挂载，因此您只需要知道到您的主目录的路径。
以下是不同操作系统的一些例子:

| Host OS | `hostPath`                               |
| ------- | ---------------------------------------- |
| Mac OS  | `/Users/${USER}`                         |
| Windows | `/run/desktop/mnt/host/c/Users/${USER}/` |
| Linux   | `/home/${USER}`                          |

### Minikube

Minikube需要一个显式命令将一个[目录挂载](https://minikube.sigs.k8s.io/docs/handbook/mount/)到运行Kubernetes的虚拟机中。

以下命令将当前目录下的`logs`目录挂载到虚拟机的`/mnt/logs`目录中:

```bash
minikube mount ./logs:/mnt/logs
```

您还必须引用 `/mnt/logs` 作为PersistentVolume中的 `hostPath.path`。
