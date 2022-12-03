<!-- Snippet used in the following topics:
- /docs/getting-started/quickstart-install.md
- /docs/install/quickstart-install.md
-->

## 安装 Knative quickstart 插件

要开始，安装 knative `quickstart` 插件:

=== "使用 Homebrew"

    做以下任何一件事:

    - 要使用[Homebrew](https://brew.sh){target=_blank})安装`quickstart`插件，请运行以下命令(如果您是从以前的版本升级，请使用 `brew upgrade` 代替):

        ```bash
        brew install knative-sandbox/kn-plugins/quickstart
        ```

=== "使用二进制"

    1. 从[`quickstart` 发布页面](https://github.com/knative-sandbox/kn-plugin-quickstart/releases){target=_blank}下载您系统的二进制文件.

    1. 重命名文件以删除操作系统和体系结构信息。例如，将`kn-quickstart-amd64`重命名为`kn-quickstart`。

    1. 使插件可执行。例如，`chmod +x kn-quickstart`。

    1. 通过运行以下命令将可执行的二进制文件移动到“PATH”上的某个目录:

        ```bash
        mv kn-quickstart /usr/local/bin
        ```

    1. 通过运行以下命令验证插件是否正在工作:

        ```bash
        kn quickstart --help
        ```

=== "使用 Go"

    1. 查看`kn-plugin-quickstart`库:

          ```bash
          git clone https://github.com/knative-sandbox/kn-plugin-quickstart.git
          cd kn-plugin-quickstart/
          ```

    2. 构建一个可执行的二进制文件:

          ```bash
          hack/build.sh
          ```

    3. 将可执行的二进制文件移动到“PATH”上的目录:

          ```bash
          mv kn-quickstart /usr/local/bin
          ```

     4. 通过运行以下命令验证插件是否正在工作:

          ```bash
          kn quickstart --help
          ```

## 运行 Knative quickstart 插件

`quickstart`插件完成以下功能:

1. **检查是否安装了选定的 Kubernetes 实例**
1. **创建一个名为`knative`的集群**
1. **安装 Knative 服务** 其中，Kourier 作为默认的网络层，sslip.io 作为 DNS
1. **安装 Knative 事件** 并创建内存中代理和通道实现

要获得 Knative 的本地部署，运行 `quickstart` 插件:

=== "使用 kind"

    1. 使用[kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)安装Knative和Kubernetes:

        ```bash
        kn quickstart kind
        ```

    1. 插件完成后，验证你有一个名为`knative`的集群:

        ```bash
        kind get clusters
        ```

=== "使用 minikube"

    1. 在[minikube](https://minikube.sigs.k8s.io/docs/start/)实例中安装Knative和Kubernetes:

        !!! note

            minikube集群将使用6&nbsp;GB的RAM创建。
            如果没有足够的内存，可以在此命令之前运行命令`minikube config set memory 3078`，将其更改为不低于3&nbsp;GB的值。
        ```bash
        kn quickstart minikube
        ```

    1. 上一个命令的输出要求您运行minikube tunnel。
       运行以下命令在辅助终端窗口中启动进程，然后返回到主窗口并按回车继续:

        ```bash
        minikube tunnel --profile knative
        ```

        在使用Knative“quickstart”环境时，隧道必须在终端窗口中继续运行。

        需要使用tunnel命令，因为它允许您的集群作为LoadBalancer从您的主机访问Knative ingress服务。

        !!! note

            输入`Ctrl-C`可以终止隧道进程并清理网络路由。
            有关`minikube tunnel`命令的更多信息，请参见[minikube文档](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-tunnel).

    1. 插件完成后，验证你有一个名为`knative`的集群:

        ```bash
        minikube profile list
        ```
