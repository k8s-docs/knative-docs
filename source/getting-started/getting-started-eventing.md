# 关于 Knative 事件

Knative事件通过轻松地将事件源、触发器和其他选项附加到Knative服务，为您提供了用于创建事件驱动应用程序的有用工具。

事件驱动的应用程序被设计为在事件发生时检测事件，并通过使用用户定义的事件处理过程处理这些事件。

!!! tip
    要了解更多关于事件驱动体系结构和Knative事件的信息，请查看CNCF关于[带有Knative事件的事件驱动体系结构](https://www.cncf.io/online-programs/event-driven-architecture-with-knative-events/){target=blank}的会议

安装Knative event之后，可以创建、发送和验证事件。
本教程展示了如何使用基本工作流来管理使用**事件源**、**代理**、**触发器**和**水槽**的事件。