# 关于 ApiServerSource

![stage](https://img.shields.io/badge/Stage-stable-green?style=flat-square)
![version](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

API 服务器源是 Knative 事件 Kubernetes 自定义资源，它监听 Kubernetes API 服务器发出的事件(如 Pod 创建、部署更新等)，并将它们作为 CloudEvents 转发到一个接收器。

API 服务器源是 Knative 事件处理核心组件的一部分，在安装 Knative 事件处理时默认提供。
用户可以创建 ApiServerSource 对象的多个实例。
