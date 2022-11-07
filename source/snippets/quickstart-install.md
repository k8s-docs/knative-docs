<!-- Snippet used in the following topics:
- /docs/getting-started/quickstart-install.md
- /docs/install/quickstart-install.md
-->
## 安装Knative快速启动插件

要开始，安装knative  `quickstart` 插件:

=== "使用 Homebrew"

    Do one of the following:

    - To install the `quickstart` plugin by using [Homebrew](https://brew.sh){target=_blank}, run the command (Use `brew upgrade` instead if you are upgrading from a previous version):

        ```bash
        brew install knative-sandbox/kn-plugins/quickstart
        ```

=== "使用二进制"

    1. Download the binary for your system from the [`quickstart` release page](https://github.com/knative-sandbox/kn-plugin-quickstart/releases){target=_blank}.

    1. Rename the file to remove the OS and architecture information. For example, rename `kn-quickstart-amd64` to `kn-quickstart`.

    1. Make the plugin executable. For example, `chmod +x kn-quickstart`.

    1. Move the executable binary file to a directory on your `PATH` by running the command:

        ```bash
        mv kn-quickstart /usr/local/bin
        ```

    1. Verify that the plugin is working by running the command:

        ```bash
        kn quickstart --help
        ```

=== "使用 Go"
    1. Check out the `kn-plugin-quickstart` repository:

          ```bash
          git clone https://github.com/knative-sandbox/kn-plugin-quickstart.git
          cd kn-plugin-quickstart/
          ```

    1. Build an executable binary:

          ```bash
          hack/build.sh
          ```

    1. Move the executable binary file to a directory on your `PATH`:

          ```bash
          mv kn-quickstart /usr/local/bin
          ```

     1. Verify that the plugin is working by running the command:

          ```bash
          kn quickstart --help
          ```

## 运行Knative快速启动插件

`quickstart`插件完成以下功能:

1. **检查是否安装了选定的Kubernetes实例**
1. **创建一个名为`knative`的集群**
1. **安装Knative服务** 其中，Kourier作为默认的网络层，sslip.io作为DNS
1. **安装Knative事件** 并创建内存中代理和通道实现


要获得Knative的本地部署，运行 `quickstart` 插件:

=== "使用 kind"


    1. Install Knative and Kubernetes using [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) by running:

        ```bash
        kn quickstart kind
        ```

    1. After the plugin is finished, verify you have a cluster called `knative`:

        ```bash
        kind get clusters
        ```

=== "使用 minikube"

    1. Install Knative and Kubernetes in a [minikube](https://minikube.sigs.k8s.io/docs/start/) instance by running:

        !!! note
            The minikube cluster will be created with 6&nbsp;GB of RAM. If you don't have enough memory, you can change to a
            different value not lower than 3&nbsp;GB by running the command `minikube config set memory 3078` before this command.
        ```bash
        kn quickstart minikube
        ```

    1. The output of the previous command asked you to run minikube tunnel.
       Run the following command to start the process in a secondary terminal window, then return to the primary window and press enter to continue:
        ```bash
        minikube tunnel --profile knative
        ```
        The tunnel must continue to run in a terminal window any time you are using your Knative `quickstart` environment.

        The tunnel command is required because it allows your cluster to access Knative ingress service as a LoadBalancer from your host computer.

        !!! note
            To terminate the tunnel process and clean up network routes, enter `Ctrl-C`.
            For more information about the `minikube tunnel` command, see the [minikube documentation](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-tunnel).

    1. After the plugin is finished, verify you have a cluster called `knative`:

        ```bash
        minikube profile list
        ```
