---
hide_next: true
---
# 清理

我们建议您删除本教程中使用的集群，以释放本地机器上的资源。

如果您想在删除集群后继续使用Knative，
你可以使用[`quickstart` plugin](quickstart-install.md#run-the-knative-quickstart-plugin)在新的集群上重新安装Knative。

## 删除集群

=== "kind"

    通过运行以下命令删除你的`kind`集群:

    ```bash
    kind delete clusters knative
    ```
    !!! success "Example output"
        ```{ .bash .no-copy }
        Deleted clusters: ["knative"]
        ```

=== "minikube"

    通过以下命令删除你的`minikube`集群:

    ```bash
    minikube delete -p knative
    ```
    !!! success "Example output"
        ```{ .bash .no-copy }
        🔥  Deleting "knative" in hyperkit ...
        💀  Removed all traces of the "knative" cluster.
        ```
