# 将Kubernetes部署转换为Knative服务

本主题展示如何将Kubernetes部署转换为Knative服务。

## 好处

转换为Knative服务有以下好处:

- 减少服务实例的占用空间，因为当实例变为空闲时，它会扩展到0。
- 由于Knative服务的内置自动伸缩，因此提高了性能。

## 确定您的工作负载是否适合Knative

一般来说，如果您的Kubernetes工作负载非常适合Knative，您可以删除大量清单来创建Knative Service。

你需要考虑三个方面:

- 所有完成的工作都由HTTP触发。
- 容器是无状态的。所有状态都存储在其他地方，或者可以重新创建。
- 您的工作负载只使用Secret和ConfigMap卷。

## 转换示例

下面的例子展示了[Kubernetes Nginx部署和服务](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/), 并展示了它如何转换为Knative服务。

### Kubernetes Nginx 部署和服务

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx

```

### Knative 服务

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-nginx
spec:
  template:
    spec:
      containers:
      - image: nginx
        ports:
        - containerPort: 80
```
