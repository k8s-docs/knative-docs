# 流量分发

流量分发对于[蓝/绿部署](https://martinfowler.com/bliki/BlueGreenDeployment.html){target=blank_}和[金丝雀部署](https://martinfowler.com/bliki/CanaryRelease.html){target=blank_}很有用。

[修订版](../serving/ readme .md#serving-resources){target=_blank}是应用程序代码和配置的实时快照。
每次更改Knative服务的配置时，都会创建一个新的修订。
当分流流量时，Knative会在Knative服务的不同修订版之间分流流量。

## 创建一个新的修订

替换`TARGET=World`，更新Knative服务上的环境变量`TARGET`值为"Knative"。

=== "kn"

    通过运行以下命令部署Knative服务的更新版本:

    ``` bash
    kn service update hello \
    --env TARGET=Knative
    ```

    和前面一样，`kn` 向CLI输出一些有用的信息。

    !!! Success "Expected output"
        ```{ .bash .no-copy }
        Service 'hello' created to latest revision 'hello-00002' is available at URL:
        http://hello.default.${LOADBALANCER_IP}.sslip.io
        ```

=== "YAML"
    1. 编辑你现有的 `hello.yaml` 文件以包含以下内容:
        ``` yaml
        apiVersion: serving.knative.dev/v1
        kind: Service
        metadata:
          name: hello
        spec:
          template:
            spec:
              containers:
                - image: gcr.io/knative-samples/helloworld-go
                  ports:
                    - containerPort: 8080
                  env:
                    - name: TARGET
                      value: "Knative"
        ```
    1. 通过运行以下命令部署Knative服务的更新版本:
        ``` bash
        kubectl apply -f hello.yaml
        ```

        !!! Success "Expected output"
            ```{ .bash .no-copy }
            service.serving.knative.dev/hello configured
            ```

因为您正在更新一个现有的Knative服务，所以URL不会改变，但是新的修订版本有了新的名称 `hello-00002`。

## 访问新的修订版本

要查看更改，请在浏览器上再次访问Knative服务或在终端上使用 `curl`:

```bash
echo "Accessing URL $(kn service describe hello -o url)"
curl "$(kn service describe hello -o url)"
```

!!! Success "Expected output"
    ```{ .bash .no-copy }
    Hello Knative!
    ```

## 查看现有的修正

您可以使用Knative (`kn`) or `kubectl`命令行查看现有修订的列表:

=== "kn"
    运行该命令查看版本列表:
    ```bash
    kn revisions list
    ```
    !!! Success "Expected output"
        ```{ .bash .no-copy }
        NAME            SERVICE   TRAFFIC   TAGS   GENERATION   AGE   CONDITIONS   READY   REASON
        hello-00002     hello     100%             2            30s   3 OK / 4     True
        hello-00001     hello                      1            5m    3 OK / 4     True
        ```

=== "kubectl"
    运行该命令查看版本列表:
    ```bash
    kubectl get revisions
    ```
    !!! Success "Expected output"
        ```{ .bash .no-copy }
        NAME          CONFIG NAME   K8S SERVICE NAME   GENERATION   READY   REASON   ACTUAL REPLICAS   DESIRED REPLICAS
        hello-00001   hello                            1            True             0                 0
        hello-00002   hello                            2            True             0                 0
        ```

当运行`kn`命令时，相关列为`TRAFFIC`。
你可以看到100%的流量都流向了最新的版本`hello-00002`，它位于`GENERATION`最高的那一行。
0%的流量将流向修订版`hello-00001`。

当您创建Knative服务的新版本时，Knative默认将100%的流量导向这个最新版本。
您可以通过指定希望每个修订接收多少流量来更改此默认行为。

## 在修订版之间划分流量

在两个修订版本之间划分流量:

=== "kn"
    Run the command:
    ```bash
    kn service update hello \
    --traffic hello-00001=50 \
    --traffic @latest=50
    ```

=== "YAML"
    1. Add the `traffic` section to the bottom of your existing `hello.yaml` file:
        ``` yaml
        apiVersion: serving.knative.dev/v1
        kind: Service
        metadata:
          name: hello
        spec:
          template:
            spec:
              containers:
                - image: gcr.io/knative-samples/helloworld-go
                  ports:
                    - containerPort: 8080
                  env:
                    - name: TARGET
                      value: "Knative"
          traffic:
          - latestRevision: true
            percent: 50
          - latestRevision: false
            percent: 50
            revisionName: hello-00001
        ```
    1. Apply the YAML by running the command:
        ``` bash
        kubectl apply -f hello.yaml
        ```

!!! info
    `@latest` always points to the "latest" Revision, which in this case is `hello-00002`.

## 验证流量分流

若要验证流量分割的配置是否正确，请再次通过执行该命令列出修改项:

```bash
kn revisions list
```

!!! Success "Expected output"
    ```{ .bash .no-copy }
    NAME            SERVICE   TRAFFIC   TAGS   GENERATION   AGE   CONDITIONS   READY   REASON
    hello-00002     hello     50%              2            10m   3 OK / 4     True
    hello-00001     hello     50%              1            36m   3 OK / 4     True
    ```

在浏览器中多次访问Knative Service，以查看每个Revision提供的不同输出。

类似地，您可以从终端多次访问Service URL，以查看在修订之间划分的流量。

```bash
echo "Accessing URL $(kn service describe hello -o url)"
curl "$(kn service describe hello -o url)"
```

!!! Success "Expected output"
    ```{ .bash .no-copy }
    Hello Knative!
    Hello World!
    Hello Knative!
    Hello World!
    ```
