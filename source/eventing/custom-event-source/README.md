# 自定义事件源

如果您需要从Knative中不包含的事件生成器输入事件，或者从Knative使用的CloudEvent格式之外的事件生成器输入事件，您可以使用以下方法之一来实现此目的:

- [创建一个自定义Knative事件源](custom-event-source/README.md).
- 通过创建[SinkBinding](sinkbinding/README.md)，使用PodSpecable对象作为事件源。
- 使用容器作为事件源，通过创建[ContainerSource](containersource/README.md).
