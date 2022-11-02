# 管理员配置选项

如果您对 Knative 安装具有集群管理员权限，则可以修改 ConfigMaps 以更改集群上 Knative Services 的 revision 的全局默认配置选项。

## 垃圾收集

--8<-- "about-revisions-garbage-collection.md"

您可以通过修改 `config-gc` ConfigMap 来设置集群范围的垃圾收集配置。

可以修改以下垃圾收集设置:

| Name                            | Description                                                      |
| ------------------------------- | ---------------------------------------------------------------- |
| `retain-since-create-time`      | 在考虑进行垃圾收集之前，从创建修订开始必须经过的时间。           |
| `retain-since-last-active-time` | 在考虑进行垃圾收集之前，从修订最后一次活动到修订必须经过的时间。 |
| `min-non-active-revisions`      | 要保留的非激活修订的最小数量。                                   |
| `max-non-active-revisions`      | 要保留的未激活修订的最大数量。                                   |

如果修订版属于下列任何一类，则始终保留:

- 修订是活动的，并由路由引用。
- 在 `retain-since-create-time` 设置指定的时间内创建修订。
- 在 `retain-since-last-active-time` 设置指定的时间内，路由最后一次引用修订。
- 现有修订的数量少于 `min-non-active-revisions` 设置所指定的数量。

### 例子

- 立即清理任何不活跃的修订:

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: config-gc
    namespace: knative-serving
  data:
    min-non-active-revisions: "0"
    max-non-active-revisions: "0"
    retain-since-create-time: "disabled"
    retain-since-last-active-time: "disabled"
  ```

- 保留最后 10 个未激活的修订:

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: config-gc
    namespace: knative-serving
  data:
    retain-since-create-time: "disabled"
    retain-since-last-active-time: "disabled"
    max-non-active-revisions: "10"
  ```

- 在集群上禁用垃圾收集:

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: config-gc
    namespace: knative-serving
  data:
    retain-since-create-time: "disabled"
    retain-since-last-active-time: "disabled"
    max-non-active-revisions: "disabled"
  ```
