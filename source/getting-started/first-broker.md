# 来源、代理和触发器

作为`kn quickstart`安装的一部分，一个`InMemoryChannel-backed`代理被安装在你的集群上。

运行以下命令验证代理是否已安装:

```bash
kn broker list
```

!!! Success "Expected output"
    ```{ .bash .no-copy }
    NAME             URL                                                                                AGE   CONDITIONS   READY   REASON
    example-broker   http://broker-ingress.knative-eventing.svc.cluster.local/default/example-broker     5m    5 OK / 5    True
    ```
!!! warning
    InMemoryChannel-backed代理仅用于开发，不能用于生产部署。

接下来，您将通过一个名为云事件播放器的应用程序了解源、代理、触发器和接收器的**简单实现**。