<!-- Snippet used in the following topics:
- /docs/concepts/eventing-resources/brokers.md
- /docs/eventing/broker/README.md
-->

代理是Kubernetes的自定义资源，它定义了用于收集事件池的事件网格。
代理为事件输入提供可发现的端点，并使用触发器进行事件传递。
事件生产者可以通过发布事件将事件发送到代理。

![Source 1 and Source 2 are transmitting some data -- ones and twos -- to the Broker, which then gets filtered by Triggers to the desired Sink.](https://user-images.githubusercontent.com/16281246/116248768-1fe56080-a73a-11eb-9a85-8bdccb82d16c.png){draggable=false}
