# 在Knative中收集度量

Knative支持收集指标的不同流行工具:

- [Prometheus](https://prometheus.io/)
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)

[Grafana](https://grafana.com/oss/) 仪表板可用于直接从Prometheus收集的指标。

您还可以设置OpenTelemetry Collector，以便从Knative组件接收度量，并将它们分发给支持OpenTelemetry的其他度量提供程序。

!!! warning
    您不能同时使用OpenTelemetry Collector和Prometheus。
    默认的度量后端是Prometheus。
    你需要从`config-observability` Configmap中删除`metrics.backend-destination` 和 `metrics.request-metrics-backend-destination`键来启用Prometheus度量。

## 关于 Prometheus

[Prometheus](https://prometheus.io/)是一个用于收集、聚合时间序列度量和警报的开源工具。
它还可以用于刮除OpenTelemetry Collector，下面将在使用Prometheus时演示这一点。

## 配置 Prometheus

1. Install the [Prometheus Operator](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) by using [Helm](https://helm.sh/docs/intro/using_helm/):

       ```bash
       helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
       helm repo update
       helm install prometheus prometheus-community/kube-prometheus-stack -n default -f values.yaml
       # values.yaml contains at minimum the configuration below
       ```

    !!! caution
        You will need to ensure that the helm chart has following values configured, otherwise the ServiceMonitors/Podmonitors will not work.
        ```yaml
        kube-state-metrics:
          metricLabelsAllowlist:
            - pods=[*]
            - deployments=[app.kubernetes.io/name,app.kubernetes.io/component,app.kubernetes.io/instance]
        prometheus:
          prometheusSpec:
            serviceMonitorSelectorNilUsesHelmValues: false
            podMonitorSelectorNilUsesHelmValues: false
        ```

1. Apply the ServiceMonitors/PodMonitors to collect metrics from Knative.

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/knative-sandbox/monitoring/main/servicemonitor.yaml
    ```

1. Grafana dashboards can be imported from the [`knative-sandbox` repository](https://github.com/knative-sandbox/monitoring/tree/main/grafana).

1. If you are using the Grafana Helm Chart with the Dashboard Sidecar enabled, you can load the dashboards by applying the following configmaps.

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/knative-sandbox/monitoring/main/grafana/dashboards.yaml
    ```

### 本地访问Prometheus实例

By default, the Prometheus instance is only exposed on a private service named `prometheus-operated`.

To access the console in your web browser:

1. Enter the command:

    ```bash
    kubectl port-forward -n default svc/prometheus-operated 9090
    ```

1. Access the console in your browser via `http://localhost:9090`.

## 关于 OpenTelemetry

OpenTelemetry是一个针对云原生软件的CNCF可观察性框架，它提供了一组工具、api和sdk。

您可以使用OpenTelemetry来测量、生成、收集和导出遥测数据。
这些数据包括度量、日志和跟踪，您可以通过分析这些数据来了解Knative组件的性能和行为。

OpenTelemetry允许您轻松地将指标导出到多个监视服务，而不需要重新构建或重新配置Knative二进制文件。

## 理解收集器

收集器提供了一个位置，各种Knative组件可以在其中推送由监视服务保留和收集的指标。

在下面的示例中，您可以使用ConfigMap和Deployment配置单个收集器实例。

!!! tip
    对于更复杂的部署，您可以通过使用[OpenTelemetry Operator](https://github.com/open-telemetry/opentelemetry-operator)自动化这些步骤中的一些。

!!! caution
    https://github.com/knative-sandbox/monitoring/tree/main/grafana上的Grafana仪表板不能使用从OpenTelemetry Collector中提取的指标。

![Diagram of components reporting to collector, which is scraped by Prometheus](system-diagram.svg)

<!-- yuml.me UML rendering of:
[queue-proxy1]->[Collector]
[queue-proxy2]->[Collector]
[autoscaler]->[Collector]
[controller]->[Collector]
[Collector]<-scrape[Prometheus]
-->

## 设置收集器

1. Create a namespace for the collector to run in, by entering the following command:

       ```bash
       kubectl create namespace metrics
       ```
    The next step uses the `metrics` namespace for creating the collector.

1. Create a Deployment, Service, and ConfigMap for the collector by entering the following command:

       ```bash
       kubectl apply -f https://raw.githubusercontent.com/knative/docs/main/docs/serving/observability/metrics/collector.yaml
       ```

1. Update the `config-observability` ConfigMaps in the Knative Serving and
   Eventing namespaces, by entering the follow command:

       ```bash
       kubectl patch --namespace knative-serving configmap/config-observability \
         --type merge \
         --patch '{"data":{"metrics.backend-destination":"opencensus","metrics.request-metrics-backend-destination":"opencensus","metrics.opencensus-address":"otel-collector.metrics:55678"}}'
       kubectl patch --namespace knative-eventing configmap/config-observability \
         --type merge \
         --patch '{"data":{"metrics.backend-destination":"opencensus","metrics.opencensus-address":"otel-collector.metrics:55678"}}'
       ```

## 验证收集器设置

1. You can check that metrics are being forwarded by loading the Prometheus export port on the collector, by entering the following command:

    ```bash
    kubectl port-forward --namespace metrics deployment/otel-collector 8889
    ```

1. Fetch `http://localhost:8889/metrics` to see the exported metrics.
