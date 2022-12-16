# 配置资源请求和限制

您可以为各个Knative服务配置资源限制和请求，特别是CPU和内存。

下面的例子展示了如何为一个服务设置`requests` 和 `limits`字段:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: example-service
  namespace: default
spec:
  template:
    spec:
      containers:
        - image: docker.io/user/example-app
          resources:
            requests:
              cpu: 250m
              memory: 512Mi
            limits:
              cpu: 1
```

## 额外的资源

* 有关Kubernetes资源的请求和限制的更多信息，请参见[管理容器资源](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/).
