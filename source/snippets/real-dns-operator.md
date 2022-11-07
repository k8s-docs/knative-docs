=== "真正的DNS"

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

    - Once your DNS provider has been configured, add `spec.config.domain` into
    your existing Serving CR, and apply it:

        ```yaml
        # Replace knative.example.com with your domain suffix
        apiVersion: operator.knative.dev/v1alpha1
        kind: KnativeServing
        metadata:
          name: knative-serving
          namespace: knative-serving
        spec:
          ...
          config:
            domain:
              "knative.example.com": ""
          ...
        ```
