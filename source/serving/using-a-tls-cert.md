# 配置HTTPS使用TLS证书

了解如何在Knative中使用TLS证书配置安全HTTPS连接([TLS取代SSL](https://en.wikipedia.org/wiki/Transport_Layer_Security)).
配置安全HTTPS连接，使您的Knative服务和路由能够[终止外部TLS连接](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_interception).
可以配置Knative来处理手动指定的证书，也可以启用Knative来自动获取和更新证书。

您可以使用[Certbot][cb]或[cert-manager][cm]两种方式获取证书。
这两个工具都支持TLS证书，但如果您想启用Knative来自动发放TLS证书，您必须安装和配置证书管理器工具:

- **手动获取和更新证书**: 可以使用Certbot和cert-manager工具手动获取TLS证书。
  通常，在您获得证书后，您必须创建一个Kubernetes秘密以在您的集群中使用该证书。
  有关手动获取和配置证书的详细信息，请参见本主题后面的步骤。

- **启用Knative自动获取和更新TLS证书**: 您还可以使用证书管理器配置Knative以自动获取新的TLS证书并更新现有的证书。
  如果您希望启用Knative自动发放TLS证书，请参见[启用TLS证书自动发放](using-auto-tls.md)主题。

默认情况下，[Let's Encrypt颁发机构(CA)][le]用于演示如何启用HTTPS连接，但是您可以配置Knative使用来自支持ACME协议的CA的任何证书。但是，您必须使用并配置您的证书颁发者使用[`DNS-01`挑战类型](https://letsencrypt.org/docs/challenge-types/#dns-01-challenge).

!!! warning
    Let's Encrypt颁发的证书有效期仅为[90天][le-faqs]。因此，如果您选择手动获取并配置证书，请确保每个证书都在过期前进行了更新。

[cm]: https://github.com/jetstack/cert-manager
[cm-docs]: https://cert-manager.readthedocs.io/en/latest/getting-started/
[cm-providers]:
  http://docs.cert-manager.io/en/latest/tasks/acme/configuring-dns01/index.html?highlight=supported%20DNS01%20providers#supported-dns01-providers
[le]: https://letsencrypt.org
[le-faqs]: https://letsencrypt.org/docs/faq/
[cb]: https://certbot.eff.org
[cb-docs]: https://certbot.eff.org/docs/install.html#certbot-auto
[cb-providers]: https://certbot.eff.org/docs/using.html#changing-the-acme-server
[cb-cli]: https://certbot.eff.org/docs/using.html#certbot-command-line-options

## 在开始之前

启用HTTPS安全连接需要满足以下要求:

- 必须安装Knative服务。关于安装服务组件的详细信息，请参见[Knative安装指南](../install/yaml-install/serving/install-serving-with-yaml.md)。
- 您必须配置您的Knative集群以使用[自定义域](using-a-custom-domain.md).

!!! warning
    Istio只支持每个Kubernetes集群一个证书。
    要使用Knative集群为多个域提供服务，必须确保为想要服务的每个域签署了新的或现有的证书。

## 获取TLS证书

如果您已经有了域的签名证书，请参见[手动添加TLS证书](#manually-adding-a-tls-certificate)了解有关配置Knative集群的详细信息。

如果您需要新的TLS证书，您可以选择使用以下工具之一从Let's Encrypt获取证书:

- 设置Certbot手动获取Let's Encrypt证书
- 将cert-manager设置为手动获取证书或自动提供证书

本页涵盖了这两个选项的详细信息。

有关使用其他CA的详细信息，请参见该工具的参考文档:

- [Certbot支持提供者][cb-providers]
- [证书管理器支持的提供者][cm-providers]

### 使用Certbot手动获取Let 's Encrypt证书

请按照以下步骤安装[Certbot][cb]，并使用该工具从Let's Encrypt手动获取TLS证书。

1. 按照[`certbot-auto`包装脚本][cb-docs]说明安装Certbot。

1. 使用Certbot在授权过程中通过DNS challenge请求证书:

     ```bash
     ./certbot-auto certonly --manual --preferred-challenges dns -d '*.default.yourdomain.com'
     ```

   其中`-d`指定你的域。如果你想验证多个域，你可以包含多个标志:   `-d MY.EXAMPLEDOMAIN.1 -d MY.EXAMPLEDOMAIN.2`. 
   有关更多信息，请参见[Cerbot命令行][cb-cli]参考。

   Certbot工具将引导您通过在这些域中创建TXT记录来验证您是否拥有指定的每个域。

   结果: CertBot创建两个文件:

   - Certificate:`fullchain.pem`
   - Private key: `privkey.pem`

接下来是什么:

通过[创建一个Kubernetes secret](#manually-adding-a-tls-certificate)将证书和私钥添加到您的Knative集群中.

### 使用证书管理器获取Let's Encrypt证书

您可以安装并使用[cert-manager][cm]手动获取证书，或者配置您的Knative集群以自动发放证书:

- **手动证书**: 安装cert-manager后，使用工具手动获取证书。

  使用cert-manager手动获取证书:

  1.  [安装和配置cert-manager](../install/installing-cert-manager.md).

  1.  通过创建和使用Kubernetes秘密，继续执行有关[手动添加TLS证书](#manually-adding-a-tls-certificate)的步骤。

- **自动证书**: 配置Knative使用证书管理器自动获取和更新TLS证书。为这种方法安装和配置证书管理器的步骤在[启用自动TLS证书发放](using-auto-tls.md)主题中有详细介绍。

## 手动添加TLS证书

如果您有一个现有的证书，或者使用了Certbot或证书管理器工具中的一个来手动获取一个新证书，您可以使用以下步骤将该证书添加到您的Knative集群中。

有关为自动证书发放启用Knative的说明，请参见[启用自动TLS证书发放](using-auto-tls.md)。
否则，请按照相应页签的步骤手动添加证书:


=== "Contour"

    要手动向Knative集群添加TLS证书，必须创建一个Kubernetes secret，然后配置Knative Contour插件。

    1. 创建一个Kubernetes secret 来保存您的TLS证书`cert.pem`和私钥`key.pem`，运行以下命令:

           ```bash
           kubectl create -n contour-external secret tls default-cert \
             --key key.pem \
             --cert cert.pem
           ```

        !!! note
            注意命名空间和秘密名称。在以后的步骤中您将需要这些。

    1. 要在不同的名称空间中使用此证书和私钥，必须创建一个委托。为此，使用以下模板创建一个YAML文件:

         ```yaml
         apiVersion: projectcontour.io/v1
         kind: TLSCertificateDelegation
         metadata:
           name: default-delegation
           namespace: contour-external
         spec:
           delegations:
             - secretName: default-cert
               targetNamespaces:
               - "*"
         ```
    1. 运行以下命令应用YAML文件:

        ```bash
        kubectl apply -f <filename>.yaml
        ```
        其中 `<filename>` 是您在上一步中创建的文件的名称。

    1. 更新Knative Contour插件，当autoTLS被禁用时，使用证书作为备份，通过运行以下命令:

         ```bash
         kubectl patch configmap config-contour -n knative-serving \
           -p '{"data":{"default-tls-secret":"contour-external/default-cert"}}'
         ```



=== "Istio"
    要手动添加一个TLS证书到您的Knative集群，您需要创建一个Kubernetes secret，然后配置`knative-ingress-gateway`:

    1. 输入以下命令，创建一个Kubernetes secret来保存您的TLS证书`cert.pem`和私钥`key.pem`:

       ```bash
       kubectl create --namespace istio-system secret tls tls-cert \
         --key key.pem \
         --cert cert.pem
       ```


    1. 配置Knative使用您为HTTPS连接创建的新秘密:

       1. 以编辑模式打开Knative共享的`gateway`:

          ```bash
          kubectl edit gateway knative-ingress-gateway --namespace knative-serving
          ```

       1. 更新`gateway`以包含以下`tls:`部分和配置:

          ```yaml
          tls:
            mode: SIMPLE
            credentialName: tls-cert
          ```

          实例:

          ```yaml
          # Edit the following object. Lines beginning with a '#' will be ignored.
          # An empty file will abort the edit. If an error occurs while saving this
          # file will be reopened with the relevant failures.
          apiVersion: networking.istio.io/v1alpha3
          kind: Gateway
          metadata:
            # ... skipped ...
          spec:
            selector:
              istio: ingressgateway
            servers:
              - hosts:
                  - "*"
                port:
                  name: http
                  number: 80
                  protocol: HTTP
              - hosts:
                  - TLS_HOSTS
                port:
                  name: https
                  number: 443
                  protocol: HTTPS
                tls:
                  mode: SIMPLE
                  credentialName: tls-cert
          ```
          在本例中，`TLS_HOSTS`表示TLS证书的主机。
          它可以是单个主机、多个主机或通配符主机。
          有关详细说明，请参阅[Istio文档](https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/)

## 接下来是什么:

在Knative集群上运行更改之后，可以开始使用HTTPS协议来安全访问部署的Knative服务。
