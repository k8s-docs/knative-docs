## 创建服务

1.  创建 `event-display` 服务作为一个 YAML 文件:

    ```yaml
    apiVersion: serving.knative.dev/v1
    kind: Service
    metadata:
      name: event-display
      namespace: default
    spec:
      template:
        spec:
          containers:
            - # This corresponds to
              # https://github.com/knative/eventing/tree/main/cmd/event_display/main.go
              image: gcr.io/knative-releases/knative.dev/eventing/cmd/event_display
    ```

1.  运行以下命令应用 YAML 文件:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

    其中 `<filename>` 是您在上一步中创建的文件的名称。

    示例输出:

    ```{ .bash .no-copy }
    service.serving.knative.dev/event-display created
    ```

1.  确保服务 Pod 正在运行，运行命令:

    ```bash
    kubectl get pods
    ```

    Pod 名称的前缀是 `event-display`:

    ```{ .bash .no-copy }
    NAME                                            READY     STATUS    RESTARTS   AGE
    event-display-00001-deployment-5d5df6c7-gv2j4   2/2       Running   0          72s
    ```
