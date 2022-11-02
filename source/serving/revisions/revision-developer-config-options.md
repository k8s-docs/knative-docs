# 开发人员配置选项

虽然在不修改 Knative 服务配置的情况下无法手动创建修订，但您可以修改现有修订的规范以更改其行为。

## 垃圾收集

--8<-- "about-revisions-garbage-collection.md"

### 为修订版禁用垃圾收集

你可以通过添加 `serving.knative.dev/no-gc: "true"` 注释来配置 Revision，使它永远不会被垃圾回收:

```yaml
apiVersion: serving.knative.dev/v1
kind: Revision
metadata:
  annotations:
    serving.knative.dev/no-gc: "true"
```
