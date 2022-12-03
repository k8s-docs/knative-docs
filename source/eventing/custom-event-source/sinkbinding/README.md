# SinkBinding

![API版本v1](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

SinkBinding对象支持将事件产生与交付寻址分离。

可以使用接收器绑定将主题指向接收器。
_subject_ 是一个Kubernetes资源，它嵌入PodSpec模板并产生事件。
_sink_ 是一个可以接收事件的可寻址Kubernetes对象。

SinkBinding对象将环境变量注入到接收器的PodTemplateSpec中。
因此，应用程序代码不需要直接与Kubernetes API交互来定位事件目的地。
这些环境变量如下:

- `K_SINK` - 解析接收器的URL。
- `K_CE_OVERRIDES` - 指定对出站事件的覆盖的JSON对象。
