<!-- Snippet used in the following topics:
- docs/serving/revisions/revision-admin-config-options.md
- docs/serving/revisions/revision-developer-config-options.md
-->

当 Knative 服务的修订版处于不活动状态时，它们将在设定的时间段后自动清理并回收集群资源。
这就是所谓的 _[垃圾收集](https://kubernetes.io/docs/concepts/architecture/garbage-collection/){target=\_blank}_.

如果您是开发人员，可以在特定版本中配置垃圾收集参数。
如果您具有集群管理员权限，还可以为集群上所有服务的所有部分配置默认的、集群范围的垃圾收集参数。
