<!-- Snippet used in the following topics:
- /docs/client/install-kn.md
- /docs/getting-started/quickstart-install.md
- docs/install/quickstart-install.md
-->

## 安装 Knative CLI

Knative CLI (`kn`)为创建 Knative 资源(如 Knative 服务和事件源)提供了一个快速而简单的界面，而不需要直接创建或修改 YAML 文件。

`kn` CLI 还简化了诸如自动伸缩和流量分割等复杂过程的完成。

=== "使用 Homebrew"

    做以下任何一件事:

    - 使用[Homebrew](https://brew.sh){target=_blank}安装`kn`, 运行命令(如果你从以前的版本升级，请使用`brew upgrade`代替):

        ```bash
        brew install knative/client/kn
        ```

        ??? bug "在使用Homebrew升级`kn`时有问题吗?"

            如果你在使用Homebrew升级时遇到问题，这可能是由于对CLI存储库的更改，其中`master`分支被重命名为`main`分支。
            可以通过以下命令解决此问题:

            ```bash
            brew uninstall kn
            brew untap knative/client --force
            brew install knative/client/kn
            ```

=== "使用二进制"

    您可以通过下载系统的可执行二进制文件并将其放在系统路径中来安装 `kn` 。

    1. 从[`kn`发布页面](https://github.com/knative/client/releases){target=_blank}为您的系统下载二进制文件.

    2. 将二进制文件重命名为`kn`，并通过运行命令使其可执行:

        ```bash
        mv <path-to-binary-file> kn
        chmod +x kn
        ```

        Where `<path-to-binary-file>` is the path to the binary file you downloaded in the previous step, for example, `kn-darwin-amd64` or `kn-linux-amd64`.

    3. 通过运行以下命令，将可执行的二进制文件移动到PATH上的某个目录:

        ```bash
        mv kn /usr/local/bin
        ```

    4. 通过运行以下命令验证插件是否正在工作:

        ```bash
        kn version
        ```

=== "使用 Go"

    1. 查看`kn`客户端存储库:

          ```bash
          git clone https://github.com/knative/client.git
          cd client/
          ```

    1. 构建一个可执行的二进制文件:

          ```bash
          hack/build.sh -f
          ```

    1. 将`kn`移动到系统路径中，并验证`kn`命令是否正常工作。例如:

          ```bash
          kn version
          ```

=== "使用容器映像"

    映像链接在这里:

    - [最新版本](https://gcr.io/knative-releases/knative.dev/client/cmd/kn){target=_blank}

    你可以从容器映像中运行`kn`。例如:

    ```bash
    docker run --rm -v "$HOME/.kube/config:/root/.kube/config" gcr.io/knative-releases/knative.dev/client/cmd/kn:latest service list
    ```

    !!! note

        从容器映像运行`kn`不会将二进制文件放在永久路径上。每次使用`kn`时都必须重复此过程。
