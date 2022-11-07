<!-- Snippet used in the following topics:
- /docs/client/install-kn.md
- /docs/getting-started/quickstart-install.md
- docs/install/quickstart-install.md
-->

## 安装Knative CLI

Knative CLI (`kn`)为创建Knative资源(如Knative服务和事件源)提供了一个快速而简单的界面，而不需要直接创建或修改YAML文件。

`kn` CLI还简化了诸如自动伸缩和流量分割等复杂过程的完成。

=== "使用 Homebrew"

    做以下任何一件事:

    - To install `kn` by using [Homebrew](https://brew.sh){target=_blank}, run the command (Use `brew upgrade` instead if you are upgrading from a previous version):

        ```bash
        brew install knative/client/kn
        ```

        ??? bug "Having issues upgrading `kn` using Homebrew?"

            If you are having issues upgrading using Homebrew, it might be due to a change to a CLI repository where the `master` branch was renamed to `main`. Resolve this issue by running the command:

            ```bash
            brew uninstall kn
            brew untap knative/client --force
            brew install knative/client/kn
            ```

=== "使用二进制"

    您可以通过下载系统的可执行二进制文件并将其放在系统路径中来安装 `kn` 。

    1. Download the binary for your system from the [`kn` release page](https://github.com/knative/client/releases){target=_blank}.

    2. Rename the binary to `kn` and make it executable by running the commands:

        ```bash
        mv <path-to-binary-file> kn
        chmod +x kn
        ```

        Where `<path-to-binary-file>` is the path to the binary file you downloaded in the previous step, for example, `kn-darwin-amd64` or `kn-linux-amd64`.

    3. Move the executable binary file to a directory on your PATH by running the command:

        ```bash
        mv kn /usr/local/bin
        ```

    4. Verify that the plugin is working by running the command:

        ```bash
        kn version
        ```

=== "使用 Go"

    1. Check out the `kn` client repository:

          ```bash
          git clone https://github.com/knative/client.git
          cd client/
          ```

    1. Build an executable binary:

          ```bash
          hack/build.sh -f
          ```

    1. Move `kn` into your system path, and verify that `kn` commands are working properly. For example:

          ```bash
          kn version
          ```

=== "使用容器映像"

    Links to images are available here:

    - [Latest release](https://gcr.io/knative-releases/knative.dev/client/cmd/kn){target=_blank}

    You can run `kn` from a container image. For example:

    ```bash
    docker run --rm -v "$HOME/.kube/config:/root/.kube/config" gcr.io/knative-releases/knative.dev/client/cmd/kn:latest service list
    ```

    !!! note
        Running `kn` from a container image does not place the binary on a permanent path. This procedure must be repeated each time you want to use `kn`.
