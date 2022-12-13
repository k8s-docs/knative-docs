# 使用Knative示例存储库

Knative项目提供了一个[示例存储库](https://github.com/knative-sandbox/sample-source)，其中包含一个用于基本事件源控制器和接收适配器的模板。

## 先决条件

- 您熟悉Kubernetes和Go开发。
- 已安装Git。
- 已经安装了Go。
- 您已经克隆了[`sample-source`存储库](https://github.com/knative-sandbox/sample-source).

可选:

- 安装 [`ko`](https://github.com/google/ko/) CLI 工具.
- 安装 [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/) CLI 工具.
- 配置 [minikube](https://github.com/kubernetes/minikube) 或 [kind](https://kind.sigs.k8s.io/).

## 示例文件概述

### 接收器适配器文件

- `cmd/receive_adapter/main.go` - Translates resource variables to the underlying adapter structure, so that they can be passed into the Knative system.

- `pkg/adapter/adapter.go` - The functions that support receiver adapter translation of events to CloudEvents.

### 控制器文件

- `cmd/controller/main.go` - Passes the event source's `NewController` implementation to the shared `main` method.

- `pkg/reconciler/sample/controller.go` - The `NewController` implementation that is passed to the shared `main` method.

### CRD 文件

- `pkg/apis/samples/VERSION/samplesource_types.go` - The schema for the underlying API types, which provide the variables to be defined in the resource YAML file.

### 调解人的文件

- `pkg/reconciler/sample/samplesource.go` - 接收适配器的调节函数。

- `pkg/apis/samples/VERSION/samplesource_lifecycle.go` - 包含事件源的协调细节的状态信息:
    - 源准备好了
    - 接收器提供
    - 部署
    - EventType提供
    - Kubernetes资源正确

<!-- TODO: Add definitions / explanations for these statuses-->

## 过程

1. 在其中定义资源模式中所需的类型 `pkg/apis/samples/v1alpha1/samplesource_types.go`.

    这包括资源YAML中所需的字段，以及使用源的客户端集和API在控制器中引用的字段:

    ```go
    // +genclient
    // +genreconciler
    // +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object
    // +k8s:openapi-gen=true
    type SampleSource struct {
    	metav1.TypeMeta `json:",inline"`
    	// +optional
    	metav1.ObjectMeta `json:"metadata,omitempty"`

    	// Spec holds the desired state of the SampleSource (from the client).
    	Spec SampleSourceSpec `json:"spec"`

    	// Status communicates the observed state of the SampleSource (from the controller).
    	// +optional
    	Status SampleSourceStatus `json:"status,omitempty"`
    }

    // SampleSourceSpec holds the desired state of the SampleSource (from the client).
    type SampleSourceSpec struct {
    	// inherits duck/v1 SourceSpec, which currently provides:
        // * Sink - a reference to an object that will resolve to a domain name or
        //   a URI directly to use as the sink.
        // * CloudEventOverrides - defines overrides to control the output format
        //   and modifications of the event sent to the sink.
        duckv1.SourceSpec `json:",inline"`

        // ServiceAccountName holds the name of the Kubernetes service account
        // as which the underlying K8s resources should be run. If unspecified
        // this will default to the "default" service account for the namespace
        // in which the SampleSource exists.
        // +optional
        ServiceAccountName string `json:"serviceAccountName,omitempty"`

        // Interval is the time interval between events.
        //
        // The string format is a sequence of decimal numbers, each with optional
        // fraction and a unit suffix, such as "300ms", "-1.5h" or "2h45m". Valid time
        // units are "ns", "us" (or "µs"), "ms", "s", "m", "h". If unspecified
        // this will default to "10s".
        Interval string `json:"interval"`
    }

    // SampleSourceStatus communicates the observed state of the SampleSource (from the controller).
    type SampleSourceStatus struct {
    	duckv1.Status `json:",inline"`

    	// SinkURI is the current active sink URI that has been configured
    	// for the SampleSource.
    	// +optional
    	SinkURI *apis.URL `json:"sinkUri,omitempty"`
    }
    ```

1. 定义在 `status` 和 `SinkURI` 字段中反映的生命周期:

    ```go
    const (
    	// SampleConditionReady has status True when the SampleSource is ready to send events.
    	SampleConditionReady = apis.ConditionReady
        // ...
    )
    ```

1. 通过定义要从协调器函数调用的函数来设置生命周期条件。这通常是在里面完成的 `pkg/apis/samples/VERSION/samplesource_lifecycle.go`:

    ```go
    // InitializeConditions sets relevant unset conditions to Unknown state.
    func (s *SampleSourceStatus) InitializeConditions() {
    	SampleCondSet.Manage(s).InitializeConditions()
    }

    ...

    // MarkSink sets the condition that the source has a sink configured.
    func (s *SampleSourceStatus) MarkSink(uri *apis.URL) {
    	s.SinkURI = uri
    	if len(uri.String()) > 0 {
    		SampleCondSet.Manage(s).MarkTrue(SampleConditionSinkProvided)
    	} else {
    		SampleCondSet.Manage(s).MarkUnknown(SampleConditionSinkProvided, "SinkEmpty", "Sink has resolved to empty.%s", "")
    	}
    }

    // MarkNoSink sets the condition that the source does not have a sink configured.
    func (s *SampleSourceStatus) MarkNoSink(reason, messageFormat string, messageA ...interface{}) {
    	SampleCondSet.Manage(s).MarkFalse(SampleConditionSinkProvided, reason, messageFormat, messageA...)
    }
    ```
