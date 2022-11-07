<!-- Snippet used in the following topics:
- /docs/getting-started/build-run-deploy-func.md
- /docs/functions/deploying-functions.md
-->
`deploy` 命令使用函数项目名称作为Knative服务名称。
在构建函数时，将使用项目名称和图像注册表名称为函数构造一个完全限定的图像名称。

=== "func"

    通过在项目目录中运行命令来部署函数:

    ```bash
    func deploy --registry <registry>
    ```

=== "kn func"

    通过在项目目录中运行命令来部署函数:

    ```bash
    kn func deploy --registry <registry>
    ```

!!! Success "Expected output"
    ```{ .bash .no-copy }
        🙌 Function image built: <registry>/hello:latest
        ✅ Function deployed in namespace "default" and exposed at URL:
        http://hello.default.127.0.0.1.sslip.io
    ```

你可以通过使用 `invoke` 命令并观察输出来验证你的函数已经成功部署: 

=== "func"

    ```bash
    func invoke
    ```

=== "kn func"

    ```bash
    kn func invoke
    ```

!!! Success "Expected output"
    ```{ .bash .no-copy }
    Received response
    POST / HTTP/1.1 hello.default.127.0.0.1.sslip.io
      User-Agent: Go-http-client/1.1
      Content-Length: 25
      Accept-Encoding: gzip
      Content-Type: application/json
      K-Proxy-Request: activator
      X-Request-Id: 9e351834-0542-4f32-9928-3a5d6aece30c
      Forwarded: for=10.244.0.15;proto=http
      X-Forwarded-For: 10.244.0.15, 10.244.0.9
      X-Forwarded-Proto: http
    Body:
    ```
