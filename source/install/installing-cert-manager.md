# 为TLS证书安装证书管理器

安装[Cert-Manager](https://github.com/jetstack/cert-manager)工具获取TLS证书，可用于Knative中的安全HTTPS连接。
有关在Knative中启用HTTPS连接的更多信息，请参见[使用TLS证书配置HTTPS](../serving/using-a-tls-cert.md)。

您可以使用cert-manager手动获取证书，也可以启用Knative自动发放证书。
关于自动证书发放的完整说明在[启用自动TLS证书发放](../serving/using-auto-tls.md)中提供。

无论您是手动获取证书，还是配置Knative以实现自动发放，都可以使用以下步骤安装cert-manager。

## 在开始之前

为Knative安装cert-manager需要满足以下要求:

- 必须安装Knative服务。关于安装服务组件的详细信息，请参见[Knative安装指南](yaml-install/serving/install-serving-with-yaml.md)。
- 您必须配置您的Knative集群以使用[自定义域](../serving/using-a-custom-domain.md).
- Knative目前支持`1.0.0`及更高版本的证书管理器。

## 下载并安装cert-manager

要下载和安装cert-manager，请从官方的`cert-manager`网站上遵循[安装步骤](https://cert-manager.io/docs/installation/kubernetes/)。

## 正在完成TLS支持的Knative配置

在使用TLS证书进行安全连接之前，必须完成Knative的配置:

- **手动**: 如果您安装了cert-manager 手动获取证书，请继续阅读以下主题了解如何创建 Kubernetes secret: [手动添加TLS证书](../serving/using-a-tls-cert.md#manually-adding-a-tls-certificate)

- **自动**: 如果安装了用于自动证书发放的cert-manager，请继续执行以下主题以启用该功能: [在Knative中启用TLS证书自动发放](../serving/using-auto-tls.md)
