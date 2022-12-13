# 创建接收适配器

作为源协调过程的一部分，您必须创建和部署底层接收适配器。

接收适配器需要一个基于注入的`main`方法，它位于`cmd/receiver_adapter/main.go`中:

```go
// This Adapter generates events at a regular interval.
package main

import (
	"knative.dev/eventing/pkg/adapter"
	myadapter "knative.dev/sample-source/pkg/adapter"
)

func main() {
	adapter.Main("sample-source", myadapter.NewEnv, myadapter.NewAdapter)
}
```

接收适配器的`pkg`实现包含两个主要函数:

1. 一个`NewAdapter(ctx context.Context, aEnv adapter.EnvConfigAccessor, ceClient cloudevents.Client) adapter.Adapter {}`调用，它通过`EnvConfigAccessor`创建带有传递变量的新适配器。
   创建的适配器被传递给CloudEvents客户端(事件被转发到该客户端)。
   这在Knative生态系统中有时被称为sink或`ceClient`。
   返回值是由适配器的本地结构定义的对适配器的引用。

    在示例源的情况下:

    ```go
    // Adapter generates events at a regular interval.
    type Adapter struct {
    	logger   *zap.Logger
    	interval time.Duration
    	nextID   int
    	client   cloudevents.Client
    }
    ```

1. 一个`Start`函数，作为适配器`struct`的接口实现:

    ```go
    func (a *Adapter) Start(stopCh <-chan struct{}) error {
    ```

    `stopCh`是停止适配器的信号。否则，函数的作用是处理下一个事件。

    在`sample-source`的情况下，该函数创建一个CloudEvent，每隔X时间转发到指定的接收器，由`EnvConfigAccessor`参数指定，该参数由资源YAML加载:

    ```go
    func (a *Adapter) Start(stopCh <-chan struct{}) error {
        a.logger.Infow("Starting heartbeat", zap.String("interval", a.interval.String()))
    	for {
    		select {
    		case <-time.After(a.interval):
    			event := a.newEvent()
    			a.logger.Infow("Sending new event", zap.String("event", event.String()))
    			if result := a.client.Send(context.Background(), event); !cloudevents.IsACK(result) {
                    a.logger.Infow("failed to send event", zap.String("event", event.String()), zap.Error(result))
                    // We got an error but it could be transient, try again next interval.
                    continue
                }
    		case <-stopCh:
    			a.logger.Info("Shutting down...")
    			return nil
    		}
    	}
    }
    ```

## 管理控制器中的接收适配器

1. 更新`ObservedGeneration`并初始化`Status`条件，如在`samplesource_lifecycle.go`和`samplesource_types.go`文件中定义的:

    ```go
    src.Status.InitializeConditions()
    src.Status.ObservedGeneration = src.Generation
    ```

2. 创建一个接收适配器。

    1. 验证指定的Kubernetes资源是否有效，并相应地更新`Status`。

    2. 集合 `ReceiveAdapterArgs`:

        ```go
        raArgs := resources.ReceiveAdapterArgs{
        		EventSource:    src.Namespace + "/" + src.Name,
                Image:          r.ReceiveAdapterImage,
                Source:         src,
                Labels:         resources.Labels(src.Name),
                AdditionalEnvs: r.configAccessor.ToEnvVars(), // Grab config envs for tracing/logging/metrics
        	}
        ```

        !!! note
            确切的参数可能会根据功能需求而改变。
            根据提供的参数创建底层部署，根据需要匹配pod模板、标签、所有者引用等来填写部署。
            例子:[pkg/reconciler/sample/resources/receive_adapter.go](https://github.com/knative-sandbox/sample-source/blob/main/pkg/reconciler/sample/resources/receive_adapter.go)

    1. 获取现有的接收适配器部署:

        ```go
        namespace := owner.GetObjectMeta().GetNamespace()
        ra, err := r.KubeClientSet.AppsV1().Deployments(namespace).Get(expected.Name, metav1.GetOptions{})
        ```

    1. 如果没有现有的接收适配器部署，创建一个:

        ```go
        ra, err = r.KubeClientSet.AppsV1().Deployments(namespace).Create(expected)
        ```

    1. 检查预期的规范是否与现有的规范不同，并在需要时更新部署:

        ```go
        } else if r.podSpecImageSync(expected.Spec.Template.Spec, ra.Spec.Template.Spec) {
            ra.Spec.Template.Spec = expected.Spec.Template.Spec
            if ra, err = r.KubeClientSet.AppsV1().Deployments(namespace).Update(ra); err != nil {
                return ra, err
            }
        ```

    1. 如果有更新，请记录事件:

        ```go
        return pkgreconciler.NewEvent(corev1.EventTypeNormal, "DeploymentUpdated", "updated deployment: \"%s/%s\"", namespace, name)
        ```

    1. 如果成功，更新`Status` 和 `MarkDeployed`:

        ```go
        src.Status.PropagateDeploymentAvailability(ra)
        ```

1. 创建一个SinkBinding将接收适配器与接收器绑定。

    1. 为接收适配器部署创建一个`Reference`。这个部署是SinkBinding的源代码:

        ```go
        tracker.Reference{
            APIVersion: appsv1.SchemeGroupVersion.String(),
            Kind:       "Deployment",
            Namespace:  ra.Namespace,
            Name:       ra.Name,
        }
        ```

    1. 获取现有的SinkBinding:

        ```go
        namespace := owner.GetObjectMeta().GetNamespace()
        sb, err := r.EventingClientSet.SourcesV1alpha2().SinkBindings(namespace).Get(expected.Name, metav1.GetOptions{})
        ```

    1. 如果不存在SinkBinding，创建一个:

        ```go
        sb, err = r.EventingClientSet.SourcesV1alpha2().SinkBindings(namespace).Create(expected)
        ```

    1. 检查预期的规范是否与现有的规范不同，如果需要的话更新SinkBinding:

        ```go
        else if r.specChanged(sb.Spec, expected.Spec) {
            sb.Spec = expected.Spec
            if sb, err = r.EventingClientSet.SourcesV1alpha2().SinkBindings(namespace).Update(sb); err != nil {
                return sb, err
            }
        ```

    1. 如果有更新，请记录事件:

        ```go
        return pkgreconciler.NewEvent(corev1.EventTypeNormal, "SinkBindingUpdated", "updated SinkBinding: \"%s/%s\"", namespace, name)
        ```

    2. `MarkSink` 结果::

        ```go
        src.Status.MarkSink(sb.Status.SinkURI)
        ```

2. 返回一个新的协调器事件，声明流程已经完成:

    ```go
    return pkgreconciler.NewEvent(corev1.EventTypeNormal, "SampleSourceReconciled", "SampleSource reconciled: \"%s/%s\"", namespace, name)
    ```
