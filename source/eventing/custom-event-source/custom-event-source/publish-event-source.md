# 将事件源发布到集群

1. 启动一个minikube集群:

    ```sh
    minikube start
    ```

1. 设置 `ko` 来使用minikube docker实例和本地注册表:

    ```sh
    eval $(minikube docker-env)
    export KO_DOCKER_REPO=ko.local
    ```

1. 应用CRD并配置YAML:

    ```sh
    ko apply -f config
    ```

1. 一旦`sample-source-controller-manager` 在`knative-samples`命名空间中运行，您可以应用`example.yaml`每隔`10s`将我们的`sample-source`直接连接到`ksvc`。

    ```yaml
    apiVersion: serving.knative.dev/v1
    kind: Service
    metadata:
      name: event-display
      namespace: knative-samples
    spec:
      template:
        spec:
          containers:
            - image: gcr.io/knative-releases/knative.dev/eventing/cmd/event_display
    ---
    apiVersion: samples.knative.dev/v1alpha1
    kind: SampleSource
    metadata:
      name: sample-source
      namespace: knative-samples
    spec:
      interval: "10s"
      sink:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: event-display
    ```

    ```sh
    ko apply -f example.yaml
    ```

1. 协调之后，您可以确认`ksvc`每`10s`输出一次有效的云事件，以与我们指定的间隔保持一致。

    ```sh
    % kubectl -n knative-samples logs -l serving.knative.dev/service=event-display -c user-container -f
    ```

    ```
    ☁️  cloudevents.Event
    Validation: valid
    Context Attributes,
      specversion: 1.0
      type: dev.knative.sample
      source: http://sample.knative.dev/heartbeat-source
      id: d4619592-363e-4a41-82d1-b1586c390e24
      time: 2019-12-17T01:31:10.795588888Z
      datacontenttype: application/json
    Data,
      {
        "Sequence": 0,
        "Heartbeat": "10s"
      }
    ☁️  cloudevents.Event
    Validation: valid
    Context Attributes,
      specversion: 1.0
      type: dev.knative.sample
      source: http://sample.knative.dev/heartbeat-source
      id: db2edad0-06bc-4234-b9e1-7ea3955841d6
      time: 2019-12-17T01:31:20.825969504Z
      datacontenttype: application/json
    Data,
      {
        "Sequence": 1,
        "Heartbeat": "10s"
      }
    ```
