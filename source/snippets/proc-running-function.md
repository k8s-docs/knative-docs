<!-- Snippet used in the following topics:
- /docs/getting-started/build-run-deploy-func.md
- /docs/functions/running-functions.md
-->
如果需要，`run` 命令为函数构建一个映像，并在本地运行该映像，而不是将其部署到集群上。

=== "func"

    在本地运行函数，在项目目录中运行命令:

    ```bash
    func run
    ```

    如果需要，使用此命令还会构建函数。

    你可以通过运行命令强制映像的重建:

    ```bash
    func run --build
    ```

    也可以通过运行以下命令禁用构建:

    ```bash
    func run --build=false
    ```

=== "kn func"

    在本地运行函数，在项目目录中运行命令:

    ```bash
    kn func run
    ```

    如果需要，使用此命令还会构建函数。

    可以通过运行该命令强制重建映像:

    ```bash
    kn func run --build
    ```

    也可以通过运行该命令禁用构建:

    ```bash
    kn func run --build=false
    ```

你可以通过使用 `invoke` 命令并观察输出来验证你的函数已经成功运行:

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
