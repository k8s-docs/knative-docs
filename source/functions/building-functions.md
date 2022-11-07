# 构建函数

--8<-- "build-func-intro.md"

## 本地构建

您可以使用 `build` 命令在本地为函数构建容器映像，而无需将其部署到集群中。
### 先决条件

- 您的本地机器上有一个Docker守护进程。如果您已经使用了快速入门安装，则已经提供了该功能。

### 过程

--8<-- "proc-building-function.md"

## 集群构建

如果您没有运行本地Docker守护进程，或者您正在使用CI/CD管道，那么您可能希望在集群上构建函数，而不是使用本地构建。
您可以使用`func deploy --remote`命令创建一个集群上构建。

### 先决条件

- 该函数必须存在于Git存储库中。
- 您必须配置您的集群以使用Tekton pipeline。请参阅[集群构建](https://github.com/knative/func/blob/main/docs/reference/on_cluster_build.md){target=_blank}文档。

### 过程

第一次运行该命令时，必须指定该函数的Git URL:

=== "func"

    ```{ .console }
    func deploy --remote --registry <registry> --git-url <git-url> -p hello
    ```

=== "kn func"

    ```{ .console }
    kn func deploy --remote --registry <registry> --git-url <git-url> -p hello
    ```

在为函数指定Git URL一次之后，可以在后续命令中省略它。
