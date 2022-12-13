# 配置事件源默认值

本主题介绍如何配置Knative事件源的默认值。您可以根据事件源生成事件的方式来配置事件源。

## 为PingSource配置默认值

PingSource是一个事件源，它在指定的cron计划上生成具有固定有效负载的事件。
关于如何创建一个新的PingSource，请参见[创建一个PingSource对象](../../event/sources/ping-source/README.md)。
可用的参数请参见[PingSource reference](../../event/sources/ping-source/reference.md)。

除了可以在PingSource资源中配置的参数外，还有一个全局ConfigMap称为 `config-ping-defaults`。
这个ConfigMap允许您更改PingSource添加到它生成的CloudEvents中的最大数据量。

`data-max-size`参数允许您设置消息允许发送的最大字节数，不包括任何base64解码。
默认值 `-1`不设置数据限制。

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-ping-defaults
  namespace: knative-eventing
data:
  data-max-size: -1
```



```
kubectl edit cm config-ping-defaults -n knative-eventing
```
