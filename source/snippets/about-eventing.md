<!-- Snippet used in the following topics:
- /docs/eventing/README.md
- /docs/concepts/README.md
-->

Knative 事件是一个 API 集合，它使您能够在应用程序中使用[事件驱动的体系结构](https://en.wikipedia.org/wiki/Event-driven_architecture){target=_blank}。
可以使用这些 API 创建将事件从事件生产者路由到事件消费者(称为接收事件的接收器)的组件。
还可以将接收器配置为通过发送响应事件来响应 HTTP 请求。

Knative 事件使用标准的 HTTP POST 请求在事件生产者和接收器之间发送和接收事件。
这些事件符合[CloudEvents 规范](https://cloudevents.io/){target=_blank}，该规范支持在任何编程语言中创建、解析、发送和接收事件。

Knative 事件组件是松散耦合的，可以彼此独立地开发和部署。
任何生产者都可以在有活动事件消费者监听这些事件之前生成事件。
在生产者创建这些事件之前，任何事件消费者都可以表达对一类事件的兴趣。
