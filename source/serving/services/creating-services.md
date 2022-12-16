# 创建服务

您可以通过应用YAML文件或使用 `kn service create` CLI命令来创建Knative服务。

## 先决条件

要创建Knative服务，您需要:

* 安装了Knative服务的Kubernetes集群。有关更多信息，请参见[安装Knative服务](../../install/yaml-install/serving/install-serving-with-yaml.md).
* 可选:要使用`kn service create`命令，必须[安装 `kn` CLI](../../client/configure-kn.md).

## 过程

!!! tip

    下面的命令创建一个`helloworld-go`示例服务。
    您可以修改这些命令，包括容器映像URL，以将您自己的应用程序部署为Knative服务。

创建一个示例服务:

=== "Apply YAML"

    1. Create a YAML file using the following example:

        ```yaml
        apiVersion: serving.knative.dev/v1
        kind: Service
        metadata:
          name: helloworld-go
          namespace: default
        spec:
          template:
            spec:
              containers:
                - image: gcr.io/knative-samples/helloworld-go
                  env:
                    - name: TARGET
                      value: "Go Sample v1"
        ```

    1. Apply the YAML file by running the command:

        ```bash
        kubectl apply -f <filename>.yaml
        ```
        Where `<filename>` is the name of the file you created in the previous step.

=== "kn CLI"

    ```
    kn service create helloworld-go --image gcr.io/knative-samples/helloworld-go
    ```

创建服务后，Knative执行以下任务:

* 为这个版本的应用程序创建一个新的不可变修订。
* 执行网络编程，为应用程序创建路由、入口、服务和负载均衡器。
* 根据流量自动缩放您的`pods`，包括零活动`pods`。
