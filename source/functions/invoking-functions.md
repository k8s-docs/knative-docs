# 调用函数

您可以使用 `func invoke` 命令发送一个测试请求来调用本地或Knative集群上的函数。

这个命令可以用来测试一个函数是否正常工作，是否能够正确地接收HTTP请求和CloudEvents。

如果你的函数在本地运行， `func invoke` 会向本地实例发送一个测试请求。

您可以使用`func invoke`命令将测试数据发送到带有`--data`标志的函数，以及其他选项来模拟不同类型的请求。
更多信息请参见[函数调用](https://github.com/knative/func/blob/main/docs/reference/func_invoke.md){target=_blank}文档。