# 自动定量

Knative服务提供自动缩放，也称为**自动缩放**。
这意味着，Knative服务在不使用时，默认情况下会减少到零个运行的pod。

## 列出您的Knative服务

使用Knative (`kn`)命令行查看Knative服务所在的URL:

=== "kn"
    运行命令查看Knative服务列表:
    ```bash
    kn service list
    ```
    !!! Success "Expected output"
        ```bash
        NAME    URL                                                LATEST        AGE   CONDITIONS   READY
        hello   http://hello.default.${LOADBALANCER_IP}.sslip.io   hello-00001   13s   3 OK / 3     True
        ```
=== "kubectl"
    运行命令查看Knative服务列表:
    ```bash
    kubectl get ksvc
    ```
    !!! Success "Expected output"
        ```bash
        NAME    URL                                                LATESTCREATED   LATESTREADY   READY   REASON
        hello   http://hello.default.${LOADBALANCER_IP}.sslip.io   hello-00001     hello-00001   True
        ```

## 访问您的Knative服务

通过在浏览器中打开前面的URL或运行以下命令来访问您的Knative服务:

```bash
echo "Accessing URL $(kn service describe hello -o url)"
curl "$(kn service describe hello -o url)"
```

!!! Success "Expected output"
    ```{ .bash .no-copy }
    Hello World!
    ```

??? question "你看到 `curl: (6) Could not resolve host: hello.default.${LOADBALANCER_IP}.sslip.io`?"

    在某些情况下，您的DNS服务器可能设置为不解析`*.sslip.io`地址。
    如果遇到这个问题，可以通过使用不同的名称服务器来解决这些地址。

    具体的步骤将根据您的发行版有所不同。
    例如，对于使用`systemd-resolved`的Ubuntu派生系统，你可以在`/etc/systemd/resolved.conf`中添加以下条目:

    ```ini
    [Resolve]
    DNS=8.8.8.8
    Domains=~sslip.io.
    ```

    然后简单地用`sudo service systemd-resolved restart`重新启动服务。

    对于MacOS用户，可以使用[这里](https://support.apple.com/en-gb/guide/mac-help/mh14127/mac)所述的网络设置添加DNS和域。

## 观察自动伸缩

观察这些Pod，看看在流量停止访问URL后，它们是如何缩小到零的:

```bash
kubectl get pod -l serving.knative.dev/service=hello -w
```

!!! note
    可能需要2分钟才能让你的舱缩小。再次ping您的服务将重置此计时器。

!!! Success "Expected output"
    ```{ .bash .no-copy }
    NAME                                     READY   STATUS
    hello-world                              2/2     Running
    hello-world                              2/2     Terminating
    hello-world                              1/2     Terminating
    hello-world                              0/2     Terminating
    ```

## 扩大您的Knative服务

在浏览器中重新运行Knative服务。你可以看到一个新的Pod再次运行:

!!! Success "Expected output"
    ```{ .bash .no-copy }
    NAME                                     READY   STATUS
    hello-world                              0/2     Pending
    hello-world                              0/2     ContainerCreating
    hello-world                              1/2     Running
    hello-world                              2/2     Running
    ```

用`Ctrl+c`退出`kubectl watch`命令。
