# 创建代理

安装了Knative事件处理之后，就可以创建默认提供的基于多租户(MT)通道的代理实例。
基于MT通道的代理的默认后备通道类型是InMemoryChannel。

您可以通过使用 `kn`  CLI或使用 `kubectl` 应用YAML文件来创建代理。

=== "kn"

    1. 您可以通过输入以下命令在当前名称空间中创建代理:

        ```bash
        kn broker create <broker-name> -n <namespace>
        ```

        !!! note
            如果选择不指定名称空间，则代理将在当前名称空间中创建。

    1. 可选:验证是否通过列出现有的代理创建了代理。输入以下命令:

        ```bash
        kn broker list
        ```

    1. 可选:您还可以通过描述您创建的代理来验证代理是否存在。输入以下命令:

        ```bash
        kn broker describe <broker-name>
        ```


=== "kubectl"

    下面示例中的YAML在当前名称空间中创建了一个名为`default`的代理。

    1. 通过使用以下模板创建YAML文件，在当前名称空间中创建代理:

        ```yaml
        apiVersion: eventing.knative.dev/v1
        kind: Broker
        metadata:
         name: <broker-name>
        ```

    1. 通过运行该命令应用YAML文件:

        ```bash
        kubectl apply -f <filename>.yaml
        ```
        Where `<filename>` is the name of the file you created in the previous step.

    1. 可选:通过输入以下命令，验证代理是否正常工作:

        ```bash
        kubectl -n <namespace> get broker <broker-name>
        ```

        这将显示有关您的代理的信息。如果代理正常工作，它会显示`True`的`READY`状态:

        ```{ .bash .no-copy }
        NAME      READY   REASON   URL                                                                                 AGE
        default   True             http://broker-ingress.knative-eventing.svc.cluster.local/event-example/default      1m
        ```

        如果`READY`状态为`False`，等待几分钟，然后再次运行命令。
