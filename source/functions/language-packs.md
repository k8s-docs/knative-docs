# 语言包

语言包可用于扩展Knative函数，以支持额外的运行时、函数签名、操作系统和已安装的函数工具。
语言包通过Git存储库或作为磁盘上的目录分发。

有关更多信息，请参阅[语言包](https://github.com/knative/func/blob/main/docs/language-pack-providers/language-pack-contract.md){target=_blank}文档。

## 使用外部Git存储库

在创建新函数时，可以指定Git存储库作为模板文件的源。
Knative沙盒维护了一组[示例模板](https://github.com/knative-sandbox/func-tastic){target=_blank}，可以在项目创建过程中使用。

例如，你可以运行以下命令为Node.js使用[`metacontroller`](https://metacontroller.github.io/metacontroller/){target=_blank}模板:

```{ .console }
func create myfunc -l nodejs -t metacontroller --repository https://github.com/knative-sandbox/func-tastic
```

## 在本地安装语言包

语言包可以通过使用[`func repository`](https://github.com/knative/func/blob/main/docs/reference/func_repository.md){target=_blank}命令在本地安装。

例如，要添加Knative Sandbox示例模板，可以运行以下命令:

```{ .console }
func repository add knative https://github.com/knative-sandbox/func-tastic
```

安装Knative沙盒示例模板后，你可以通过在 `create` 命令中指定 `Knative` 前缀来使用 `metacontroller` 模板:

```{ .console }
func create -t knative/metacontroller -l nodejs my-controller-function
```
