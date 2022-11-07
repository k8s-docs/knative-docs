# 安装 Knative CLI

本指南详细介绍了如何安装Knative `kn`命令行。

--8<-- "install-kn.md"

## 使用夜间生成的二进制文件安装kn
!!! warning
    每晚容器映像包括可能不包含在最新Knative版本中的特性，并且被认为是不稳定的。


想要安装 `kn` 最新预发布版本的用户可以使用夜间构建的可执行二进制文件。

到最新的夜间构建的可执行二进制文件的链接在这里:

- [macOS](https://storage.googleapis.com/knative-nightly/client/latest/kn-darwin-amd64){target=_blank}
- [Linux](https://storage.googleapis.com/knative-nightly/client/latest/kn-linux-amd64){target=_blank}
- [Windows](https://storage.googleapis.com/knative-nightly/client/latest/kn-windows-amd64.exe){target=_blank}

## Tekton使用 kn

参见[Tekton文档](http://hub.tekton.dev/tekton/task/kn){target=_blank}.
