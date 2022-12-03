# 自动缩放

Knative Serving为应用程序提供自动伸缩，或 _autoscaling_ ，以匹配传入的需求。
这是通过使用Knative Pod Autoscaler (KPA)默认提供的。


例如，如果一个应用程序没有接收流量，并且启用了向零扩展，Knative Serving将应用程序扩展到零副本。
如果禁用了向零伸缩，则应用程序将被缩小到集群上为应用程序指定的最小副本数量。
如果应用程序的流量增加，副本将被扩大以满足需求。

如果您具有集群管理员权限，可以为集群启用和禁用伸缩至零功能。
参见[配置缩放到零](scale-to-zero.md)。
<!--TODO: How can you check if you have it enabled if you're not a cluster admin?-->
如果在您的集群上启用了自动伸缩功能，要为您的应用程序使用自动伸缩功能，您必须配置[并发](concurrency.md)和[伸缩边界](scale-bounds.md)。
<!--TODO: Include this in the basic config before other settings-->

## 额外的资源

<!--TODO: Move KPA details, metrics to admin / advanced section; too in depth for intro)-->
* 试试[Go Autoscale Sample App](autoscale-go/README.md).
* 配置您的Knative部署以使用Kubernetes Horizontal Pod Autoscaler (HPA)而不是默认的KPA。关于如何安装HPA，请参见[安装可选服务扩展](../../install/yaml-install/serving/install-serving-with-yaml.md#install-optional-serving-extensions).
* 配置自动伸缩器使用的[度量类型](autoscaling-metrics.md).
* 配置您的Knative服务使用[container-freeze](container-freezer.md)，它会在Pod的流量降为零时冻结正在运行的进程。最有价值的好处是减少了这种配置中的冷启动时间。
