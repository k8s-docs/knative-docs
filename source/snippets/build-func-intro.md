<!-- Snippet used in the following topics:
- /docs/getting-started/build-run-deploy-func.md
- /docs/functions/building-functions.md
-->
构建函数会为函数创建一个OCI容器映像，可以将其推入容器注册表。
它不运行或部署函数，如果您想在本地为函数构建容器映像，但不想自动运行函数或将其部署到集群(例如在测试场景中)，这可能很有用。