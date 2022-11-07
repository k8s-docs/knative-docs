## 路由更新状态

在rollout过程中，系统会更新路由和Knative服务状态条件。
`traffic` and `conditions`状态参数都受到影响。

以以下流量配置为例:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
...
spec:
...
  traffic:
  - percent: 55
    configurationName: config # Pinned to latest ready Revision
  - percent: 45
    revisionName: config-00005 # Pinned to a specific Revision.
```

最初，1%的流量被推出到修订:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
...
spec:
...
  traffic:
  - percent: 54
    revisionName: config-00008
  - percent: 1
    revisionName: config-00009
  - percent: 45
    revisionName: config-00005 # Pinned to a specific Revision.
```

然后其余的流量以18%的增量推出:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
...
spec:
...
  traffic:
  - percent: 36
    revisionName: config-00008
  - percent: 19
    revisionName: config-00009
  - percent: 45
    revisionName: config-00005 # Pinned to a specific Revision.
```

rollout继续进行，直到达到目标流量配置:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
...
spec:
...
  traffic:
  - percent: 55
    revisionName: config-00009
  - percent: 45
    revisionName: config-00005 # Pinned to a specific Revision.
```

rollout过程中，路由和Knative服务状态条件如下:

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
...
spec:
...
status:
  conditions:
  ...
  - lastTransitionTime: "..."
    message: A gradual rollout of the latest revision(s) is in progress.
    reason: RolloutInProgress
    status: Unknown
    type: Ready
```
