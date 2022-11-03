# 部署 Knative 服务

在本教程中，您将部署一个"Hello world"Knative服务，该服务接受环境变量`TARGET`并打印`Hello ${TARGET}!`

=== "kn"

    运行命令部署服务:

    ```bash
    kn service create hello \
    --image gcr.io/knative-samples/helloworld-go \
    --port 8080 \
    --env TARGET=World
    ```
    !!! Success "Expected output"
        ```{ .bash .no-copy }
        Service hello created to latest revision 'hello-world' is available at URL:
        http://hello.default.${LOADBALANCER_IP}.sslip.io
        ```
        The value of `${LOADBALANCER_IP}` above depends on your type of cluster,
        for `kind` it will be `127.0.0.1` for `minikube` depends on the local tunnel.

=== "YAML"
    1. 将以下YAML复制到名为`hello.yaml`的文件中:

        ```yaml
        apiVersion: serving.knative.dev/v1
        kind: Service
        metadata:
          name: hello
        spec:
          template:
            spec:
              containers:
                - image: gcr.io/knative-samples/helloworld-go
                  ports:
                    - containerPort: 8080
                  env:
                    - name: TARGET
                      value: "World"
        ```
    2. 运行命令部署Knative Service:

        ```bash
        kubectl apply -f hello.yaml
        ```
        !!! Success "Expected output"
            ```{ .bash .no-copy }
            service.serving.knative.dev/hello created
            ```
