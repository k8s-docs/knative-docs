# Knative 代码示例

您可以使用Knative代码示例来帮助您建立和运行常用用例。

## Knative所有样品

由Knative工作组积极测试和维护的Knative代码示例:

- [事件和事件的源代码示例](eventing.md)
- [服务代码示例](serving.md)

## 社区拥有的样品

使用一个社区代码示例启动并运行。 
这些样本是由Knative社区的成员贡献和维护的。
[查看由社区贡献和维护的代码示例](https://github.com/knative/docs/tree/main/code-samples/community).

!!! note 
    这些样本可能会过时，或者原始作者可能无法维持他们的贡献。如果你发现有什么东西不能工作，伸出援助之手，在公共关系中解决它。

[了解更多关于样本寿命的信息](https://github.com/knative/docs/blob/main/contribute-to-docs/what-to-contribute/creating-code-samples.md#user-focused-content)

| Sample Name      | Description                                       | Language(s)                                                                                                                                                                                                                                                                                                                                                                                                          |
| ---------------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hello World      | 快速介绍Knative服务，重点介绍如何部署应用程序。   | [Clojure](serving/helloworld-clojure/), [Dart](serving/helloworld-dart/), [Elixir](serving/helloworld-elixir/), [Haskell](serving/helloworld-haskell/), [Java - Micronaut](serving/helloworld-java-micronaut/), [Java - Quarkus](serving/helloworld-java-quarkus/), [R - Go Server](serving/helloworld-r/), [Rust](serving/helloworld-rust/), [Swift](serving/helloworld-swift/), [Vertx](serving/helloworld-vertx/) |
| Machine Learning | 快速介绍如何使用Knative Serving为机器学习模型服务 | [Python - BentoML](serving/machinelearning-python-bentoml)                                                                                                                                                                                                                                                                                                                                                           |

## 外部代码示例

位于Knative repos之外的Knative代码示例链接列表:

<!--LINK TITLES must match the title of the sample page they link to to avoid confusion and provide a consistent UX). If descriptions are required here, this should be converted to a table as above-->

- [使用Knative事件图像处理，云运行在GKE和谷歌云视觉API](https://github.com/akashrv/knative-samples/blob/master/docs/image-processing.md)
- [Knative事件示例](https://github.com/lionelvillard/knative-examples)
- [knfun](https://github.com/maximilien/knfun)
- [开始使用Knative 2020](https://salaboy.com/2020/02/20/getting-started-with-knative-2020/)
- [图像处理管道](https://github.com/meteatamel/knative-tutorial/blob/master/docs/image-processing-pipeline.md)
- [BigQuery处理管道](https://github.com/meteatamel/knative-tutorial/blob/master/docs/bigquery-processing-pipeline.md)
- [使用SLOs进行简单的性能测试](/blog/articles/performance-test-with-slos/)

!!! tip
    在这里添加一个链接到您的外部托管Knative代码示例。
