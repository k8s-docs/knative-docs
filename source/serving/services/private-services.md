# 配置私有服务

默认情况下，通过Knative部署的服务被发布到一个外部IP地址，使它们成为公共IP地址和公共URL上的公共服务。

Knative提供了两种方法来启用仅在集群内部可用的私有服务:

1. 要使所有Knative服务私有，请将默认域更改为`svc.cluster.local`[编辑`config-domain` ConfigMap](https://github.com/knative/serving/blob/main/config/core/configmaps/domain.yaml). 
   这将更改通过Knative部署的所有服务，只将其发布到集群。
2. 要使单个服务私有，服务或路由可以标记为`networking.knative.dev/visibility=cluster-local`，这样它就不会被发布到外部网关。

## 使用 cluster-local 标签

要配置一个Knative服务，使其只在集群-本地网络上可用，而不是在公共互联网上可用，您可以将`networking.knative.dev/visibility=cluster-local`标签应用到Knative服务、路由或Kubernetes服务对象上。

- 标记本地服务:

    ```bash
    kubectl label kservice ${KSVC_NAME} networking.knative.dev/visibility=cluster-local
    ```

    通过标记Kubernetes服务，您可以以更细粒度的方式限制可见性。
    有关标记路由的信息，请参见[流量管理](../traffic-management.md)。

- 当路由直接被使用而没有本地服务时，要标记一个路由:

    ```bash
    kubectl label route ${ROUTE_NAME} networking.knative.dev/visibility=cluster-local
    ```

- 标记Kubernetes服务:

    ```bash
    kubectl label service ${SERVICE_NAME} networking.knative.dev/visibility=cluster-local
    ```

### 示例

您可以部署[Hello World示例](https://github.com/knative/docs/tree/main/code-samples/serving/hello-world/helloworld-go)，然后通过标记该服务将其转换为集群本地服务:

```bash
kubectl label kservice helloworld-go networking.knative.dev/visibility=cluster-local
```

然后，您可以通过验证 `helloworld-go` 服务的URL来验证更改已经完成:

```bash
kubectl get kservice helloworld-go

NAME            URL                                              LATESTCREATED         LATESTREADY           READY   REASON
helloworld-go   http://helloworld-go.default.svc.cluster.local   helloworld-go-2bz5l   helloworld-go-2bz5l   True
```

该服务返回带有`svc.cluster.local`域的URL，表示该服务仅在集群-本地网络中可用。
