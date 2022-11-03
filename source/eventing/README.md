# Knative 事件

--8<-- "about-eventing.md"

支持的Knative事件用例示例:

- 在不创建消费者的情况下发布事件。您可以将事件作为HTTP POST发送到代理，并使用绑定将目标配置与生成事件的应用程序分离开来。

- 在不创建发布者的情况下使用事件。可以使用触发器根据事件属性使用来自代理的事件。应用程序以HTTP POST的形式接收事件。

!!! tip
    多个事件生产者和接收器可以一起使用，以创建更高级的[Knative事件](flows/README.md)流，以解决复杂的用例。

## 事件示例

:material-file-document: [创建并响应Kubernetes API事件](../eventing/sources/apiserversource/README.md){target=blank}

--8<-- "YouTube_icon.svg"
[创建一个图像处理管道](https://www.youtube.com/watch?v=DrmOpjAunlQ){target=blank}

--8<-- "YouTube_icon.svg"
[在大规模、无人机驱动的可持续农业项目中促进AI工作](https://www.youtube.com/watch?v=lVfJ5WEQ5_s){target=blank}

## 下一步

- 您可以使用[安装页面](../install/README.md)中列出的方法安装Knative事件处理.
