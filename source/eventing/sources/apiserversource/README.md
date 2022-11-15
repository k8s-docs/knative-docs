# 关于 ApiServerSource

![stage](https://img.shields.io/badge/Stage-stable-green?style=flat-square)
![version](https://img.shields.io/badge/API_Version-v1-green?style=flat-square)

API服务器源是Knative事件Kubernetes自定义资源，它监听Kubernetes API服务器发出的事件(如pod创建、部署更新等)，并将它们作为CloudEvents转发到一个接收器。

API服务器源是Knative事件处理核心组件的一部分，在安装Knative事件处理时默认提供。
用户可以创建ApiServerSource对象的多个实例。