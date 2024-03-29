# 为Knative安装Istio

本指南将指导您手动安装和自定义与Knative一起使用的Istio。

如果您的云平台提供托管的Istio安装，我们建议以这种方式安装Istio，除非您需要自定义您的安装。

## 在开始之前

你需要:

- 完成Kubernetes集群的创建。
- [`istioctl`](https://istio.io/docs/setup/install/istioctl/) 安装.

## 支持的Istio版本

您可以在[Knative Net Istio发布页面](https://github.com/knative-sandbox/net-istio/releases)上查看最新测试的Istio版本.

## 安装Istio

When you install Istio, there are a few options depending on your goals. For a
basic Istio installation suitable for most Knative use cases, follow the
[Installing Istio without sidecar injection](#installing-istio-without-sidecar-injection)
instructions. If you're familiar with Istio and know what kind of installation
you want, read through the options and choose the installation that suits your
needs.

You can easily customize your Istio installation with `istioctl`. The following sections
cover a few useful Istio configurations and their benefits.

### 选择Istio安装

你可以安装带有或不带有服务网格的Istio:

- [Installing Istio without sidecar injection](#installing-istio-without-sidecar-injection) (Recommended
     default installation)

- [Installing Istio with sidecar injection](#installing-istio-with-sidecar-injection)

If you want to get up and running with Knative quickly, we recommend installing
Istio without automatic sidecar injection. This install is also recommended for
users who don't need the Istio service mesh, or who want to enable the service
mesh by [manually injecting the Istio sidecars][1].

#### 安装Istio不需要sidecar注入

输入以下命令安装Istio:

安装Istio时不需要sidecar注入:

```sh
istioctl install -y
```

#### 安装Istio需要sidecar注入

If you want to enable the Istio service mesh, you must enable [automatic sidecar
injection][2]. The Istio service mesh provides a few benefits:

- Allows you to turn on [mutual TLS][3], which secures service-to-service
  traffic within the cluster.

- Allows you to use the [Istio authorization policy][4], controlling the access
  to each Knative service based on Istio service roles.

For automatic sidecar injection, set `autoInject: enabled` in addition to the earlier
operator configuration.

```
    global:
      proxy:
        autoInject: enabled
```

#### 使用Istio mTLS特性

Since there are some networking communications between knative-serving namespace
and the namespace where your services running on, you need additional
preparations for mTLS enabled environment.

1. Enable sidecar container on `knative-serving` system namespace.

    ```bash
    kubectl label namespace knative-serving istio-injection=enabled
    ```

1. Set `PeerAuthentication` to `PERMISSIVE` on knative-serving system namespace by creating a YAML file using the following template:

    ```bash
    apiVersion: "security.istio.io/v1beta1"
    kind: "PeerAuthentication"
    metadata:
      name: "default"
      namespace: "knative-serving"
    spec:
      mtls:
        mode: PERMISSIVE
    ```

1. Apply the YAML file by running the command:

    ```bash
    kubectl apply -f <filename>.yaml
    ```
    Where `<filename>` is the name of the file you created in the previous step.

After you install the cluster local gateway, your service and deployment for the local gateway is named `knative-local-gateway`.

### 更新 `config-istio`  configmap以使用非默认本地网关

If you create a custom service and deployment for local gateway with a name other than `knative-local-gateway`, you
need to update gateway configmap `config-istio` under the `knative-serving` namespace.

1. Edit the `config-istio` configmap:

    ```bash
    kubectl edit configmap config-istio -n knative-serving
    ```

2. Replace the `local-gateway.knative-serving.knative-local-gateway` field with the custom service. As an example, if you name both
the service and deployment `custom-local-gateway` under the namespace `istio-system`, it should be updated to:

    ```
    custom-local-gateway.istio-system.svc.cluster.local
    ```

As an example, if both the custom service and deployment are labeled with `custom: custom-local-gateway`, not the default
`istio: knative-local-gateway`, you must update gateway instance `knative-local-gateway` in the `knative-serving` namespace:

```bash
kubectl edit gateway knative-local-gateway -n knative-serving
```

Replace the label selector with the label of your service:

```
istio: knative-local-gateway
```

For the service mentioned earlier, it should be updated to:

```
custom: custom-local-gateway
```

If there is a change in service ports (compared to that of
`knative-local-gateway`), update the port info in the gateway accordingly.

### 验证您的Istio安装

View the status of your Istio installation to make sure the install was
successful. It might take a few seconds, so rerun the following command until
all of the pods show a `STATUS` of `Running` or `Completed`:

```bash
kubectl get pods --namespace istio-system
```
!!! tip
    You can append the `--watch` flag to the `kubectl get` commands to view
    the pod status in realtime. You use `CTRL + C` to exit watch mode.

### 配置DNS

Knative dispatches to different services based on their hostname, so it is recommended to have DNS properly configured.

To do this, begin by looking up the external IP address that Istio received:

```
$ kubectl get svc -nistio-system
NAME                    TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)                                      AGE
istio-ingressgateway    LoadBalancer   10.0.2.24    34.83.80.117   15020:32206/TCP,80:30742/TCP,443:30996/TCP   2m14s
istio-pilot             ClusterIP      10.0.3.27    <none>         15010/TCP,15011/TCP,8080/TCP,15014/TCP       2m14s
```

This external IP can be used with your DNS provider with a wildcard `A` record. However, for a basic non-production set
up, this external IP address can be used with `sslip.io` in the `config-domain` ConfigMap in `knative-serving`.

You can edit this by using the following command:

```
kubectl edit cm config-domain --namespace knative-serving
```

Given this external IP, change the content to:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-domain
  namespace: knative-serving
data:
  # sslip.io is a "magic" DNS provider, which resolves all DNS lookups for:
  # *.{ip}.sslip.io to {ip}.
  34.83.80.117.sslip.io: ""
```

## Istio 资源

- For the official Istio installation guide, see the
  [Istio Kubernetes Getting Started Guide](https://istio.io/docs/setup/kubernetes/).

- For the full list of available configs when installing Istio with `istioctl`, see
  the
  [Istio Installation Options reference](https://istio.io/docs/setup/install/istioctl/).

## 清理Istio

See the [Uninstall Istio](https://istio.io/docs/setup/install/istioctl/#uninstall-istio).

## 接下来是什么

- View the [Knative Serving documentation](../serving/README.md).
- Try some Knative Serving [code samples](../samples/README.md).

[1]: https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/#manual-sidecar-injection
[2]: https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/#automatic-sidecar-injection
[3]: https://istio.io/docs/concepts/security/#mutual-tls-authentication
[4]: https://istio.io/docs/tasks/security/authz-http/
