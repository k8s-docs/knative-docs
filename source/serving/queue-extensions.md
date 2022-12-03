# 用QPOptions扩展队列代理映像

Knative服务pods包括两个容器:

- 用户主服务容器，命名为`user-container`
- 队列代理 - 一个名为 `queue-proxy` 的sidecar，在`user-container`前面充当反向代理。

可以扩展队列代理以提供其他功能。队列代理的QPOptions特性允许附加的运行时包扩展队列代理功能。

For example, the [security-guard](https://knative.dev/security-guard#section-readme) repository provides an extension that uses the QPOptions feature. The [QPOption](https://knative.dev/security-guard/pkg/qpoption#section-readme) package enables users to add additional security features to Queue Proxy.

可用的运行时特性在构建队列代理映像时确定。
队列代理定义了激活和配置扩展的有序方式。

## 额外的信息

- [Enabling Queue Proxy Pod Info](./configuration/feature-flags.md#queue-proxy-pod-info) - discussing a necessary step to enable the use of extensions.
- [Using extensions enabled by QPOptions](./services/using-queue-extensions.md) - discussing how to configure a service to use features implemented in extensions.

## 添加扩展

You can add extensions by replacing the `cmd/queue/main.go` file before the Queue Proxy image is built. The following example shows a `cmd/queue/main.go` file that adds the `test-gate` extension:

```go
  package main

  import "os"

  import "knative.dev/serving/pkg/queue/sharedmain"
  import "knative.dev/security-guard/pkg/qpoption"
  import _ "knative.dev/security-guard/pkg/test-gate"

  func main() {
      qOpt := qpoption.NewQPSecurityPlugs()
      defer qOpt.Shutdown()

        if sharedmain.Main(qOpt.Setup) != nil {
          os.Exit(1)
      }
  }
```
