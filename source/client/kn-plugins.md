# kn 插件

`kn` 命令行支持使用插件。
插件允许您通过添加自定义命令和其他不属于 `kn` 核心发行版的共享命令来扩展`kn`安装的功能。

!!! warning

    插件必须以前缀`kn-`命名，以便由`kn`检测。
    例如，`kn-func`会被检测到，但`func`不会被检测到。

## kn 源的插件

事件源插件具有以下特征:

- 它的名称属于`kn source`组的一部分。
- 它提供 CRUD 子命令; `create`, `update`, `delete`, `describe`, 有时 `apply`.
- 当使用`create`命令时，它要求传递一个强制的`--sink`标志。

## Knative 插件列表

您可以在[Knative Sandbox 库](https://github.com/orgs/knative-sandbox/repositories?q=kn+plugin&type=all&language=&sort=)中查看所有可用的`kn`插件.

<!--TODO: If we're including the following table, the Client WG must be responsible for ensuring that the table is kept up to date, otherwise it should be removed from the docs and just the link to the sandbox repo should be provided-->

| 插件                                                                                    | 描述                                                                  | 可以通过 Homebrew? |
| --------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | :----------------: |
| [kn-plugin-admin](https://github.com/knative-sandbox/kn-plugin-admin)                   | `kn` plugin 用于管理基于 Kubernetes 的 Knative 安装                   |         Y          |
| [kn-plugin-diag](https://github.com/knative-sandbox/kn-plugin-diag)                     | `kn` plugin 用于通过公开 Knative 对象的不同层的详细信息来诊断问题     |         N          |
| [kn-plugin-event](https://github.com/knative-sandbox/kn-plugin-event)                   | `kn` plugin 用于将事件发送到 Knative 接收器                           |         Y          |
| [kn-plugin-func](https://github.com/knative/func)                                       | `kn` plugin 用户函数                                                  |         Y          |
| [kn-plugin-migration](https://github.com/knative-sandbox/kn-plugin-migration)           | `kn` plugin 用于将 Knative 服务从一个集群迁移到另一个集群             |         N          |
| [kn-plugin-operator](https://github.com/knative-sandbox/kn-plugin-operator)             | `kn` plugin 使用 Knative Operator 管理 Knative                        |         N          |
| [kn-plugin-quickstart](https://github.com/knative-sandbox/kn-plugin-quickstart)         | `kn` plugin 为开发人员安装一个 quickstart 的 Knative 集群，以进行实验 |         Y          |
| [kn-plugin-service-log](https://github.com/knative-sandbox/kn-plugin-service-log)       | `kn` plugin 用于显示 Knative 服务的标准输出                           |         N          |
| [kn-plugin-source-kafka](https://github.com/knative-sandbox/kn-plugin-source-kafka)     | `kn` plugin 用于管理 Kafka 事件源                                     |         Y          |
| [kn-plugin-source-kamelet](https://github.com/knative-sandbox/kn-plugin-source-kamelet) | `kn` plugin 用于管理 Kamelets 和 KameletBindings                      |         Y          |

## 手动安装插件

您可以手动安装所有插件。手动安装插件:

1. 从 GitHub 下载插件的当前版本。你可以下载[Knative 插件列表](#list-of-knative-plugins)。
1. 重命名文件以删除操作系统和体系结构信息。例如，将`kn-admin-darwin-amd64`重命名为`kn-admin`。
1. 使插件可执行。例如，`chmod +x kn-admin`。
1. 将文件移动到`PATH`上的目录中。例如,`/usr/local/bin`。

## 使用 Homebrew 安装插件

可以使用[Knative plugins Homebrew Tap](https://github.com/knative-sandbox/homebrew-kn-plugins/)安装一些插件。
例如，你可以通过运行`brew install knative-sandbox/kn-plugins/admin`来安装`kn-admin`插件。

## 可用插件列表

你可以输入以下命令列出所有可用的(已安装的)插件:

```bash
kn plugin list
```
