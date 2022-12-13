# 创建一个控制器

您可以使用示例存储库[`update-codegen.sh`](https://github.com/knative-sandbox/sample-source/blob/main/hack/update-codegen.sh)脚本来生成所需的组件(`clientset`, `cache`, `informers`, 和 `listers`) 并将其注入到自定义控制器中。

**控制器例子:**

```go
import (
    // ...
    sampleSourceClient "knative.dev/sample-source/pkg/client/injection/client"
    samplesourceinformer "knative.dev/sample-source/pkg/client/injection/informers/samples/v1alpha1/samplesource"
)
// ...
func NewController(ctx context.Context, cmw configmap.Watcher) *controller.Impl {
    sampleSourceInformer := samplesourceinformer.Get(ctx)
        r := &Reconciler{
        // ...
        samplesourceClientSet: sampleSourceClient.Get(ctx),
        samplesourceLister:    sampleSourceInformer.Lister(),
        // ...
}
```

## 过程

1. 运行命令生成组件:

    ```bash
    ${CODEGEN_PKG}/generate-groups.sh "deepcopy,client,informer,lister" \
      knative.dev/sample-source/pkg/client knative.dev/sample-source/pkg/apis \
      "samples:v1alpha1" \
      --go-header-file ${REPO_ROOT}/hack/boilerplate/boilerplate.go.txt
    ```

1. 注入组件的命令如下:

    ```bash
    # Injection
    ${KNATIVE_CODEGEN_PKG}/hack/generate-knative.sh "injection" \
      knative.dev/sample-source/pkg/client knative.dev/sample-source/pkg/apis \
      "samples:v1alpha1" \
      --go-header-file ${REPO_ROOT}/hack/boilerplate/boilerplate.go.txt
    ```

1. 将新的控制器实现传递给`sharedmain`方法:

    ```go
    import (
    	// The set of controllers this controller process runs.
    	"knative.dev/sample-source/pkg/reconciler/sample"

    	// This defines the shared main for injected controllers.
    	"knative.dev/pkg/injection/sharedmain"
    )

    func main() {
    	sharedmain.Main("sample-source-controller", sample.NewController)
    }
    ```

1. 定义 `NewController` 实现:

    ```go
    func NewController(
    	ctx context.Context,
    	cmw configmap.Watcher,
    ) *controller.Impl {
        // ...
    	deploymentInformer := deploymentinformer.Get(ctx)
    	sinkBindingInformer := sinkbindinginformer.Get(ctx)
    	sampleSourceInformer := samplesourceinformer.Get(ctx)

    	r := &Reconciler{
    	dr:  &reconciler.DeploymentReconciler{KubeClientSet: kubeclient.Get(ctx)},
    	sbr: &reconciler.SinkBindingReconciler{EventingClientSet: eventingclient.Get(ctx)},
    	// Config accessor takes care of tracing/config/logging config propagation to the receive adapter
    	configAccessor: reconcilersource.WatchConfigurations(ctx, "sample-source", cmw),
    }
    ```

    一个`configmap.Watcher`和一个上下文(注入的列表用于协调器结构参数)被传递给这个实现。

1. 从`knative.dev/pkg`依赖项导入基本协调器:

    ```go
    import (
        // ...
        reconcilersource "knative.dev/eventing/pkg/reconciler/source"
        // ...
    )
    ```

1. 确保事件处理程序被过滤到正确的信息提供者:

    ```go
        sampleSourceInformer.Informer().AddEventHandler(controller.HandleAll(impl.Enqueue))
    ```

1. 确保为示例源使用的辅助资源正确配置了通知器，以部署和绑定事件源和接收适配器:

    ```go
        deploymentInformer.Informer().AddEventHandler(cache.FilteringResourceEventHandler{
            FilterFunc: controller.FilterGroupKind(v1alpha1.Kind("SampleSource")),
            Handler:    controller.HandleAll(impl.EnqueueControllerOf),
        })

        sinkBindingInformer.Informer().AddEventHandler(cache.FilteringResourceEventHandler{
            FilterFunc: controller.FilterGroupKind(v1alpha1.Kind("SampleSource")),
            Handler:    controller.HandleAll(impl.EnqueueControllerOf),
        })
    ```
