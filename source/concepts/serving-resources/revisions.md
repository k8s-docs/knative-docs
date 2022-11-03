# 修订

--8<-- "about-revisions.md"

## 相关的概念

### 自动缩放

修订可以根据传入的流量自动放大或缩小。
有关更多信息，请参见[自动缩放](../../serving/autoscaling/README.md){target=_blank}。

### 逐步推出修订流量

修订支持应用程序更改的逐步转出和回滚。
有关详细信息，请参见[配置逐步向修订版推出流量](../../serving/rolling-out-latest-revision.md){target=_blank}。

### 垃圾收集

--8<-- "about-revisions-garbage-collection.md"

有关更多信息，请参见[版本的配置选项](#configuration-options-for-revisions).

## 修订的配置选项

- [集群管理员(全局的、集群范围的)配置选项](../../serving/revisions/revision-admin-config-options.md)
- [开发人员(每修订)配置选项](../../serving/revisions/revision-developer-config-options.md)

## 额外资源

- [修订API规范](https://github.com/knative/specs/blob/main/specs/serving/knative-api-specification-1.0.md#revision){target=_blank}

## 下一步

- [使用Knative (`kn`)命令行删除、描述和列出修订](https://github.com/knative/client/blob/main/docs/cmd/kn_revision.md){target=_blank}
- [检查修订的状态](../../serving/troubleshooting/debugging-application-issues.md#check-revision-status){target=_blank}
- [在服务的修订之间路由通信](../../serving/traffic-management.md#traffic-routing-examples){target=_blank}
