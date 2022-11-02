<!-- Snippet used in the following topics:
- /docs/concepts/servng-resources/revisions.md
- /docs/serving/revisions/README.md
-->

修订是 Knative 服务资源，其中包含应用程序代码的时间点快照，以及对 Knative 服务所做的每次更改的配置。

您不能直接创建修订或更新修订规范;修订总是根据配置规范的更新而创建的。
但是，您可以强制删除修订，以处理泄漏的资源，以及删除已知的坏修订，以避免在管理 Knative Service 时出现未来的错误。

修订通常是不可变的，除非它们可能引用可变的核心 Kubernetes 资源，如 ConfigMaps 和 Secrets。
修改版本也可以通过修改版本默认值的更改而发生突变。
对默认值的更改会使修订版本发生变化，这通常是语法上的，而不是语义上的。
