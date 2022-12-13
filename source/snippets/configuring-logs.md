# 配置日志设置

所有Knative组件的日志配置都通过对应命名空间中的`config-logging` ConfigMap进行管理。
例如，服务组件通过`knative-serving`命名空间中的`config-logging`配置，事件组件通过`knative-eventing` 命名空间中的`config-logging`配置，等等。

Knative组件使用[zap](https://github.com/uber-go/zap)日志库;选项[在该项目中有更详细的文档](https://github.com/uber-go/zap/blob/master/config.go#L58)。

除了`zap-logger-config`，这是一个通用键，适用于该命名空间中的所有组件，`config-logging` ConfigMap支持覆盖单个组件的日志级别。

| ConfigMap key                     | Description                                                                                                                                        |
| :-------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| `zap-logger-config`               | 用于zap记录器配置的JSON对象容器。关键字段在下面突出显示。                                                                                          |
| `zap-logger-config.level`         | 组件的默认日志记录级别。在此级别或以上的消息将被记录。                                                                                             |
| `zap-logger-config.encoding`      | 组件日志的日志编码格式(默认为JSON)。                                                                                                               |
| `zap-logger-config.encoderConfig` | 用于自定义记录内容的 `zap` [EncoderConfig](https://github.com/uber-go/zap/blob/10d89a76cc8b9787e408aee8882e40a8bd29c585/zapcore/encoder.go#L312)。 |
| `loglevel.<component>`            | 仅覆盖给定组件的日志记录级别。在此级别或以上的消息将被记录。                                                                                       |

Zap支持的日志级别有:

- `debug` - 细粒度的调试
- `info` - 正常的日志
- `warn` - 意外但非关键的错误
- `error` - 关键的错误;正常操作时出现意外
- `dpanic` - 在调试模式下，触发恐慌(崩溃)
- `panic` - 引发恐慌(崩溃)
- `fatal` - 立即退出，退出状态为1(失败)
