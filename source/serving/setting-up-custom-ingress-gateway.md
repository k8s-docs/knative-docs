# 配置入口网关

Knative uses a shared ingress Gateway to serve all incoming traffic within
Knative service mesh, which is the `knative-ingress-gateway` Gateway under
the `knative-serving` namespace. By default, we use Istio gateway service
`istio-ingressgateway` under `istio-system` namespace as its underlying service.
You can replace the service and the gateway with that of your own as follows.

## 更换默认的`istio-ingressgateway`服务

### 步骤1: 创建网关服务和部署实例

You'll need to create the gateway service and deployment instance to handle
traffic first. Let's say you customized the default `istio-ingressgateway` to
`custom-ingressgateway` as follows.

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  components:
    ingressGateways:
      - name: custom-ingressgateway
        enabled: true
        namespace: custom-ns
        label:
          istio: custom-gateway
```

### 步骤2: 更新Knative网关

Update gateway instance `knative-ingress-gateway` under `knative-serving`
namespace:

```bash
kubectl edit gateway knative-ingress-gateway -n knative-serving
```

Replace the label selector with the label of your service:

```
istio: ingressgateway
```

For the example `custom-ingressgateway` service mentioned earlier, it should be updated to:

```
istio: custom-gateway
```

If there is a change in service ports (compared with that of
`istio-ingressgateway`), update the port info in the gateway accordingly.

### 步骤3: 更新网关ConfigMap

1. Update gateway configmap `config-istio` under `knative-serving` namespace:

     ```bash
     kubectl edit configmap config-istio -n knative-serving
     ```

     This command opens your default text editor and allows you to edit the config-istio ConfigMap.

     ```yaml
     apiVersion: v1
     data:
       _example: |
         ################################
         #                              #
         #    EXAMPLE CONFIGURATION     #
         #                              #
         ################################
         # ...
         gateway.knative-serving.knative-ingress-gateway: "istio-ingressgateway.istio-system.svc.cluster.local"
     ```

1. Edit the file to add the `gateway.knative-serving.knative-ingress-gateway: <ingress_name>.<ingress_namespace>.svc.cluster.local` field with
the fully qualified url of your service.
For the example `custom-ingressgateway` service mentioned earlier, it should be updated to:

     ```yaml
     apiVersion: v1
     data:
       gateway.knative-serving.knative-ingress-gateway: custom-ingressgateway.custom-ns.svc.cluster.local
     kind: ConfigMap
     [...]
     ```

## 替换`knative-ingress-gateway`网关

We customized the gateway service so far, but we may also want to use our own gateway.
We can replace the default gateway with our own gateway with following steps.

### 步骤1:创建网关

Let's say you replace the default `knative-ingress-gateway` gateway with
`knative-custom-gateway` in `custom-ns`.
First, create the `knative-custom-gateway` gateway:

1. Create a YAML file using the following template:

    ```yaml
    apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: knative-custom-gateway
      namespace: custom-ns
    spec:
      selector:
        istio: <service-label>
      servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        hosts:
        - "*"
    ```
    Where `<service-label>` is a label to select your service, for example, `ingressgateway`.

2. Apply the YAML file by running the command:

    ```bash
    kubectl apply -f <filename>.yaml
    ```
    Where `<filename>` is the name of the file you created in the previous step.

### 步骤2:更新网关ConfigMap

1. Update gateway configmap `config-istio` under `knative-serving`
namespace:

     ```bash
     kubectl edit configmap config-istio -n knative-serving
     ```

     This command opens your default text editor and allows you to edit the config-istio ConfigMap.

     ```yaml
     apiVersion: v1
     data:
       _example: |
         ################################
         #                              #
         #    EXAMPLE CONFIGURATION     #
         #                              #
         ################################
         # ...
         gateway.knative-serving.knative-ingress-gateway: "istio-ingressgateway.istio-system.svc.cluster.local"
     ```

1. Edit the file to add the `gateway.<gateway-namespace>.<gateway-name>: istio-ingressgateway.istio-system.svc.cluster.local` field with
the customized gateway.
For the example `knative-custom-gateway` mentioned earlier, it should be updated to:

     ```yaml
     apiVersion: v1
     data:
       gateway.custom-ns.knative-custom-gateway: "istio-ingressgateway.istio-system.svc.cluster.local"
     kind: ConfigMap
     [...]
     ```

The configuration format should be `gateway.<gateway-namespace>.<gateway-name>`.
`<gateway-namespace>` is optional. When it is omitted, the system searches for
the gateway in the serving system namespace `knative-serving`.
