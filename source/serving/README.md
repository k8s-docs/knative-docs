# Knative 服务

--8<-- "about-serving.md"

## 常见用例

支持的Knative服务用例示例:

- 快速部署无服务器容器。
- 自动缩放，包括将豆荚缩放到零。
- 支持多个网络层，如Contour、Kourier和Istio，以便集成到现有环境中。

Knative服务同时支持HTTP和[HTTPS](using-a-tls-cert.md)网络协议。

## 安装

您可以通过[安装页面](../install/README.md)中列出的方法安装Knative服务.

## 开始

要开始使用服务，请查看[hello world](../samples/serving.md)示例项目之一。
这些项目使用`Service`资源，它为你管理所有的细节。

使用`Service`资源，部署的服务将自动创建匹配的路由和配置。
每次`Service`更新时，都会创建一个新的修订。

## 更多示例和演示

- [Knative服务代码示例](../samples/serving.md)

## 调试Knative服务问题

- [调试应用程序问题](troubleshooting/debugging-application-issues.md)

## 配置和网络

- [配置集群本地路由](services/private-services.md)
- [使用自定义域](using-a-custom-domain.md)
- [交通管理](traffic-management.md)

## 可观察性

- [服务标准API](observability/metrics/serving-metrics.md)

## 已知的问题

有关已知问题的完整列表，请参见[Knative服务问题](https://github.com/knative/serving/issues)页面。