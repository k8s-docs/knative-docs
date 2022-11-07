# 配置域名

您可以自定义单个Knative服务的域，或者为集群上创建的所有服务设置全局默认域。
缺省情况下，路由的完全限定域名为`{route}.{namespace}.example.com`。

## 为单个Knative服务配置域

如果您想自定义单个服务的域，请参见有关[`DomainMapping`](services/custom-domains.md)的文档。

## 为集群中的所有Knative服务配置默认域

通过修改 [`config-domain` ConfigMap](https://github.com/knative/serving/blob/main/config/core/configmaps/domain.yaml)，可以更改集群中所有Knative服务的默认域。

### 过程

1. 在默认文本编辑器中打开`config-domain` ConfigMap

    ```bash
    kubectl edit configmap config-domain -n knative-serving
    ```

1. 编辑文件，将`example.com`替换为您想要使用的域，然后删除`_example`键并保存更改。
   在本例中，`knative.dev`被配置为所有路由的域:

    ```yaml
    apiVersion: v1
    data:
      knative.dev: ""
    kind: ConfigMap
    [...]
    ```

如果您有一个现有的部署，Knative会协调对ConfigMap所做的更改，并自动更新所有部署的服务和路由的主机名。

## 验证步骤

1. 将一个应用程序部署到您的集群。
1. 检索路由的URL:

    ```bash
    kubectl get route <route-name> --output jsonpath="{.status.url}"
    ```

    其中`<route-name>`是路由的名称。

1. 观察已配置的自定义域。

## 发布您的域

要使您的域公开可访问，您必须更新您的DNS提供程序以指向您的服务入口的IP地址。

1. 为命名空间创建[通配符记录](https://support.google.com/domains/answer/4633759)，并自定义入接口IP地址的域，这将使同一命名空间中的多个服务的主机名能够工作，而无需创建额外的DNS条目。

    ```dns
    *.default.knative.dev                   59     IN     A   35.237.28.44
    ```

1. 创建一个A记录，从完全限定的域名指向您的Knative网关的IP地址。
   需要为创建的每个Knative服务或路由执行此步骤。

    ```dns
    helloworld-go.default.knative.dev       59     IN     A   35.237.28.44
    ```

1. 在域更新传播之后，您可以通过使用部署路由的完全限定域名访问您的应用程序。