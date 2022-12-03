# 事件流

Knative事件提供了一组[自定义资源定义(crd)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)，您可以使用它们来定义事件流:

* [Sequence](sequence/README.md) 用于定义按顺序排列的函数列表。
* [Parallel](parallel.md) 它用于定义一个分支列表，每个分支接收相同的CloudEvent。
