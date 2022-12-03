# 使用kubectl升级

如果使用YAML安装Knative，则可以在本主题中使用`kubectl apply`命令升级Knative组件和插件。
如果使用Operator安装，请参见[使用Knative Operator升级](upgrade-installation-with-operator.md).

## 在开始之前

在升级之前，必须采取一些步骤以确保升级过程成功。

### 识别破坏性更改

您应该注意Knative当前版本和期望版本之间的任何破坏性更改。
Knative版本之间的突破性更改记录在Knative发布说明中。
在升级之前，请查看目标版本的发布说明，了解您可能需要对Knative应用程序进行的任何更改:

- [Serving](https://github.com/knative/serving/releases)
- [Eventing](https://github.com/knative/eventing/releases)

每个版本的发布说明都发布在GitHub中各自存储库的"Releases"页面上。

### 查看当前Pod状态

在升级之前，查看您计划升级的名称空间的 `pods` 的状态。
这允许您比较名称空间的前后状态。
例如，如果您正在升级Knative服务和事件，输入以下命令查看每个名称空间的当前状态:

```bash
kubectl get pods -n knative-serving
```

```bash
kubectl get pods -n knative-eventing
```

### 升级插件

如果您安装了一个插件，请确保在升级Knative组件的同时升级它。

### 升级前运行预安装工具

对于某些升级，在实际升级之前必须完成一些步骤。
这些步骤(如果适用的话)在发布说明中进行了标识。

### 将现有资源升级到最新的存储版本

Knative自定义资源存储在Kubernetes的特定版本中。
当我们引入较新的支持版本并删除较旧的支持版本时，您必须将资源迁移到指定的存储版本。
这确保了在升级时删除旧版本会成功。

对于各种子项目，有一个K8s工作来帮助操作人员执行这种迁移。
每个版本的版本说明将明确说明是否需要迁移。

## 执行升级

要升级，请为所有已安装Knative组件和特性的后续小版本应用YAML文件，记住每次只升级一个小版本。

升级前，[检查您的Knative版本](check-install-version.md).

对于运行Knative Serving和Knative event组件1.1版本的集群，以下命令将安装升级到1.2版本:

```bash
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.2.0/serving-core.yaml \
-f https://github.com/knative/eventing/releases/download/knative-v1.2.0/eventing.yaml \
```

### 升级完成后运行安装后工具

在某些升级中，有些步骤必须在实际升级之后进行，这些步骤在发布说明中进行了标识。

## 升级后验证

要确认组件和插件已经成功升级，请在相关名称空间中查看它们的 `pods` 的状态。
所有的`Pod`将在升级过程中重新启动，它们的年龄将重置。
如果升级了Knative服务和事件，请输入以下命令来获取关于每个名称空间的`Pod`的信息:

```bash
kubectl get pods -n knative-serving
```

```bash
kubectl get pods -n knative-eventing
```

这些命令返回类似于:

```bash
NAME                                READY   STATUS        RESTARTS   AGE
activator-79f674fb7b-dgvss          2/2     Running       0          43s
autoscaler-96dc49858-b24bm          2/2     Running       1          43s
autoscaler-hpa-d887d4895-njtrb      1/1     Running       0          43s
controller-6bcdd87fd6-zz9fx         1/1     Running       0          41s
net-istio-controller-7fcdf7-z2xmr   1/1     Running       0          40s
webhook-747b799559-4sj6q            1/1     Running       0          41s
```

```bash
NAME                                READY   STATUS        RESTARTS   AGE
eventing-controller-69ffcc6f7d-5l7th   1/1     Running   0          83s
eventing-webhook-6c56fcd86c-42dr8      1/1     Running   0          81s
imc-controller-6bcf5957b5-6ccp2        1/1     Running   0          80s
imc-dispatcher-f59b7c57-q9xcl          1/1     Running   0          80s
sources-controller-8596684d7b-jxkmd    1/1     Running   0          83s
```

如果所有`Pod`的年龄都已重置，并且所有`Pod`都已启动并运行，则升级已成功完成。
你可能会注意到旧`Pod`被清理时的`Terminating`状态。

如果需要，重复升级过程，直到达到所需的次要版本号。
