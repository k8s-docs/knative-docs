### 配置 DNS

可以配置DNS以避免使用主机标头运行curl命令。

下面的选项卡展开显示配置DNS的说明。
按照以下步骤选择DNS:

=== "Magic DNS (sslip.io)"

    Knative provides a Kubernetes Job called `default-domain` that configures Knative Serving to use [sslip.io](http://sslip.io) as the default DNS suffix.

    ```bash
    kubectl apply -f {{artifact(repo="serving",file="serving-default-domain.yaml")}}
    ```

    !!! warning
        This will only work if the cluster `LoadBalancer` Service exposes an
        IPv4 address or hostname, so it will not work with IPv6 clusters or local setups
        like minikube unless [`minikube tunnel`](https://minikube.sigs.k8s.io/docs/commands/tunnel/)
        is running.

        In these cases, see the "Real DNS" or "Temporary DNS" tabs.
