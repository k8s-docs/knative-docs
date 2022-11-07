<!-- Snippet used in the following topics:
- /docs/getting-started/build-run-deploy-func.md
- /docs/functions/deploying-functions.md
-->
部署函数会为函数创建一个OCI容器映像，并将该容器映像推到映像注册表中。
该功能作为Knative服务部署到集群中。
重新部署函数将更新在集群上运行的容器映像和生成的服务。
已经部署到集群的函数可以在集群上访问，就像任何其他Knative服务一样。