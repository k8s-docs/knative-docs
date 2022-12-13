# Knative 事件度量

管理员可以查看Knative事件组件的度量。

## Broker - Ingress

使用以下度量来调试代理入接口的执行方式以及通过入接口组件分派的事件。

通过在http代码上聚合度量，可以将事件分为两类:成功事件(2xx)和失败事件(5xx)。

| Metric Name              | Description                                      | Type      | Tags                                                                                               | Unit          | Status |
| :----------------------- | :----------------------------------------------- | :-------- | :------------------------------------------------------------------------------------------------- | :------------ | :----- |
| event_count              | Number of events received by a Broker            | Counter   | broker_name<br>event_type<br>namespace_name<br>response_code<br>response_code_class<br>unique_name | Dimensionless | Stable |
| event_dispatch_latencies | The time spent dispatching an event to a Channel | Histogram | broker_name<br>event_type<br>namespace_name<br>response_code<br>response_code_class<br>unique_name | Milliseconds  | Stable |

## Broker - Filter

使用以下指标来调试代理筛选器如何执行以及通过筛选器组件分派什么事件。
此外，用户还可以测量事件上实际过滤操作的延迟时间。
通过在http代码上聚合度量，可以将事件分为两类:成功事件(2xx)和失败事件(5xx)。

| Metric Name                | Description                                                                        | Type      | Tags                                                                                                                                   | Unit          | Status |
| :------------------------- | :--------------------------------------------------------------------------------- | :-------- | :------------------------------------------------------------------------------------------------------------------------------------- | :------------ | :----- |
| event_count                | Number of events received by a Broker                                              | Counter   | broker_name<br>container_name=<br>filter_type<br>namespace_name<br>response_code<br>response_code_class<br>trigger_name<br>unique_name | Dimensionless | Stable |
| event_dispatch_latencies   | The time spent dispatching an event to a Channel                                   | Histogram | broker_name<br>container_name<br>filter_type<br>namespace_name<br>response_code<br>response_code_class<br>trigger_name<br>unique_name  | Milliseconds  | Stable |
| event_processing_latencies | The time spent processing an event before it is dispatched to a Trigger subscriber | Histogram | broker_name<br>container_name<br>filter_type<br>namespace_name<br>trigger_name<br>unique_name                                          | Milliseconds  | Stable |

## In-memory Dispatcher

内存通道可以通过以下指标进行评估。
通过在http代码上聚合度量，可以将事件分为两类:成功事件(2xx)和失败事件(5xx)。

| Metric Name              | Description                                                  | Type      | Tags                                                                                                    | Unit          | Status |
| :----------------------- | :----------------------------------------------------------- | :-------- | :------------------------------------------------------------------------------------------------------ | :------------ | :----- |
| event_count              | Number of events dispatched by the in-memory channel         | Counter   | container_name<br>event_type=<br>namespace_name=<br>response_code<br>response_code_class<br>unique_name | Dimensionless | Stable |
| event_dispatch_latencies | The time spent dispatching an event from a in-memory Channel | Histogram | container_name<br>event_type<br>namespace_name=<br>response_code<br>response_code_class<br>unique_name  | Milliseconds  | Stable |

!!! note
    A number of metrics eg. controller, Go runtime and others are omitted here as they are common
    across most components. For more about these metrics check the
    [Serving metrics API section](../../../serving/observability/metrics/serving-metrics.md).

## 事件源

事件源由拥有相关系统的用户创建，因此它们可以用事件触发应用程序。
每个源在默认情况下都公开了一些度量，以帮助用户监视分派的事件。
使用以下度量来验证事件已经从源端交付，从而验证源和与源的任何连接是否正常工作。

| Metric Name       | Description                                    | Type    | Tags                                                                                                                                                 | Unit          | Status |
| :---------------- | :--------------------------------------------- | :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------- | :------------ | :----- |
| event_count       | Number of events sent by the source            | Counter | event_source<br>event_type<br>name<br>namespace_name<br>resource_group<br>response_code<br>response_code_class<br>response_error<br>response_timeout | Dimensionless | Stable |
| retry_event_count | Number of events sent by the source in retries | Counter | event_source<br>event_type<br>name<br>namespace_name<br>resource_group<br>response_code<br>response_code_class<br>response_error<br>response_timeout | Dimensionless | Stable |
