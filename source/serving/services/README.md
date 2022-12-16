# 关于Knative服务

Knative服务用于部署应用程序。
要使用Knative创建应用程序，必须创建一个定义服务的YAML文件。
这个YAML文件指定关于应用程序的元数据，指向应用程序的托管映像，并允许配置服务。

每个服务都由一个路由和一个与服务同名的配置定义。
配置和路由由服务控制器创建，并从服务的配置派生出它们的配置。

每次更新配置时，都会创建一个新的修订。
修订是特定配置的不可变快照，并使用底层Kubernetes资源来根据流量扩展pods数量。

## 修改Knative服务

对服务的规范、元数据标签或元数据注释的任何更改都必须复制到该服务拥有的路由和配置中。
路由和配置上的`serving.knative.dev/service`标签也必须设置为服务的名称。
必须删除路由和配置上未指定的任何附加标签或注释。

服务根据所属路由和配置的相应`status`值更新其`status`字段。
除了通用的`Ready`条件外，服务还必须包括`RoutesReady` 和 `ConfigurationsReady`条件。
其他情况也可能出现。

## 额外的资源

* 有关Knative服务对象的更多信息，请参阅[资源类型](https://github.com/knative/specs/blob/main/specs/serving/overview.md)文档。
