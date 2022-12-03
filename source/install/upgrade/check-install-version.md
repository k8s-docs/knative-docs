# 检测 Knative 版本

要检查Knative安装的版本，可以使用以下命令之一，具体取决于您是用YAML还是用Operator安装Knative。

## 如果你使用 YAML 安装

要验证在集群上运行的Knative组件的版本，请查询`<component>.knative.dev/release` 标签。

=== "Knative Serving"
    运行命令查看已安装的Knative服务版本:

    ```bash
    {% raw %}
    kubectl get namespace knative-serving -o 'go-template={{index .metadata.labels "serving.knative.dev/release"}}'
    {% endraw %}
    ```

    示例输出:

    ```{ .bash .no-copy }
    v0.23.0
    ```


=== "Knative Eventing"
    运行以下命令检查安装的Knative event版本:

    ```bash
    {% raw %}
    kubectl get namespace knative-eventing -o 'go-template={{index .metadata.labels "eventing.knative.dev/release"}}'
    {% endraw %}
    ```

    示例输出:

    ```{ .bash .no-copy }
    v0.23.0
    ```

## 如果你使用 Operator 安装

要验证您当前安装的Knative版本:

=== "Knative Serving"
    运行命令查看已安装的Knative服务版本:

    ```bash
    kubectl get KnativeServing knative-serving --namespace knative-serving
    ```

    示例输出:

    ```{ .bash .no-copy }
    NAME              VERSION         READY   REASON
    knative-serving   0.23.0          True
    ```

=== "Knative Eventing"
    运行以下命令检查安装的Knative event版本:

    ```bash
    kubectl get KnativeEventing knative-eventing --namespace knative-eventing
    ```

    示例输出:

    ```{ .bash .no-copy }
    NAME               VERSION         READY   REASON
    knative-eventing   0.23.0          True
    ```
