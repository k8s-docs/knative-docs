# CLI 工具



## kubectl

可以使用`kubectl`应用安装Knative组件所需的YAML文件，也可以使用YAML创建Knative资源，例如服务和事件源。

参见[安装和设置`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/){target=_blank}.

## kn

`kn` 为创建Knative资源(如服务和事件源)提供了一个快速、简单的接口，而不需要直接创建或修改YAML文件。
`kn` 还简化了自动伸缩和流量分割等复杂程序的完成。

!!! note
    `kn`不能用于安装Knative组件，如服务或事件。

### 额外的资源

- 查看 [安装 `kn`](install-kn.md).
- 参见Github中的[`kn` 文档]({{ clientdocs() }}){target=_blank}。

## func

`func`  CLI使您能够创建、构建和部署Knative函数，而不需要直接创建或修改YAML文件。

### 额外的资源

- 参见[安装Knative函数](../functions/install-func.md).
- 参见Github中的[`func`文档]({{ funcdocs() }}){target=_blank}。

## 将CLI工具连接到集群

安装了`kubectl`或`kn`后，这些工具将在默认位置`$HOME/.kube/config`中搜索集群的`kubeconfig`文件，并使用该文件连接到集群。
在创建Kubernetes集群时，通常会自动创建一个`kubeconfig`文件。

您还可以设置环境变量`$KUBECONFIG`，并将其指向KUBECONFIG文件。



- `--kubeconfig`: 使用此选项指向`kubeconfig`文件。这相当于设置`$KUBECONFIG`环境变量。
- `--context`: 使用此选项可从现有的`kubeconfig`文件中指定上下文的名称。使用`kubectl`输出中的一个上下文。


您还可以通过以下方式指定配置文件:

- 设置环境变量 `$KUBECONFIG`，并将其指向KUBECONFIG文件。

- 使用`kn` CLI `--config`选项，例如，`kn service list --config path/to/config.yaml`。
  默认的配置是`~/.config/kn/config.yaml`。

有关`kubeconfig`文件的更多信息，请参见[使用kubeconfig文件组织集群访问](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/){target=_blank}.

### 在平台上使用kubeconfig文件

使用`kubeconfig`文件的说明可用于以下平台:

- [Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html){target=_blank}
- [Google GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl){target=_blank}
- [IBM IKS](https://cloud.ibm.com/docs/containers?topic=containers-getting-started){target=_blank}
- [Red Hat OpenShift Cloud Platform](https://docs.openshift.com/container-platform/4.6/cli_reference/openshift_cli/administrator-cli-commands.html#create-kubeconfig){target=_blank}
- 启动[minikube](https://minikube.sigs.k8s.io/docs/start/){target=_blank}会自动写入该文件，或者在现有配置文件中提供适当的上下文。
