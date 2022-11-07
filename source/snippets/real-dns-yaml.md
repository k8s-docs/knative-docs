=== "Real DNS"

    要为Knative配置DNS，请从建立网络中获取External IP或CNAME，并按照以下方法与您的DNS提供商进行配置:

    - If the networking layer produced an External IP address, then configure a
      wildcard `A` record for the domain:

        ```
        # Here knative.example.com is the domain suffix for your cluster
        *.knative.example.com == A 35.233.41.212
        ```

    - If the networking layer produced a CNAME, then configure a CNAME record for the domain:

        ```
        # Here knative.example.com is the domain suffix for your cluster
        *.knative.example.com == CNAME a317a278525d111e89f272a164fd35fb-1510370581.eu-central-1.elb.amazonaws.com
        ```

    - Once your DNS provider has been configured, direct Knative to use that domain:

        ```yaml
        # Replace knative.example.com with your domain suffix
        kubectl patch configmap/config-domain \
          --namespace knative-serving \
          --type merge \
          --patch '{"data":{"knative.example.com":""}}'
        ```
