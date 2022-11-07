<!-- Snippet used in the following topics:
- /docs/concepts/eventing-resources/brokers.md
-->
安装Knative函数后，可以使用 `func` 命令行或 `kn func` 插件创建函数项目:

=== "`func` CLI"

    ```bash
    func create -l <language> <function-name>
    ```

    举例:

    ```bash
    func create -l go hello
    ```

=== "`kn func` 插件"

    ```bash
    kn func create -l <language> <function-name>
    ```

    例子:

    ```bash
    kn func create -l go hello
    ```

!!! Success "Expected output"

    ```{ .bash .no-copy }
    Created go function in hello
    ```

有关函数 `create` 命令选项的更多信息，请参阅[func create](https://github.com/knative/func/blob/main/docs/reference/func_create.md){target=_blank}文档。
